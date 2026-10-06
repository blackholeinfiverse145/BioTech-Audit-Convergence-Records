# BIOTECH — CONTRACT VERIFICATION REGISTER
**Auditor:** Pratik Bhuvad  
**Role:** AI/ML + Data Audit Owner — Independent verification of reproducibility and analytical/data integrity  
**Date:** 2026-10-05  
**Status:** Phase 5 Decision & Contract Verification (Authoritative & Verified — READ-ONLY)

---

## 1. Executive Summary & Purpose

This register provides a rigorous, evidence-backed evaluation of all proposed and candidate cross-repository data contracts across the six Biotech repositories:
1. `Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main` (Sarvesh Patil — Task 1 PV Pipeline)
2. `bhiv-MHEALTH-Production-main` (MHEALTH Task 4 Team — Telemetry Microservice & FHIR Gateway)
3. `BHIV_Formulation_Stability_T3_Day1` (Sara — Formulation & Stability Intelligence)
4. `Drug_Safety_QC_Evidence_Map-main` (Shravani — Drug Safety Evidence Integration)
5. `drug-discovery--main` (Shamit / TMS — TB Drug-Discovery Candidate Intelligence)
6. `symmetrical-octo-invention-main` (Shyamal — Biosimilarity & Preclinical Evidence QC)

### Contract Maturity Model (CMM)
Every candidate contract is assessed against the five-level maturity scale:
* **LEVEL 0 — NO CONTRACT:** No shared schema, interface, or data exchange mechanism exists. Systems operate as isolated silos.
* **LEVEL 1 — INFORMAL / DOCUMENTED ONLY:** Conceptual alignment or narrative documentation suggests integration, but no executable schema or code contract exists.
* **LEVEL 2 — SCHEMA EXISTS:** One or both repositories define formal schemas (Pydantic, JSON Schema, Dataclass), but schemas are divergent or incompatible.
* **LEVEL 3 — IMPLEMENTED INTERFACE:** An adapter, serializer, or API endpoint is implemented to exchange data between repositories.
* **LEVEL 4 — INTEGRATION TESTED:** An automated test verifies data serialization, transmission, and deserialization across repositories.
* **LEVEL 5 — REPRODUCIBLY VERIFIED:** End-to-end data exchange is proven deterministic, idempotent, and validated in CI/CD.

---

## 2. Cross-Repository Contract Verification Matrix

| Contract ID | Source Repo | Target Repo | Technical Purpose | Maturity Level | Status | Transformation Required? | Implementation Required? |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CONTRACT-PV-DS-001** | Sarvesh (Task 1 PV) | Shravani (Task 3 Safety) | Adverse Event Signal Ingestion & QC | **LEVEL 1** | **COMPATIBLE WITH TRANSFORMATION** | **YES (Schema Adapter Mandatory)** | **YES** |
| **CONTRACT-FS-BIO-001** | Sara (Formulation) | Shyamal (Biosimilarity) | Formulation Stability & Preclinical QC | **LEVEL 0** | **INCOMPATIBLE / NO CONTRACT** | **YES (Domain Boundary Redesign)** | **YES** |
| **CONTRACT-MH-001** | MHEALTH (Task 4) | External / Downstream | Clinical Telemetry & FHIR Observations | **LEVEL 3** | **PARTIALLY COMPATIBLE (ISOLATED)** | **YES (Synthetic Tag Extension)** | **YES** |
| **CONTRACT-DD-001** | Shamit (TB Discovery) | Downstream Pipelines | Candidate Compound Ranking & Properties | **LEVEL 0** | **BLOCKED / NO CONTRACT** | **YES (Data Pipeline Restoration)** | **YES** |
| **CONTRACT-PROV-001** | All 6 Repositories | Common Governance | System-Wide Synthetic Data Provenance | **LEVEL 1** | **PROPOSED — NOT APPROVED** | **YES (Standardization Required)** | **YES** |
| **CONTRACT-ID-001** | All 6 Repositories | Common Governance | Global Entity & Evidence Identifier System | **LEVEL 0** | **NO CONTRACT ESTABLISHED** | **YES (Identifier Architecture)** | **YES** |

---

## 3. Detailed Contract Verification Records

---

