# BIOTECH — TEAM CONVERGENCE REGISTER
**Auditor:** Pratik Bhuvad  
**Role:** AI/ML + Data Audit Owner — Independent verification of reproducibility and analytical/data integrity  
**Date:** 2026-10-05  
**Status:** Phase 4 Six-Repository System-Wide Audit (Authoritative & Verified)

---

## 1. Executive Summary & Purpose

This Convergence Register evaluates the architectural, semantic, and interface compatibility across the complete six-repository Biotech implementation set:
1. `Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main` (Pharmacovigilance Pipeline v1 — Sarvesh Patil)
2. `bhiv-MHEALTH-Production-main` (BHIV MHEALTH Production Runtime Task 4 — MHEALTH Task 4 Team)
3. `BHIV_Formulation_Stability_T3_Day1` (Formulation & Stability Evidence Intelligence — Sara)
4. `Drug_Safety_QC_Evidence_Map-main` (Drug Safety Evidence Integration — Shravani)
5. `drug-discovery--main` (TB Drug-Discovery Candidate Intelligence — Shamit / TMS)
6. `symmetrical-octo-invention-main` (Biosimilarity & Preclinical Evidence QC — Shyamal)

### Purpose & Explicit Non-Goals
* **Purpose:** Identify where the six systems **can connect**, **may overlap**, **are incompatible**, **duplicate effort**, **require contract definition**, or **require team decisions**.
* **Strict Non-Goal:** This document is **NOT** a code-merging plan or an implementation blueprint.
* **Architectural Boundary Rule:** This audit does **NOT** prescribe merging or refactoring. Where potential integration points exist, this register states:
  > *Potential convergence point identified. Requires team-level contract decision before implementation.*

---

## 2. Six-Repository System-Wide Capability Mapping

| Capability Area | Lead / Documented Owner | Repository | Primary Focus | Runtime Interface | Verification Status |
|---|---|---|---|---|---|
| **Formulation & Stability Evidence Intelligence** | Sara | `BHIV_Formulation_Stability_T3_Day1` | Drug formulation excipients, shelf-life stability tests, and storage conditions | CLI (`run.py`) | **VERIFIED** (11/11 tests pass out-of-box) |
| **Drug Safety Evidence Integration** | Shravani | `Drug_Safety_QC_Evidence_Map-main` | Multi-source safety evidence aggregation, duplicate/conflict detection, human review flagging | CLI & Module (`evidence_integration.py`, `drug_safety_tracker.py`) | **VERIFIED WITH WRAPPER** (23/23 tests pass with shim) |
| **TB Drug-Discovery Candidate Intelligence** | Shamit (TMS) | `drug-discovery--main` | Tuberculosis candidate compound identification, multi-criteria evidence scoring (0–40), and gap analysis | CLI (`drug_discovery_pipeline.py`, `calculate_completeness.py`) | **PARTIALLY VERIFIED** (Manual CLI passed; missing JSON pipeline artifacts) |
| **Biosimilarity & Preclinical Evidence QC** | Shyamal | `symmetrical-octo-invention-main` | Quality control classification, candidate-reference biosimilar comparison, synthetic data protection | CLI (`qc_engine/engine.py`, `evidence_tracker.py`) | **VERIFIED** (24/24 tests pass out-of-box) |
| **Pharmacovigilance Core Ingestion Pipeline** | Sarvesh Patil | `Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main` | Ingestion of live OpenFDA FAERS adverse event reports, entity resolution, and evidence store | CLI (`app.py`) | **VERIFIED WITH WRAPPER** (47/47 tests pass with shim) |
| **MHEALTH Telemetry Microservice & FHIR Gateway** | MHEALTH Task 4 Team | `bhiv-MHEALTH-Production-main` | Patient physiological vital signs ingestion, threshold classification, SQLite persistence, and HL7 FHIR R4 export | REST API (FastAPI / Uvicorn) | **VERIFIED WITH WRAPPER** (39/39 tests pass with shim) |

---

## 3. Convergence Evaluation Across 15 Architectural Categories

