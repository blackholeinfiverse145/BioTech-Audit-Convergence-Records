# BIOTECH — DECISION VERIFICATION REGISTER
**Auditor:** Pratik Bhuvad  
**Role:** AI/ML + Data Audit Owner — Independent verification of reproducibility and analytical/data integrity  
**Date:** 2026-10-05  
**Status:** Phase 5 Decision & Contract Verification (Authoritative & Verified — READ-ONLY)

---

## 1. Executive Summary & Purpose

This register documents the formal verification of the seven major convergence decisions formulated in Phase 4. Each decision has been independently challenged and audited against direct executable evidence, source code, and configuration across the six Biotech repositories:
1. `Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main` (Sarvesh Patil — Task 1 PV Pipeline)
2. `bhiv-MHEALTH-Production-main` (MHEALTH Task 4 Team — Telemetry Microservice & FHIR Gateway)
3. `BHIV_Formulation_Stability_T3_Day1` (Sara — Formulation & Stability Intelligence)
4. `Drug_Safety_QC_Evidence_Map-main` (Shravani — Drug Safety Evidence Integration)
5. `drug-discovery--main` (Shamit / TMS — TB Drug-Discovery Candidate Intelligence)
6. `symmetrical-octo-invention-main` (Shyamal — Biosimilarity & Preclinical Evidence QC)

### Methodological Rule: Separation of Evidence Tiers
Every record strictly distinguishes:
* **FACT:** Directly observed source code, schema definition, file presence/absence, or execution output.
* **INFERENCE:** Logical deduction regarding architectural intent, failure propagation, or boundary compatibility.
* **PROPOSED DECISION:** Forward-looking recommendation requiring formal team approval before implementation.

---

## 2. Decision Verification Matrix

| Decision ID | Decision Title | Evidence Exists? | Existing Implementation | Contract Exists? | Tested? | Status | Implementation Required? |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **DEC-001** | D1: AI/ML Nomenclature Alignment | **YES** | Deterministic Rules Only (Zero ML) | **NO** | **YES** | **NOT SUPPORTED (NO ML PRESENT)** | **YES (If ML Mandated)** |
| **DEC-002** | D2: Shared Packaging & Namespace | **YES** | Divergent (Flat vs app/ vs src/) | **NO** | **NO** | **NOT YET APPROVED** | **YES** |
| **DEC-003** | D3: Task 1 $\to$ Task 3 Safety Contract | **YES** | Incompatible Schemas | **NO** | **YES** | **COMPATIBLE W/ TRANSFORMATION** | **YES** |
| **DEC-004** | D4: Shamit TB Discovery Remediation | **YES** | Blocked (Path defect & Missing JSON) | **NO** | **NO** | **REMEDIATION REQUIRED** | **YES** |
| **DEC-005** | D5: Shared QC / Entity Logic | **YES** | String Norm True; Conflict/QC Distinct | **NO** | **NO** | **PARTIAL JUSTIFICATION** | **YES** |
| **DEC-006** | D6: Synthetic Provenance Standard | **YES** | Text Enum (Shyamal) vs Unflagged (MHEALTH) | **NO** | **NO** | **PARTIAL STANDARD** | **YES** |
| **DEC-007** | D7: Service vs. Batch Architecture | **YES** | 1 Daemon (MHEALTH), 5 Batch/CLI/UI | **NO** | **NO** | **NO APPROVED TARGET ARCH** | **YES** |

---

## 3. Detailed Decision Verification Records

---