### [CONTRACT-PV-DS-001] Task 1 Pharmacovigilance → Task 3 Drug Safety QC Ingestion
* **Source Repository:** `Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main`
* **Target Repository:** `Drug_Safety_QC_Evidence_Map-main`
* **Business / Technical Purpose:** Transfer extracted and normalized pharmacovigilance adverse event signals from Sarvesh's OpenFDA ingestion pipeline into Shravani's Drug Safety QC map for duplicate detection, conflict analysis, and evidence validation.
* **Current Source Output:** Canonical JSON list of `EvidenceRecord` dataclass objects persisted in [evidence_output.json](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main/evidence_output.json).
* **Current Target Expected Input:**
  1. Module `validate_evidence.py`: Expects dictionary with `evidence_id`, `source`, `drug_product`, `observation`, `evidence_type`, `evidence_provenance`.
  2. Schema `canonical_schema.json`: Expects 16 required fields including `source_organisation`, `population_context`, `measurement_result`, `supporting_documentation`, `evidence_quality`, `validation_status`, `reviewer_action`, `human_review_flag`.
* **Field-Level Direct Compatibility Audit:**

| Sarvesh Field (Task 1) | Sarvesh Type | Shravani Expected Field (Task 3) | Shravani Type | Directly Compatible? | Transformation / Remediation Required |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `evidence_id` | `str` (e.g. `"EV-000001"`) | `evidence_id` | `str` (e.g. `"EV-001"`) | **YES** | Format difference (6-digit vs 3-digit), but string-compatible. |
| `source` | `str` (e.g. `"OpenFDA/FAERS"`) | `source` | `str` (e.g. `"FAERS"`) | **YES** | String-compatible; normalization recommended. |
| *None* | *Missing* | `source_organisation` | `str` | **NO** | Must default to source system (e.g. `"US FDA"`). |
| `drug_name` | `str` (e.g. `"IBUPROFEN"`) | `drug_product` | `str` | **NO** | Rename `drug_name` $\to$ `drug_product`. |
| `drug_identifier` | `str` (e.g. `"ANDA078682"`) | `drug_product_identifier` | `str` | **NO** | Rename `drug_identifier` $\to$ `drug_product_identifier`. |
| `adverse_event` | `str` (e.g. `"Dyspepsia"`) | `observation` | `str` | **NO** | Rename `adverse_event` $\to$ `observation`. |
| `evidence_type` | `str` (`"FAERS adverse-event report"`) | `evidence_type` | Enum (`"ADVERSE_EVENT"`, etc.) | **NO** | Value rejected by Shravani validator. Must map `"FAERS adverse-event report"` $\to$ `"ADVERSE_EVENT"`. |
| `event_date` / `retrieved_at` | `str` (ISO-8601 or `null`) | `date` | `str` (`YYYY-MM-DD`) | **NO** | Must extract date component (`YYYY-MM-DD`) from timestamp. |
| *None* | *Missing* | `population_context` | `str` | **NO** | Must provide default or extract from FAERS patient demography. |
| *None* | *Missing* | `measurement_result` | `str` | **NO** | Set to `"NOT_REPORTED"` or `"ADVERSE_EVENT_OCCURRENCE"`. |
| `source_url` | `str` | `supporting_documentation` | `str` | **NO** | Map `source_url` and `raw_record_reference` $\to$ `supporting_documentation`. |
| `provenance` | `dict` (keys: `source_system`, `underlying_dataset`, `retrieval_method`, `transformation`) | `evidence_provenance` | `object` or `str` | **NO** | Rename key `provenance` $\to$ `evidence_provenance`. Align subfields (`original_source`, `transformation_path` array). |
| *None* | *Missing* | `evidence_quality` | Enum (`High`, `Moderate`, `Low`, `Unknown`) | **NO** | Calculate initial quality tier or default to `"Unknown"`. |
| `entity_resolution_status` | Enum (`MATCHED`, `AMBIGUOUS`, `UNRESOLVED`) | `validation_status` | Enum (`Validated`, `Unverified`, etc.) | **NO** | Map resolution status to validation status enum. |
| *None* | *Missing* | `limitations` | `str` | **NO** | Populate from pipeline metadata or default to empty string. |
| *None* | *Missing* | `reviewer_action` | `str` | **NO** | Default to `"PENDING_QC_REVIEW"`. |
| *None* | *Missing* | `human_review_flag` | `bool` | **NO** | Derive from `entity_resolution_status != "MATCHED"`. |