| Category # | Evaluation Category | Status Across 6 Repositories | Architectural Assessment |
|---|---|---|---|
| **1** | Input Schema | **INCOMPATIBLE** | Each of the 6 repositories consumes an entirely distinct input format (FAERS nested JSON, biometric telemetry CSV/JSON, formulation stability CSV, multi-source safety CSV, TB compound CSV, biosimilarity QC JSON). No shared input parser exists. |
| **2** | Output Schema | **INCOMPATIBLE** | Five repositories produce proprietary, structurally divergent JSON dictionaries or text reports. Only Repo 2 (`bhiv-MHEALTH-Production-main`) implements a standardized healthcare export format (HL7 FHIR R4 `Observation`). |
| **3** | Identifiers | **INCOMPATIBLE** | Repositories utilize divergent identity strategies: random UUID v4 (Repo 2), content-derived MD5 fingerprints (Repo 1), prefixed strings (`E001` in Repo 3, `EV001` in Repo 4, `C-001` in Repo 5, `SYN-001` in Repo 6), and SQLite integer primary keys. |
| **4** | Provenance | **PARTIALLY COMPATIBLE** | Five repositories record source references and timestamps, but operate at conflicting grains (record-level vs batch upload vs DOI citation). Neither repository provides a provenance structure consumable by another. |
| **5** | Validation | **INCOMPATIBLE** | Validation methodologies range from strict Pydantic v2 schemas (Repo 2) and JSON Schema v1.0.0 (Repo 4, Repo 6) to procedural dictionary checking (Repo 1, Repo 3). Levels of rigor conflict (permissive nulls in Repo 1 vs strict checks in Repo 6). |
| **6** | Error Handling | **PARTIALLY COMPATIBLE** | Repositories 1, 2, 3, 4, and 6 return structured error payloads or exception handling, but Repo 2 uses HTTP status codes while the others return internal Python dictionaries or console messages. |
| **7** | Persistence | **INCOMPATIBLE** | Repo 2 uses a relational SQL database (`signals` table in SQLite/PostgreSQL). Repos 1, 3, 4, 5, and 6 write to flat JSON files or local CSV spreadsheets. No shared database or storage repository exists. |
| **8** | API Interfaces | **INCOMPATIBLE** | Only Repo 2 exposes an HTTP REST API microservice with OpenAPI/Swagger. The remaining 5 repositories are local command-line scripts or importable Python modules with no listening network interfaces. |
| **9** | Reproducibility | **PARTIALLY COMPATIBLE** | Two repositories execute and pass tests 100% out-of-the-box (`BHIV_Formulation_Stability_T3_Day1`: 11/11; `symmetrical-octo-invention-main`: 24/24). Three require diagnostic shims (Repo 1, Repo 2, Repo 4). One lacks automated tests (Repo 5). |
| **10** | Test Strategy | **PARTIALLY COMPATIBLE** | Five repositories use `pytest` with synthetic fixtures, mocking, or unit testing. However, their test directories, fixtures, and execution environments are completely isolated. |
| **11** | Data Semantics | **INCOMPATIBLE** | Repositories represent six disjoint clinical/scientific subdomains: post-marketing pharmacovigilance, real-time sensor telemetry, formulation stability excipients, drug safety QC records, TB small-molecule candidates, and biosimilar preclinical assays. |
| **12** | External Dependencies | **PARTIALLY COMPATIBLE** | Five repositories are fully hermetic local runtimes relying primarily on the Python standard library. Only Repo 1 has an active external network dependency (live US FDA OpenFDA REST API). |
| **13** | Evidence Traceability | **PARTIALLY COMPATIBLE** | All six repositories emphasize preserving raw input values alongside normalized or classified outputs, which strongly supports auditability. |
| **14** | Export Formats | **INCOMPATIBLE** | Flat proprietary JSON files vs HL7 FHIR R4 Observations. No transformation contract exists to convert pharmacovigilance, formulation, or biosimilarity evidence into FHIR resources (`AdverseEvent`, `MedicationKnowledge`, etc.). |
| **15** | AI/ML Dependency | **NOT APPLICABLE** | **NO ML MODEL PRESENT ACROSS ALL SIX REPOSITORIES.** All six systems operate entirely on deterministic algorithms, threshold comparators, or scoring lookup tables. |

---

## 4. Deep-Dive on Critical Integration Interfaces