### [DEC-001] D1: AI/ML Nomenclature Alignment
* **Phase 4 Decision Question:** Does the team formally agree that deterministic, rule-based algorithms, threshold alerts, and heuristic scoring satisfy project deliverables, or must true machine-learning models (e.g., BioBERT, scikit-learn classifiers, molecular GNNs) be integrated?
* **Repository Evidence:**
  - **FACT:** Static code analysis across all `.py` files confirmed 0 imports of `torch`, `tensorflow`, `keras`, `sklearn`, `scikit-learn`, `xgboost`, `lightgbm`, `transformers`, `huggingface`.
  - **FACT:** Filesystem audit confirmed 0 model weight files (`.pt`, `.pth`, `.onnx`, `.joblib`, `.pkl`, `.bin`, `.safetensors`, `.h5`).
  - **FACT:** API inspection confirmed 0 integrations with OpenAI, Anthropic, Gemini, Claude, Ollama, or external model-serving endpoints.
  - **FACT:** In Sara's repository ([evidence_intelligence.py:L11](file:///c:/Pratik_Bhuwad/Biotech/BHIV_Formulation_Stability_T3_Day1/src/evidence_intelligence.py#L11)), `api_model` represents the "Active Pharmaceutical Ingredient Model" (e.g. Griseofulvin, Theophylline, Metronidazole), NOT an AI/ML API.
  - **FACT:** Pharmacovigilance uses dictionary/regex lookups; MHEALTH uses static vital sign thresholds ($HR > 100$); Drug Safety QC uses string normalizers; TB Discovery uses Lipinski arithmetic filters; Formulation Stability uses Arrhenius kinetic equations; Biosimilarity uses statistical tolerance intervals.
* **INFERENCE:** The phrase "AI-Biotech" in repository titles reflects future aspirational branding or high-level project taxonomy, rather than existing algorithmic implementation.
* **PROPOSED DECISION:** Formally designate current deliverables as "Deterministic Rule-Based & Statistical QC Engines". If genuine ML is required by stakeholders, a separate ML workstream must be scoped.
* **Verification Status:** **NOT SUPPORTED BY EXISTING IMPLEMENTATION (NO ML PRESENT)**.
* **Implementation Required:** **YES (If ML is mandated by team leadership)**.

---