* **Empirical Execution Evidence:** Direct execution of Shravani's `validate_record()` on Sarvesh's canonical record `EV-000001` resulted in immediate rejection:
  ```python
  {
      'evidence_id': 'EV-000001',
      'human_review_required': True,
      'validation_errors': [
          'MISSING_REQUIRED_FIELD:drug_product',
          'MISSING_REQUIRED_FIELD:observation',
          'INVALID_EVIDENCE_TYPE',
          'MISSING_PROVENANCE',
      ],
      'validation_status': 'INVALID',
  }
```
* **Contract Maturity:** **LEVEL 1 (INFORMAL / DOCUMENTED ONLY)**. Both repositories exist and process drug safety evidence, but no shared contract or transformation adapter has been implemented or tested.
* **Status:** **COMPATIBLE WITH TRANSFORMATION — CANONICAL CONTRACT NOT YET ESTABLISHED**.
* **Implementation Required:** **YES**. A bidirectional transformation adapter or shared schema module must be implemented before data can flow between Task 1 and Task 3.

---

### [CONTRACT-FS-BIO-001] Sara Formulation Stability $\longleftrightarrow$ Shyamal Biosimilarity QC
* **Source Repository:** `BHIV_Formulation_Stability_T3_Day1`
* **Target Repository:** `symmetrical-octo-invention-main`
* **Business / Technical Purpose:** Exchange formulation degradation parameters, excipient profiles, and stability boundaries with preclinical biosimilarity analytical equivalence testing.
* **Current Status:** **INCOMPATIBLE / NO CONTRACT (LEVEL 0)**.
* **Schema Comparison Evidence:**
  - Sara ([evidence_intelligence.py:L10-17](file:///c:/Pratik_Bhuwad/Biotech/BHIV_Formulation_Stability_T3_Day1/src/evidence_intelligence.py#L10-L17)) requires 23 specific chemical formulation fields: `dosage_form`, `formulation_excipients`, `storage_conditions`, `stability_test`, `measurement`, `result`, `observed_evidence`, `api_model`.
  - Shyamal ([engine.py:L72-94](file:///c:/Pratik_Bhuwad/Biotech/symmetrical-octo-invention-main/qc_engine/engine.py#L72-L94)) requires 21 biologic preclinical fields: `sample_identification`, `control_identification`, `experimental_conditions`, `measurement_method`, `raw_data_availability`, `synthetic_data_declaration`.
  - **Overlap:** Only generic metadata fields overlap (`evidence_id`, `source`, `evidence_quality`, `limitations`). The scientific domains are distinct: chemical small-molecule/tablet formulation vs. biologic biosimilarity assay testing.
* **Contract Maturity:** **LEVEL 0 (NO CONTRACT)**.
* **Implementation Required:** **YES (Architectural Re-scoping)**. These systems do not have an operational integration need unless the team defines a composite drug-product stability-equivalence workflow.

---

### [CONTRACT-MH-001] MHEALTH Telemetry Service $\longrightarrow$ Downstream Clinical / FHIR Consumers
* **Source Repository:** `bhiv-MHEALTH-Production-main`
* **Target Repository:** External FHIR Clients / Downstream Biotech Consumers
* **Business / Technical Purpose:** Provide real-time patient telemetry ingestion and standardized HL7 FHIR R4 Observation resource serialization for vital signs monitoring.
* **Current Status:** **PARTIALLY COMPATIBLE (LEVEL 3 — IMPLEMENTED INTERFACE)**.
* **Evidence:**
  - Exposes working FastAPI REST endpoints (`/api/v1/telemetry/signals`, `/api/v1/fhir/Observation`).
  - Serializes observations to valid FHIR R4 JSON schemas with official LOINC coding (`8867-4` for Heart Rate, `2708-6` for Oxygen Saturation).
  - **Defect:** Ingests synthetic test subjects (`subj-01`) from `test-signals.csv` and emits them as genuine clinical Observations with status `final` and no `is_synthetic` extension tag.
* **Contract Maturity:** **LEVEL 3 (IMPLEMENTED INTERFACE)**.
* **Implementation Required:** **YES**. Add synthetic provenance extension to FHIR Observation serializer (`fhir_service.py`) and connect downstream consumer.

---

### [CONTRACT-DD-001] Shamit TB Drug Discovery Candidate Pipeline
* **Source Repository:** `drug-discovery--main`
* **Target Repository:** Downstream Biology & Safety Pipelines
* **Business / Technical Purpose:** Provide ranked TB candidate compounds, molecular identifiers (PubChem CID), targets, and physicochemical scoring to safety and preclinical evaluation engines.
* **Current Status:** **BLOCKED / NO CONTRACT (LEVEL 0)**.
* **Evidence:**
  - [generate_comparison.py:L3](file:///c:/Pratik_Bhuwad/Biotech/drug-discovery--main/generate_comparison.py#L3) crashes on root execution due to hardcoded relative path `../candidate_compounds.csv`.
  - [candidate_provenance.py:L7](file:///c:/Pratik_Bhuwad/Biotech/drug-discovery--main/candidate_provenance.py#L7) fails because prerequisite artifact `candidate_records_v1.json` is missing from the repository.
  - Zero automated tests exist to validate candidate data contracts.
* **Contract Maturity:** **LEVEL 0 (NO CONTRACT)**.
* **Implementation Required:** **YES**. Fix filesystem path references, supply missing canonical JSON dataset, and create automated test harness.

---

### [CONTRACT-PROV-001] System-Wide Synthetic Data Provenance Standard
* **Scope:** All six Biotech repositories
* **Business / Technical Purpose:** Guarantee that synthetic, mock, and generated test records can never be blended with or mistaken for genuine real-world evidence or clinical observations.
* **Current Status:** **PROPOSED — NOT APPROVED (LEVEL 1)**.
* **Evidence of Current Disparity:**
  - Shyamal enforces `synthetic_data_declaration` string enum (`"SAMPLE / SYNTHETIC / FOR SOFTWARE TESTING"` vs `"REAL / SOURCE-VERIFIED"`).
  - MHEALTH defaults `DatasetProvenance.anonymized = True` but has no synthetic flag, exporting synthetic subjects directly to standard FHIR.
  - Sarvesh and Sara use mock records in unit tests without explicit schema-level synthetic declaration flags.
* **Contract Maturity:** **LEVEL 1 (INFORMAL / PROPOSED)**.
* **Implementation Required:** **YES**. Team must approve a unified provenance specification (e.g. `is_synthetic: bool` or `synthetic_data_declaration: str`).

---

### [CONTRACT-ID-001] Global Cross-Repository Identifier Architecture
* **Scope:** All six Biotech repositories
* **Business / Technical Purpose:** Enable correlation and joins across repositories for identical drugs, studies, batches, and evidence items.
* **Current Status:** **NO CONTRACT ESTABLISHED (LEVEL 0)**.
* **Evidence of Current Disparity:**
  - Sarvesh: Zero-padded string `EV-000001` or `EV-<source_record_id>`.
  - Shravani: 3-digit string `EV-001` or `case_id`.
  - Sara: Unhyphenated string `E001` and DOI identifier `10.1016/...`.
  - Shamit: `C-001` and PubChem `CID: 2244`.
  - Shyamal: `SYN-001` / `EVID-001` and `STUDY-001`.
  - MHEALTH: SQLite auto-increment integer `id: 1` and `subj-01`.
* **Contract Maturity:** **LEVEL 0 (NO CONTRACT)**.
* **Implementation Required:** **YES**. Establish a namespaced identifier standard (e.g., `BIO:<DOMAIN>:<ENTITY>:<ID>`).

---

## 4. Contract Maturity Summary

```text
LEVEL 0 (NO CONTRACT): 3 Contracts (CONTRACT-FS-BIO-001, CONTRACT-DD-001, CONTRACT-ID-001)
LEVEL 1 (INFORMAL / DOCUMENTED ONLY): 2 Contracts (CONTRACT-PV-DS-001, CONTRACT-PROV-001)
LEVEL 2 (SCHEMA EXISTS): 0 Contracts
LEVEL 3 (IMPLEMENTED INTERFACE): 1 Contract (CONTRACT-MH-001)
LEVEL 4 (INTEGRATION TESTED): 0 Contracts
LEVEL 5 (REPRODUCIBLY VERIFIED): 0 Contracts
```

*Definitive Finding: The Biotech ecosystem currently possesses ZERO integration-tested (Level 4) or reproducibly verified (Level 5) cross-repository data contracts.*