### A. Evidence Ingestion Pipelines
```text
Repo 1 (Sarvesh):    OpenFDA / FAERS ──► Normalization ──► Entity Resolution ──► Evidence Store
Repo 2 (MHEALTH):    Telemetry (CSV) ──► Validation ──► Rule Classifier ──► SQLite DB ──► FHIR Export
Repo 3 (Sara):       Formulation CSV ──► 23-Field Validation ──► Duplicate Check ──► JSON Report
Repo 4 (Shravani):   Safety Evidence ──► Normalization ──► Duplicate/Conflict Check ──► Review Flagging
Repo 5 (Shamit):     TB Compounds ──► Completeness Check ──► Static Criteria Scoring ──► CSV Export
Repo 6 (Shyamal):    Biosimilar JSON ──► Schema QC Check ──► Review Routing ──► QC JSON Output
```
* **Investigation:** Can any of these ingestion pipelines consume data from another?
* **Audit Finding:** **NO DIRECT CONTRACT IDENTIFIED.**
  - Every pipeline was built as an isolated, standalone project.
  - No interface, function, adapter, or schema across any of the six repositories is designed to consume the output of another repository.
* **Convergence Note:** *Potential convergence point identified. Requires team-level contract decision before implementation.*

---

### B. Duplication & Overlap Analysis

| Functional Capability | Repositories Implementing It | Overlap Classification | Details |
|---|---|---|---|
| **Entity Normalization** | Repo 1 (`normalize_evidence.py`), Repo 4 (`normalize_evidence.py`, `drug_matcher.py`) | **DUPLICATE / SIMILAR BUT DIFFERENT** | Both lowercase and trim drug strings, but Repo 1 maps FAERS drug/reaction terms while Repo 4 matches internal product names and synonyms. |
| **Duplicate Detection** | Repo 1 (`evidence_store.py`), Repo 3 (`find_duplicates`), Repo 4 (`detect_duplicates.py`), Repo 6 (`qc_engine/engine.py`) | **DUPLICATE IMPLEMENTATION** | Four separate repositories independently implemented custom duplicate detection algorithms with different logic (MD5 hash vs source DOI vs source ID vs compound key). |
| **Conflict Detection** | Repo 4 (`detect_conflicts.py`), Repo 6 (`qc_engine/engine.py`) | **DUPLICATE IMPLEMENTATION** | Both detect contradictory evidence, but Repo 4 checks negative substrings while Repo 6 checks opposing qualitative results for the same candidate/reference pair. |
| **Quality & Completeness Classification** | Repo 3 (`classify_quality`), Repo 5 (`calculate_completeness.py`), Repo 6 (`EVIDENCE_QUALITY_STATES`) | **SIMILAR BUT FRAGMENTED** | Repo 3 scores quality (`High`, `Moderate`), Repo 5 calculates percentage completeness (80%–100%), and Repo 6 assigns quality states (`VERIFIED`, `PARTIAL`, `NEEDS REVIEW`, `CONFLICTING`, `INSUFFICIENT`). |
| **Human Review Routing** | Repo 3 (`human_review_required`), Repo 4 (`human_review.py`), Repo 6 (`reviewer_status`, `final_review_status`) | **COMPLEMENTARY BUT DISCONNECTED** | Three repositories flag records requiring human review, but utilize incompatible status enums and data structures. |
| **FHIR Standardization** | Repo 2 (`fhir_service.py`) | **ISOLATED CAPABILITY** | Only Repo 2 implements HL7 FHIR R4 standard export. None of the other five repositories utilize FHIR. |

---

### C. Identifier Generation & Persistence Audit