### [DEC-002] D2: Shared Packaging Architecture / Namespace Standard
* **Phase 4 Decision Question:** Should the team standardize repositories around `pyproject.toml` manifests and `src/<namespace>` directory layouts?
* **Repository Evidence:**
  - **FACT:** 0 of 6 repositories contain a `pyproject.toml` or `setup.py` file.
  - **FACT:** Only 2 repositories have `requirements.txt`: [BHIV_Formulation_Stability_T3_Day1](file:///c:/Pratik_Bhuwad/Biotech/BHIV_Formulation_Stability_T3_Day1/requirements.txt) and [bhiv-MHEALTH-Production-main](file:///c:/Pratik_Bhuwad/Biotech/bhiv-MHEALTH-Production-main/requirements.txt).
  - **FACT:** 3 repositories have root module flattening ([Live-PV](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main), [Drug-Safety-QC](file:///c:/Pratik_Bhuwad/Biotech/Drug_Safety_QC_Evidence_Map-main), [drug-discovery](file:///c:/Pratik_Bhuwad/Biotech/drug-discovery--main)), requiring `PYTHONPATH` overrides or packaging shims to execute unit tests cleanly.
  - **FACT:** Sara uses `src/evidence_intelligence.py` with `pytest.ini` (`pythonpath = src`); Shyamal uses `qc_engine/engine.py` with `__init__.py`; MHEALTH uses `app/` subpackages.
  - **FACT:** No repository-level documentation or commit history references a team-approved namespace convention or standard packaging mandate.
* **INFERENCE:** Packaging disparities arose because each repository was developed as an independent greenfield prototype without centralized DevOps guidelines.
* **PROPOSED DECISION:** Adopt modern Python packaging standard: add `pyproject.toml` (Hatchling or Flit) and adopt `src/biotech/<capability>` namespacing across all six repositories.
* **Verification Status:** **NOT YET APPROVED / PROPOSED ONLY**.
* **Implementation Required:** **YES**.

---

### [DEC-003] D3: Task 1 Pharmacovigilance $\longrightarrow$ Task 3 Drug Safety Contract
* **Phase 4 Decision Question:** Shall Sarvesh's Task 1 pharmacovigilance output schema become the formal input schema for Shravani's Task 3 Drug Safety QC engine?
* **Repository Evidence:**
  - **FACT:** Sarvesh outputs [evidence_output.json](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main/evidence_output.json) containing `EvidenceRecord` objects with fields: `evidence_id`, `drug_name`, `drug_identifier`, `adverse_event`, `source`, `source_record_id`, `evidence_type`, `event_date`, `retrieved_at`, `source_url`, `schema_version`, `provenance`, `entity_resolution_status`.
  - **FACT:** Shravani's validator ([validate_evidence.py](file:///c:/Pratik_Bhuwad/Biotech/Drug_Safety_QC_Evidence_Map-main/validate_evidence.py)) requires: `evidence_id`, `source`, `drug_product`, `observation`, `evidence_type`, `evidence_provenance`.
  - **FACT:** Direct execution of `validate_record(sarvesh_record)` fails with 4 fatal validation errors:
    1. `MISSING_REQUIRED_FIELD:drug_product`
    2. `MISSING_REQUIRED_FIELD:observation`
    3. `INVALID_EVIDENCE_TYPE` (`"FAERS adverse-event report"` not in allowed enum)
    4. `MISSING_PROVENANCE` (`provenance` dict vs expected `evidence_provenance`)
  - **FACT:** Shravani's [canonical_schema.json](file:///c:/Pratik_Bhuwad/Biotech/Drug_Safety_QC_Evidence_Map-main/canonical_schema.json) requires 16 fields, of which 10 are completely missing from Sarvesh's output (`source_organisation`, `population_context`, `measurement_result`, `supporting_documentation`, `evidence_quality`, `validation_status`, `reviewer_action`, `human_review_flag`).
* **INFERENCE:** Direct coupling is impossible without an adapter layer. However, the data semantics are highly compatible; every missing field in Shravani can be deterministically mapped, derived, or defaulted from Sarvesh's output.
* **PROPOSED DECISION:** Implement `BiotechSafetyAdapter` (or `Task1ToTask3Mapper`) to convert `EvidenceRecord` into `BHIV_Drug_Safety_Evidence` canonical format.
* **Verification Status:** **COMPATIBLE WITH TRANSFORMATION — CANONICAL CONTRACT NOT YET ESTABLISHED**.
* **Implementation Required:** **YES**.

---

### [DEC-004] D4: Shamit TB Drug-Discovery Remediation
* **Phase 4 Decision Question:** Shall Shamit's repository be prioritized for immediate path refactoring, artifact restoration, and automated test creation?
* **Repository Evidence:**
  - **FACT:** [generate_comparison.py:L3](file:///c:/Pratik_Bhuwad/Biotech/drug-discovery--main/generate_comparison.py#L3) hardcodes `CSV_FILE = "../candidate_compounds.csv"`. Executing `python generate_comparison.py` crashes out-of-the-box with `FileNotFoundError: [Errno 2] No such file or directory: '../candidate_compounds.csv'`.
  - **FACT:** [calculate_scores.py:L3](file:///c:/Pratik_Bhuwad/Biotech/drug-discovery--main/calculate_scores.py#L3) also hardcodes `CSV_FILE = "../candidate_compounds.csv"`.
  - **FACT:** `candidate_compounds.csv` is present in the repository root ([candidate_compounds.csv](file:///c:/Pratik_Bhuwad/Biotech/drug-discovery--main/candidate_compounds.csv)), proving the `../` parent prefix is an execution path defect.
  - **FACT:** [candidate_provenance.py:L7](file:///c:/Pratik_Bhuwad/Biotech/drug-discovery--main/candidate_provenance.py#L7) attempts to load `INPUT_FILE = BASE_DIR / "candidate_records_v1.json"`. This file does not exist anywhere in the repository or workspace, causing the script to exit with `ERROR: candidate_records_v1.json not found`.
  - **FACT:** [calculate_scores_v2.py:L7](file:///c:/Pratik_Bhuwad/Biotech/drug-discovery--main/calculate_scores_v2.py#L7) depends on `candidate_records_with_provenance.json` (generated by `candidate_provenance.py`), causing it to fail with `ERROR: Input file not found`.
  - **FACT:** `pytest` discovers 0 tests in `drug-discovery--main`. There is no `tests/` directory.
* **INFERENCE:** The repository was checked into source control with scripts authored in a subfolder and incomplete JSON generation artifacts.
* **PROPOSED DECISION:** Approve remediation work package: (1) update relative paths to use `Path(__file__).parent`, (2) implement an ingestion script converting `candidate_compounds.csv` $\to$ `candidate_records_v1.json`, (3) author a unit test suite under `tests/`.
* **Verification Status:** **REMEDIATION REQUIRED (CONFIRMED DEFECTIVE BASELINE)**.
* **Implementation Required:** **YES**.

---

### [DEC-005] D5: Shared Entity Resolution, Conflict Detection, and QC Library
* **Phase 4 Decision Question:** Shall a single shared core library (e.g., `biotech-common-qc`) be created to eliminate duplicated entity normalization, duplicate detection, and conflict resolution logic?
* **Repository Evidence:**
  - **FACT:** Entity name normalization logic is character-for-character identical between Sarvesh ([entity_resolution.py:L57](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main/entity_resolution.py#L57)) and Shravani ([drug_matcher.py:L18](file:///c:/Pratik_Bhuwad/Biotech/Drug_Safety_QC_Evidence_Map-main/drug_matcher.py#L18)):
    `" ".join(str(name).strip().lower().split())`.
    $\longrightarrow$ **TRUE FUNCTIONAL DUPLICATION**.
  - **FACT:** Entity matching algorithms have identical three-tier logic (`MATCHED`/`EXACT_MATCH`, `AMBIGUOUS`/`AMBIGUOUS_MATCH`, `UNRESOLVED`/`UNMATCHED`), but operate on different data structures (Sarvesh uses `EntityCandidate` dataclasses; Shravani queries JSON dictionaries).
    $\longrightarrow$ **SIMILAR PURPOSE BUT DIFFERENT DATA STRUCTURES**.
  - **FACT:** Duplicate detection algorithms check completely different keys:
    * Sara checks `evidence_id` and `doi_identifier` / `citation`.
    * Shravani checks `evidence_id`, `source + source_identifier`, and `drug_product + observation`.
    * Shyamal checks `evidence_id` and `study_id`.
    $\longrightarrow$ **SIMILAR PURPOSE BUT DIFFERENT DOMAIN SEMANTICS**.
  - **FACT:** Conflict detection algorithms are completely distinct:
    * Shravani searches for clinical safety antonym word pairs ("signal present" vs "signal not observed").
    * Shyamal checks for discordant assay methodology reporting attributes (`measurement_method`, `control_identification`) within the same preclinical study ID.
    $\longrightarrow$ **SIMILAR PURPOSE BUT DIFFERENT ALGORITHMIC SEMANTICS**.
* **INFERENCE:** Merging all conflict and duplicate detection into a monolithic function would violate domain boundaries. However, consolidating string normalization, hash utilities, and generic key-based deduplication into a lightweight shared package is fully justified.
* **PROPOSED DECISION:** Author `biotech-common-core` containing string normalizers, hashing helpers, and base provenance schemas; keep domain conflict rules in respective capability modules.
* **Verification Status:** **CONSOLIDATION PARTIALLY JUSTIFIED (STRING NORMALIZATION ONLY)**.
* **Implementation Required:** **YES**.

---

### [DEC-006] D6: Synthetic Data Provenance Standard
* **Phase 4 Decision Question:** Shall Shyamal's strict synthetic data labeling standard be mandated across MHEALTH and all clinical export pipelines?
* **Repository Evidence:**
  - **FACT:** Shyamal ([engine.py:L67-70](file:///c:/Pratik_Bhuwad/Biotech/symmetrical-octo-invention-main/qc_engine/engine.py#L67-L70)) implements:
    `SYNTHETIC_DECLARATIONS = {"SAMPLE / SYNTHETIC / FOR SOFTWARE TESTING", "REAL / SOURCE-VERIFIED"}`
    Field `synthetic_data_declaration` is mandatory on every record.
  - **FACT:** Phase 4 reported Shyamal's field as `is_synthetic: true`. This was a conceptual summary; the actual implementation uses a text declaration enum.
  - **FACT:** MHEALTH ([dataset.py:L14](file:///c:/Pratik_Bhuwad/Biotech/bhiv-MHEALTH-Production-main/app/schemas/dataset.py#L14)) defaults `anonymized: bool = True`, but contains no `is_synthetic` or `data_nature` field.
  - **FACT:** MHEALTH ingests test telemetry (`subj-01`) from `test-signals.csv` and serializes them as genuine HL7 FHIR R4 Observations with status `final`, lacking synthetic tags or extension markers.
  - **FACT:** Sarvesh and Sara use mock records in unit tests (`FAERS-001`, `E001`) with no schema-level synthetic declaration requirements.
* **INFERENCE:** If MHEALTH FHIR exports are consumed by real-world clinical systems, artificial test fixtures could pollute medical records.
* **PROPOSED DECISION:** Mandate a dual synthetic provenance standard: (1) `is_synthetic: bool = False` on all core schemas, and (2) standard FHIR extension `http://biotech.org/fhir/StructureDefinition/is-synthetic` on all MHEALTH Observation exports.
* **Verification Status:** **PARTIAL STANDARD IN REPO 6 / INTEGRATION RISK IN REPO 2**.
* **Implementation Required:** **YES**.

---

### [DEC-007] D7: Service vs. Batch Runtime Architecture
* **Phase 4 Decision Question:** What is the target runtime convergence model: a distributed microservice network or a coordinated DAG orchestrator (e.g., Prefect / Airflow)?
* **Repository Evidence:**
  - **FACT:** MHEALTH is implemented as an asynchronous FastAPI microservice running under Uvicorn ASGI with SQLite persistence.
  - **FACT:** Sara's Formulation Stability is implemented as a standard library CLI batch script ([run.py](file:///c:/Pratik_Bhuwad/Biotech/BHIV_Formulation_Stability_T3_Day1/run.py)).
  - **FACT:** Shravani's Drug Safety QC is implemented as standalone batch Python scripts ([drug_safety_tracker.py](file:///c:/Pratik_Bhuwad/Biotech/Drug_Safety_QC_Evidence_Map-main/drug_safety_tracker.py)).
  - **FACT:** Shamit's TB Discovery is implemented as standalone CLI scripts and an interactive terminal menu ([drug_discovery_pipeline.py](file:///c:/Pratik_Bhuwad/Biotech/drug-discovery--main/drug_discovery_pipeline.py)).
  - **FACT:** Sarvesh's Task 1 is an OpenFDA batch retrieval and normalization pipeline ([app.py](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main/app.py)).
  - **FACT:** Shyamal's Biosimilarity is a standalone batch QC engine with CLI arguments ([engine.py](file:///c:/Pratik_Bhuwad/Biotech/symmetrical-octo-invention-main/qc_engine/engine.py)) and a separate Vite frontend.
  - **FACT:** 0 references to Airflow, Prefect, Celery, Kafka, or RabbitMQ exist in any repository code or configuration.
  - **FACT:** No architectural blueprint or specification document exists defining the multi-repository deployment model.
* **INFERENCE:** The six repositories were designed for fundamentally different execution contexts (real-time telemetry gateway vs. scientific research and QC batch pipelines).
* **PROPOSED DECISION:** Adopt a hybrid convergence model: preserve MHEALTH as an independent microservice; organize scientific evidence pipelines (Sarvesh $\to$ Shravani, Sara, Shamit, Shyamal) as modular workflow steps orchestrated by a lightweight runner.
* **Verification Status:** **NO APPROVED TARGET ARCHITECTURE — DECISION REQUIRED**.
* **Implementation Required:** **YES**.

---

## 4. Decision Verification Summary

```text
DECISIONS VERIFIED: 0
DECISIONS PARTIALLY VERIFIED: 3 (DEC-003, DEC-005, DEC-006)
DECISIONS NOT VERIFIED: 0
DECISIONS CONTRADICTED / NOT SUPPORTED: 4 (DEC-001, DEC-002, DEC-004, DEC-007)

TOTAL DECISIONS AUDITED: 7
IMPLEMENTATION REQUIRED: YES (All 7 decisions require technical implementation or governance approval)
```