| Repository | Source Identifier | Business / Natural Key | Database Identifier | Surrogate / UUID | Reproducibility & Stability |
|---|---|---|---|---|---|
| **Repo 1 (Pharmacovigilance)** | FAERS `safetyreportid` | MD5 hash of `drug_event_timestamp` | In-memory list index | None | **STABLE & REPRODUCIBLE** (Deterministic hash) |
| **Repo 2 (MHEALTH)** | `subject_id` (e.g. `subj-01`) | None | SQLite autoincrement integer PK | Random UUID v4 per row & upload | **UNSTABLE** (Fresh UUIDs on duplicate ingestion) |
| **Repo 3 (Sara)** | DOI or literature citation | `evidence_id` (`E001`–`E008`) | Array index in JSON | None | **STABLE & REPRODUCIBLE** (Fixed prefixed IDs) |
| **Repo 4 (Shravani)** | Source report ID (`EV001`) | `evidence_id` (`EV001`–`EV010`) | List index in JSON | None | **STABLE & REPRODUCIBLE** (Fixed prefixed IDs) |
| **Repo 5 (Shamit)** | Literature source | Candidate ID (`C-001`–`C-010`) | CSV row index | None | **STABLE & REPRODUCIBLE** (Fixed candidate IDs) |
| **Repo 6 (Shyamal)** | Study/source citation | `evidence_id` (`SYN-001`–`SYN-011`) | Array index in JSON | None | **STABLE & REPRODUCIBLE** (Fixed synthetic IDs) |

---

### D. AI/ML Inventory & Verification Across All Six Repositories

```text
================================================================================
AI/ML INVENTORY RESULT ACROSS ALL SIX REPOSITORIES
================================================================================
Does the complete Biotech system currently contain an actual ML model?
NO.
No ML model was identified in the six audited repositories.
================================================================================
```
* **Repo 1 (`Live-Pharmacovigilance...`):** Deterministic string normalization and static dictionary alias matching.
* **Repo 2 (`bhiv-MHEALTH-Production`):** Deterministic rule-based threshold comparison (`if value < lower: WARNING`).
* **Repo 3 (`BHIV_Formulation_Stability`):** Deterministic schema field presence and duplicate checking.
* **Repo 4 (`Drug_Safety_QC_Evidence_Map`):** Deterministic exact/partial string matching and keyword conflict detection.
* **Repo 5 (`drug-discovery`):** Deterministic scoring lookup table (`SCORES = {"C-001": [5, 5, ...], ...}`).
* **Repo 6 (`symmetrical-octo-invention`):** Deterministic rule-based QC state machine and schema validation.
* **Conclusion:** No machine learning weights, learned embeddings, neural networks, or external AI APIs exist in any of the six repositories. There is no ML convergence requirement.

---

## 5. Cross-Repository Data Contract Matrix

| Repository | Input Format | Output Format | Primary Identifier | Provenance Representation | Runtime Interface | Downstream Consumer Identified |
|---|---|---|---|---|---|---|
| **Repo 1 (Pharmacovigilance)** | OpenFDA FAERS JSON | JSON `EvidenceRecord` array | FAERS ID + MD5 hash | `source`, `timestamp`, `metadata` | Standalone CLI (`app.py`) | **NO VERIFIED CONSUMER** |
| **Repo 2 (MHEALTH)** | Telemetry CSV / JSON | JSON `SignalResponse` + FHIR R4 | Random UUID v4 + `subject_id` | `dataset_provenance` table | FastAPI REST API | **NO VERIFIED CONSUMER** |
| **Repo 3 (Sara)** | Formulation CSV | JSON validation & summary report | `evidence_id` (`E001`) | `citation`, `doi_identifier` | Standalone CLI (`run.py`) | **NO VERIFIED CONSUMER** |
| **Repo 4 (Shravani)** | Multi-source safety CSV | Integrated evidence JSON | `evidence_id` (`EV001`) | `source`, `supporting_doc` | CLI (`drug_safety_tracker.py`) | **NO VERIFIED CONSUMER** |
| **Repo 5 (Shamit)** | TB candidate CSV | Console text + comparison CSV | Candidate ID (`C-001`) | `data_source` column | Interactive CLI (`pipeline.py`) | **NO VERIFIED CONSUMER** |
| **Repo 6 (Shyamal)** | Biosimilarity JSON / CSV | Structured QC output JSON | `evidence_id` (`SYN-001`) | `study_or_source`, `synthetic` | CLI (`qc_engine/engine.py`) | **NO VERIFIED CONSUMER** |

---

## 6. Capability Convergence Matrix

| Capability Area | Repository | Implementation State | AI/ML Present | Input Schema | Output Schema | Reproducibility Status | Integration Status |
|---|---|---|---|---|---|---|---|
| **Formulation & Stability (Sara)** | `BHIV_Formulation_Stability_T3_Day1` | **IMPLEMENTED** (Complete validator) | **NO** | CSV (23 fields) | JSON report | **REPRODUCIBLE** (11/11 tests pass out-of-box) | **ISOLATED BUILD** (No active integration) |
| **Drug Safety Integration (Shravani)** | `Drug_Safety_QC_Evidence_Map-main` | **IMPLEMENTED** (Complete engine) | **NO** | CSV / JSON | JSON output | **TESTS PASS WITH WORKAROUND** (23/23 pass with shim) | **ISOLATED BUILD** (No active integration) |
| **TB Drug Discovery (Shamit)** | `drug-discovery--main` | **PARTIAL** (Core search works; advanced JSON missing) | **NO** | CSV (10 compounds) | Text / CSV | **PARTIALLY REPRODUCIBLE** (0 automated tests; missing JSON) | **ISOLATED BUILD** (No active integration) |
| **Biosimilarity QC (Shyamal)** | `symmetrical-octo-invention-main` | **IMPLEMENTED** (Complete QC engine) | **NO** | JSON (11 records) | QC JSON | **REPRODUCIBLE** (24/24 tests pass out-of-box) | **ISOLATED BUILD** (No active integration) |
| **Pharmacovigilance Core (Sarvesh)** | `Live-Pharmacovigilance-Evidence...` | **IMPLEMENTED** (Complete pipeline) | **NO** | OpenFDA JSON | JSON array | **TESTS PASS WITH WORKAROUND** (47/47 pass with shim) | **ISOLATED BUILD** (No active integration) |
| **MHEALTH Telemetry (Task 4)** | `bhiv-MHEALTH-Production-main` | **IMPLEMENTED** (Complete REST API) | **NO** | Telemetry CSV/JSON | JSON + FHIR | **TESTS PASS WITH WORKAROUND** (39/39 pass with shim) | **ISOLATED BUILD** (No active integration) |

---

## 7. Six-Repository Convergence Decision Matrix

| Area | Observed System State Across 6 Repositories | Compatibility Status | Decision Needed from Engineering Team |
|---|---|---|---|
| **Input Ingestion** | 6 disjoint input formats across 6 repositories. | **INCOMPATIBLE** | Should an API gateway or shared ingestion adapter normalize inputs before processing? |
| **Output Formats** | 5 proprietary JSON formats vs 1 HL7 FHIR R4 implementation. | **INCOMPATIBLE** | Should all evidence repositories adopt HL7 FHIR R4 (e.g. `AdverseEvent`, `MedicationKnowledge`, `Observation`)? |
| **Duplicate Detection** | 4 separate, incompatible duplicate detection implementations. | **DUPLICATE EFFORT** | Should duplicate and conflict detection be unified into a shared core library? |
| **Validation Framework** | Pydantic v2 (Repo 2) vs JSON Schema (Repo 4, 6) vs procedural dicts. | **INCOMPATIBLE** | What validation standard should be established across all Biotech capabilities? |
| **Persistence** | Relational SQL (Repo 2) vs local flat files (Repos 1, 3, 4, 5, 6). | **INCOMPATIBLE** | Will non-telemetry evidence records be stored in a shared SQL database or document store? |
| **API Architecture** | FastAPI REST service (Repo 2) vs standalone CLI scripts (Repos 1, 3, 4, 5, 6). | **INCOMPATIBLE** | Should the CLI capabilities (Sara, Shravani, Shamit, Shyamal, Sarvesh) be exposed as microservice endpoints on FastAPI? |
| **Synthetic Labeling** | Explicit `is_synthetic` protection (Repo 6) vs unflagged synthetic exports (Repo 2). | **PARTIALLY COMPATIBLE** | Should Repo 6's strict synthetic data labeling rules be mandated across Repo 2 and the rest of the system? |
| **Reproducibility** | Repos 3 & 6 pass 100% out-of-box; Repos 1, 2, 4 require wrappers; Repo 5 has 0 tests. | **PARTIALLY COMPATIBLE** | What CI/CD standardization and packaging fixes will be officially adopted? |
| **AI/ML Strategy** | 0 ML models present; 100% rule-based deterministic algorithms. | **NOT APPLICABLE** | Is the system intended to remain purely deterministic, or will external ML models be introduced? |

---

## 8. Strategic Convergence Boundary Recommendations

1. **Do NOT Merge Repositories Without Formal Data Contracts:**
   Each repository operates in a distinct scientific or operational domain. Forcing code merges without agreed schemas will break isolated functionality.
2. **Adopt a Unified Validation & Evidence Standard (Pydantic v2 + FHIR R4):**
   Standardize on Pydantic v2 for data models and adopt HL7 FHIR R4 profiles across all clinical and safety domains.
3. **Extract Shared Logic into Core Libraries:**
   Consolidate duplicate implementations of duplicate detection, conflict detection, and string normalization into a shared `biotech-core` utility package.
4. **Enforce Synthetic Data Provenance Across All Pipelines:**
   Propagate Shyamal's explicit synthetic data labeling standard (`is_synthetic: bool`) across MHEALTH and the rest of the ecosystem.

---

## 9. Phase 5 Decision & Contract Verification Status

**Audit Pass:** Phase 5 Decision & Contract Verification (READ-ONLY)  
**Authoritative Reference Documents:**
- [BIOTECH_CONTRACT_VERIFICATION_REGISTER.md](file:///c:/Pratik_Bhuwad/Biotech/BIOTECH_CONTRACT_VERIFICATION_REGISTER.md)
- [BIOTECH_DECISION_VERIFICATION_REGISTER.md](file:///c:/Pratik_Bhuwad/Biotech/BIOTECH_DECISION_VERIFICATION_REGISTER.md)
- [BIOTECH_PHASE5_DECISION_CONTRACT_VERIFICATION.md](file:///c:/Pratik_Bhuwad/Biotech/BIOTECH_PHASE5_DECISION_CONTRACT_VERIFICATION.md)

### Verified Convergence Status Across Capability Areas

1. **Sara (Formulation Stability) $\longleftrightarrow$ Shyamal (Biosimilarity QC):**
   * **Domain Relationship:** Distinct scientific capabilities (chemical small-molecule tablet stability vs. biologic preclinical biosimilarity).
   * **Contract Status:** **INCOMPATIBLE / NO CONTRACT (CONTRACT-FS-BIO-001, Maturity Level 0)**. Zero schema overlap beyond basic metadata. No operational requirement to merge these engines.
2. **Sarvesh (Task 1 PV) $\longrightarrow$ Shravani (Task 3 Drug Safety QC):**
   * **Domain Relationship:** Highly complementary adverse event intelligence and validation.
   * **Contract Status:** **COMPATIBLE WITH TRANSFORMATION (CONTRACT-PV-DS-001, Maturity Level 1)**. Incompatible out-of-the-box (`validate_record` fails with 4 fatal errors on Sarvesh's `EV-000001`). Requires implementing `BiotechSafetyAdapter` to map schemas.
3. **Shamit (TB Drug Discovery) $\longrightarrow$ Complete Ecosystem:**
   * **Domain Relationship:** Upstream candidate compound screening.
   * **Contract Status:** **BLOCKED / NO CONTRACT (CONTRACT-DD-001, Maturity Level 0)**. Execution blocked by path error `../candidate_compounds.csv`, missing `candidate_records_v1.json`, and 0 automated tests.
4. **MHEALTH (Task 4 Telemetry) $\longrightarrow$ Downstream Clinical EHR:**
   * **Domain Relationship:** Patient vitals monitoring gateway.
   * **Contract Status:** **PARTIALLY COMPATIBLE (CONTRACT-MH-001, Maturity Level 3)**. Functional FHIR R4 export exists, but lacks `is_synthetic` extension tag on test fixtures (`subj-01`).
5. **System-Wide Synthetic Provenance:**
   * **Contract Status:** **PARTIAL / UNAPPROVED (CONTRACT-PROV-001, Maturity Level 1)**. Shyamal enforces string enum `synthetic_data_declaration`; MHEALTH lacks synthetic metadata.
6. **System-Wide Identifiers:**
   * **Contract Status:** **NO CONTRACT (CONTRACT-ID-001, Maturity Level 0)**. Six disjoint identifier formats (`EV-000001`, `EV-001`, `E001`, `C-001`, `SYN-001`, `id: 1`).

