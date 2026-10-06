# BIOTECH — INDEPENDENT AI/ML + DATA AUDIT REPORT
**Auditor:** Pratik Bhuvad  
**Role:** AI/ML + Data Audit — Independent verification of reproducibility and analytical/data integrity  
**Date:** 2026-09-29  
**Core Question Investigated:** *Can another developer reproduce the same result from the supplied inputs?*

---

## 1. Executive Summary

An independent, read-only software, data, and analytical integrity audit was conducted across two repositories in the Biotech workspace:
1. `Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main` (Pharmacovigilance Pipeline v1)
2. `bhiv-MHEALTH-Production-main` (BHIV MHEALTH Production Runtime Task 4)

### Primary Audit Findings:
- **Zero Trained AI/ML Models:** Despite repository naming ("AI Biotech", "Health Intelligence"), neither repository contains machine learning models, statistical neural networks, or learned weights. Both systems are 100% deterministic, rule-based algorithmic pipelines (Repo 1 uses string normalization, dictionary mapping, and OpenFDA REST queries; Repo 2 uses hardcoded numeric threshold comparisons against a JSON configuration).
- **Repo 1 Packaging / Execution Blocker:** Direct execution of `pytest` and `python app.py` as documented in `FINAL_DELIVERABLE_REPORT.md` **fails immediately** with `ModuleNotFoundError: No module named 'src'`. All source and test files were flattened into the repository root, but import statements reference a non-existent `src.` package, and `pytest.ini` targets a non-existent `tests/` directory. When an external wrapper aliases `src`, all 47 unit/integration tests pass in 0.14s.
- **Repo 1 Normalization / Data Integrity Flaw:** In `normalize_evidence.py`, the pipeline unconditionally extracts the first drug listed (`patient.drug[0]`) from multi-drug FAERS safety reports. In multi-drug regimens, this causes adverse events to be attributed to the wrong drug, causing entity resolution against the queried drug to fail with `UNRESOLVED`.
- **Repo 2 Test Execution & Windows Compatibility Issues:** Direct execution of `pytest tests/test_task4.py` fails with 39 `TypeError` collection errors in environments with modern `httpx >= 0.28` due to Starlette 0.27 passing an obsolete `app` parameter. Additionally, unclosed SQLite connection handles in `app/database/db.py` lock the database file on Windows, triggering `PermissionError: [WinError 32]` during fixture teardown. When wrapped with compatibility shims, all 39 tests pass.
- **Repo 2 Ingestion Deduplication Gap:** The system generates a random `uuid.uuid4()` for every ingested signal row. Duplicate records within the same CSV, as well as repeat uploads of the same dataset, are unconditionally inserted as new signals without collision detection or deduplication.
- **Synthetic Data Labeling Gap:** Repo 2 includes `test-signals.csv` with artificial subjects (`subj-01`, `subj-02`), but the data schema lacks an `is_synthetic` or `data_nature` flag. The system maps test data directly to `WEARABLE_DEVICE` / `CLINICAL_MONITOR` and exports them as genuine LOINC-coded FHIR R4 Observations (`Patient/unknown`), allowing synthetic data to be silently presented as real-world clinical evidence.

---

## 2. Repositories Audited

| Repository Name | Directory | Primary Documented Purpose | Realized Technical Nature |
|---|---|---|---|
| **Repo 1** | `Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main` | Pharmacovigilance evidence and drug safety intelligence pipeline | CLI script retrieving OpenFDA/FAERS adverse events, normalising JSON, and storing deduplicated records in JSON. |
| **Repo 2** | `bhiv-MHEALTH-Production-main` | Health Signal Intelligence Platform (MHEALTH Task 4) | FastAPI web application for ingesting physiological health signals (CSV/JSON), classifying severity via threshold rules, and exporting FHIR R4 Observations. |

---

## 3. Capability Mapping

| Capability | Repo 1 (Pharmacovigilance) Status | Repo 2 (MHEALTH) Status | Evidence-Backed Finding |
|---|---|---|---|
| **External Data Ingestion** | PARTIALLY IMPLEMENTED (OpenFDA REST API) | NOT APPLICABLE | Repo 1 queries live OpenFDA endpoint; subject to remote network availability. |
| **File / Telemetry Ingestion** | NOT IMPLEMENTED | IMPLEMENTED (CSV & JSON) | Repo 2 ingests CSV/JSON multi-vital signals via `/dataset/upload`. |
| **Entity Resolution** | IMPLEMENTED (Exact string + alias match) | NOT APPLICABLE | Repo 1 normalises drug names; conservative matching (`MATCHED`, `AMBIGUOUS`, `UNRESOLVED`). |
| **Evidence Schema Validation** | IMPLEMENTED (Custom Dataclass validator) | IMPLEMENTED (Pydantic v2) | Repo 1 validates v1.1 schema; Repo 2 validates v2.0 signal schemas. |
| **Deduplication** | IMPLEMENTED (`source_record_id` + SHA-256) | DOCUMENTED ONLY / MISSING | Repo 1 deduplicates by ID or SHA-256 fingerprint; Repo 2 generates random UUIDs and never deduplicates. |
| **Rule / Classification Engine** | NOT IMPLEMENTED | IMPLEMENTED (Threshold JSON rules) | Repo 2 compares vitals against `thresholds.json` rules and generates `ThresholdTrace`. |
| **Audit Logging** | PARTIALLY IMPLEMENTED (Python logging) | IMPLEMENTED (Relational SQLite table) | Repo 2 writes audit events (`DATASET_UPLOAD`, `REPLAY`, `RUNTIME_START`) to `audit_log`. |
| **Replay Verification** | NOT IMPLEMENTED | IMPLEMENTED (`/replay/{id}`) | Repo 2 verifies classification determinism by replaying stored signals. |
| **Standards Export (FHIR)** | NOT IMPLEMENTED | IMPLEMENTED (FHIR R4 Observation + Bundle) | Repo 2 converts signals to LOINC-coded FHIR R4 Observations. |
| **Observability / Metrics** | PARTIALLY IMPLEMENTED (logger events) | IMPLEMENTED (Prometheus text `/metrics`) | Repo 2 exposes Prometheus metrics counters and web dashboard. |
| **Web UI / Dashboard** | NOT IMPLEMENTED | IMPLEMENTED (HTML at `/dashboard`) | Repo 2 includes HTML monitoring UI with auto-refresh. |

---

## 4. Repository Execution Maps

### Repository 1: Pharmacovigilance Pipeline
```text
OpenFDA API / Query Input ("ibuprofen")
    ↓
openfda_connector.py (search_drug)
    ↓ [Raw FAERS JSON response]
normalize_evidence.py (normalize_record)
    ↓ [Extract drug & reaction, resolve entity]
entity_resolution.py (resolve_entity)
    ↓ [EvidenceRecord dataclass]
evidence_schema.py (validate_evidence)
    ↓ [Validated record dictionary]
evidence_store.py (add_evidence)
    ↓ [Deduplicate via source_record_id or SHA-256]
evidence_output.json (Persistent JSON store)
```

#### Top 3 Core Execution Files (Repo 1):
1. [openfda_connector.py](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main/openfda_connector.py): **Data Ingestion Boundary.** Manages HTTP communication with the external OpenFDA/FAERS REST API (`https://api.fda.gov/drug/event.json`), handles pagination/limits, formats query parameters, and captures ISO-8601 retrieval timestamps.
2. [normalize_evidence.py](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main/normalize_evidence.py): **Core Data Transformation & Integrity Engine.** Bridges unstructured external FAERS JSON into structured `EvidenceRecord` instances, assigns provenance metadata, and invokes entity resolution. Contains critical analytical finding regarding `drugs[0]` index selection.
3. [evidence_store.py](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main/evidence_store.py): **Persistence & Deduplication Engine.** Implements idempotent storage logic using primary `source_record_id` tracking and secondary SHA-256 canonical JSON fingerprinting over 7 payload fields.

---

### Repository 2: BHIV MHEALTH Production Runtime
```text
HTTP Request (CSV / JSON payload + X-API-Key)
    ↓
app/main.py (FastAPI Routes & Auth Middleware)
    ↓
app/ingestion/ingest.py (ingest_file / _build)
    ↓ [HealthSignalInput Pydantic models]
app/classifier/engine.py (classify) ← configs/thresholds.json
    ↓ [ClassificationResult + ThresholdTrace]
app/database/db.py (insert_signal, insert_classification, audit)
    ↓ [SQLite bhiv_mhealth.db / PostgreSQL]
app/fhir/export.py (signal_to_fhir, build_bundle)
    ↓ [FHIR R4 Observation JSON]
Client Response / Dashboard / Prometheus Metrics
```

#### Top 3 Core Execution Files (Repo 2):
1. [app/main.py](file:///c:/Pratik_Bhuwad/Biotech/bhiv-MHEALTH-Production-main/app/main.py): **Application Orchestration & Security Gateway.** Wires FastAPI endpoints, enforces `X-API-Key` authentication middleware, manages system lifespan (`init_db`), logs audit traces, and increments observability metrics.
2. [app/classifier/engine.py](file:///c:/Pratik_Bhuwad/Biotech/bhiv-MHEALTH-Production-main/app/classifier/engine.py): **Deterministic Rule Evaluation Engine.** Evaluates 11 physiological modalities against rules in `configs/thresholds.json`, returning structured `ParticipationOutcome` (`ACCEPTED`, `FLAGGED`), `SeverityLevel` (`NORMAL`, `BORDERLINE`, `ELEVATED`, `CRITICAL`), and `ThresholdTrace` audit items.
3. [app/database/db.py](file:///c:/Pratik_Bhuwad/Biotech/bhiv-MHEALTH-Production-main/app/database/db.py): **Relational Persistence Layer.** Defines schemas and handles parameterized SQL queries across 6 tables (`datasets`, `signals`, `classifications`, `replays`, `audit_log`, `metrics`). Contains critical Windows connection leak finding.

---

## 5. Data Audit

### Schema Analysis
#### Repo 1 (`EvidenceRecord`):
- **Fields:** `evidence_id` (str), `drug_name` (Optional[str]), `drug_identifier` (Optional[str]), `adverse_event` (Optional[str]), `source` (str), `source_record_id` (Optional[str]), `evidence_type` (str), `event_date` (Optional[str]), `retrieved_at` (str), `source_url` (str), `schema_version` (str, "1.1"), `provenance` (Dict), `raw_record_reference` (Optional[str]), `entity_resolution_status` (str), `canonical_entity_name` (Optional[str]), `canonical_entity_identifier` (Optional[str]), `association_type` (str).
- **Validation Strictness:** Schema requires `evidence_id`, `source`, `evidence_type`, `retrieved_at`, `source_url`, `schema_version`, valid `entity_resolution_status`, and `association_type == "REPORTED_ASSOCIATION"`.
- **Validation Gap:** `drug_name`, `adverse_event`, and `source_record_id` are completely optional. An empty record with `drug_name=None` and `adverse_event=None` passes validation without error.

#### Repo 2 (`HealthSignalInput`):
- **Fields:** `signal_id` (UUID), `schema_version` ("2.0"), `signal_type` (Enum), `source` (Enum), `recorded_at` (datetime), `ingested_at` (Optional[datetime]), modality-specific payloads (`heart_rate`, `spo2`, `respiratory`, `glucose`, `blood_pressure`, `temperature`, `activity`, `mobility`, `prosthetic_event`, `lab_result`, `research_signal`), `provenance` (`DatasetProvenance`), `health_context` (`HealthContext`), `tags` (list[str]).
- **Validation Strictness:** Uses Pydantic v2 with strict range validators (e.g. Heart rate BPM bounded `0 <= bpm <= 350`; SpO2 bounded `0 <= percentage <= 100`). Enforces modality-specific payload presence via `@model_validator(mode="after")`.

### Data Integrity & Provenance Verification
| Attribute | Repo 1 (Pharmacovigilance) | Repo 2 (MHEALTH) |
|---|---|---|
| **Source System Recorded** | Yes (`provenance.source_system = "OpenFDA"`) | Yes (`source` enum, e.g. `WEARABLE_DEVICE`) |
| **Source Record Identifier** | Yes (`source_record_id`, e.g. `"10003301"`) | No (Original row ID discarded; random UUID generated) |
| **Retrieval Timestamp** | Yes (`retrieved_at` ISO-8601 UTC) | Yes (`ingested_at` + `recorded_at` ISO-8601 UTC) |
| **Transformation History** | Yes (`"raw JSON -> normalized evidence"`) | Partial (Trace rules recorded, but raw ingestion diff not logged) |
| **Duplicate Prevention** | Strong (Rejects existing `source_record_id` or SHA-256 fingerprint) | None (Duplicate rows or repeated uploads create new UUIDs) |
| **Multi-Entity Disambiguation** | Flawed (Takes `drugs[0]` blindly from FAERS response) | N/A (Processes single telemetry record per row) |

---

## 6. Synthetic Data Audit

### Audit Criterion:
*Synthetic/Test Data → Explicitly labelled → Remains identifiable → Not silently presented as real-world evidence.*

### Findings:
1. **Repo 1 (`evidence_output.json`):**
   - The persistent file `evidence_output.json` contains 15 FAERS records retrieved on `2026-08-31` from OpenFDA. These appear to be genuine OpenFDA records.
   - However, the schema lacks an `is_synthetic` or `is_test` boolean flag. In unit tests (`test_evidence_store.py`), mock records (`FAERS-001`, `E2E001`) are constructed. If an operator or test runs `add_evidence()` without monkeypatching, test records would be permanently blended into `evidence_output.json` without any labeling.
2. **Repo 2 (`test-signals.csv`):**
   - The repository provides `test-signals.csv` containing artificial records:
     ```csv
     signal_type,source,bpm,recorded_at,subject_id
     heart_rate,wearable,72,2024-01-01T08:00:00,subj-01
     heart_rate,clinical,180,2024-01-01T08:05:00,subj-02
     ```
   - **Critical Integrity Defect:** When uploaded, `subj-01` and `subj-02` are ingested as real `WEARABLE_DEVICE` and `CLINICAL_MONITOR` data. The `DatasetProvenance` schema default is `anonymized: True`, but contains no `is_synthetic` field.
   - **Downstream Clinical Presentation:** The FHIR export converts this test signal directly into an official HL7 FHIR R4 Observation resource with LOINC code `8867-4` ("Heart rate") and category `vital-signs`.
   - **Conclusion:** Synthetic test data is **silently presented as real-world physiological evidence**, violating analytical integrity standards.

---

## 7. AI / ML Audit

### Audit Scope:
*Model loading, inputs, preprocessing, feature generation, inference, post-processing, thresholds, scoring, determinism, random seeds.*

### Factual Technical Findings:
- **Repository 1:**
  - **Machine Learning Components:** None.
  - **Underlying Logic:** Purely deterministic string matching in [entity_resolution.py](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main/entity_resolution.py). `normalize_entity_name()` applies lowercase conversion, whitespace collapsing, and string strip. Matching evaluates exact canonical equality or alias membership.
  - **Scoring / Probabilities:** None. Output status is categorical: `MATCHED`, `AMBIGUOUS`, or `UNRESOLVED`.
- **Repository 2:**
  - **Machine Learning Components:** None.
  - **Underlying Logic:** Purely deterministic expert-system threshold comparator in [app/classifier/engine.py](file:///c:/Pratik_Bhuwad/Biotech/bhiv-MHEALTH-Production-main/app/classifier/engine.py). It reads boundary parameters from `configs/thresholds.json` (e.g. `heart_rate.bpm < critical_low`) and returns fixed categorical tuples:
    `ParticipationOutcome.FLAGGED, SeverityLevel.CRITICAL, "HR 180 bpm critical."`
  - **Scoring / Weights:** None. No probabilistic models, gradient boosted trees, or neural networks exist in either repository.
- **Scientific Audit Guardrail:** As mandated, this audit makes no claims regarding medical efficacy or clinical safety of the thresholds defined in `thresholds.json`.

---

## 8. Determinism Test

### Test Protocol:
$$\text{Same Input} + \text{Same Configuration} \to \text{Same Output}$$
Controlled execution was performed using `scratch/test_audit_suite.py` on both repositories.

### Results:

#### Repo 1 (Pharmacovigilance Normalizer):
- Input: Identical OpenFDA FAERS JSON payload (`safetyreportid: TEST_AUDIT_001`, drug `IBUPROFEN`, reaction `Headache`).
- Execution 1 vs Execution 2:
  - `evidence_id`: `EV-000001` vs `EV-000001` (MATCH)
  - `canonical_entity_name`: `IBUPROFEN` vs `IBUPROFEN` (MATCH)
  - `provenance`: Identical dictionary (MATCH)
  - Content Hash / Fingerprint: Identical SHA-256 (MATCH)
  - **Result:** **100% DETERMINISTIC.**

#### Repo 2 (MHEALTH Classification Engine):
- Input: Identical `HealthSignalInput` (`signal_type: HEART_RATE`, `bpm: 72.0`, `source: WEARABLE_DEVICE`).
- Execution 1 vs Execution 2:
  - `outcome`: `ACCEPTED` vs `ACCEPTED` (MATCH)
  - `severity`: `NORMAL` vs `NORMAL` (MATCH)
  - `threshold_traces`: Identical list of rule evaluations (MATCH)
  - `classified_at`: Expected execution timestamp (metadata only)
  - **Result:** **100% DETERMINISTIC.**

---

## 9. Failure Testing

Controlled failure testing was executed across both repositories covering the 7 mandatory scenarios.

| Test Scenario | Repo 1 (Pharmacovigilance) Behaviour | Repo 2 (MHEALTH) Behaviour | Comparative Assessment |
|---|---|---|---|
| **1. Valid Input** | Normalizes and validates successfully; returns `valid=True`, `errors=[]`. | Ingests and classifies successfully; returns `status_code=200`, `outcome="ACCEPTED"`, `severity="NORMAL"`. | Both succeed with valid input. |
| **2. Missing Required Field** | Missing `source`: Rejected (`Missing source`). Missing `provenance`: Rejected. Missing `drug_name`: **Accepted** (validation passes). | Missing `signal_type`: HTTP 422 Unprocessable Entity (`Field required`). Missing modality payload: HTTP 422 (`signal_type requires matching payload`). | Repo 2 enforces stricter payload completeness than Repo 1. |
| **3. Invalid Value** | Invalid `entity_resolution_status`: Rejected. Invalid `association_type`: Rejected. | BPM = 9999 (>350): HTTP 422 (`Input should be <= 350`). Unknown `signal_type`: HTTP 422. | Both reject out-of-spec enum and range values cleanly. |
| **4. Duplicate Record** | First record added: `added=1`. Repeat insertion: `added=0`. Deduplication verified. | First upload: `success=2`. Repeat upload: `success=2`, new `dataset_id` generated, signals re-inserted. Duplicate rows in same CSV: both inserted. | **Repo 1 has duplicate protection; Repo 2 completely lacks duplicate protection.** |
| **5. Conflicting Record** | Ingestion of same `source_record_id` with different adverse event: **Rejected** (`added=0`) due to source ID collision. | Two signals for same `subject_id` at same timestamp with conflicting BPM (50 vs 190): **Both accepted** without conflict resolution. | Repo 1 enforces source identity; Repo 2 allows contradictory states for the same subject. |
| **6. Empty Dataset** | `normalize_record({})`: Returns empty record with `None` fields. Validation **passes** because drug/reaction are optional. | Empty file upload: HTTP 400 Bad Request (`detail: "Empty file."`). | Repo 2 handles empty input at API boundary; Repo 1 allows empty records to pass schema validation. |
| **7. Malformed Input** | String passed to normalizer: Raises clean `ValueError: raw_record must be a dictionary`. | Malformed JSON: HTTP 422 (`JSON decode error`). Malformed CSV (non-numeric BPM): HTTP 200 upload summary with row error recorded (`could not convert string to float`). | Both repositories handle malformed syntax without unhandled server crashes. |

---

## 10. Test Audit

### Repository 1: Pharmacovigilance Pipeline
- **Existing Test Files:** 8 files (`test_e2e_pipeline.py`, `test_entity_resolution.py`, `test_error_handler.py`, `test_evidence_schema.py`, `test_evidence_store.py`, `test_monitoring.py`, `test_normalization.py`, `test_phase4.py`).
- **Direct Execution (`pytest -q`):**
  - Command: `pytest`
  - Result: **FAILED — 0 passed, 8 errors (Collection Failure).**
  - Blocker: `ModuleNotFoundError: No module named 'src'`.
  - Additional Warning: `PytestConfigWarning: No files were found in testpaths; consider removing or adjusting your testpaths configuration` (due to `pytest.ini` expecting `tests/`).
- **Wrapped Execution (with `src` namespace aliased):**
  - Command: `python -c "... pytest.main([repo, '-v'])"`
  - Total Tests: 47
  - Passed: **47**
  - Failed: 0
  - Execution Time: 0.14s

### Repository 2: BHIV MHEALTH Production Runtime
- **Existing Test Files:** 1 file (`tests/test_task4.py`).
- **Direct Execution (`python -m pytest tests/test_task4.py -v`):**
  - Command: `python -m pytest tests/test_task4.py -v`
  - Result: **FAILED — 39 errors.**
  - Blocker 1: Incompatibility between Starlette 0.27 `TestClient` and `httpx >= 0.28` (`TypeError: Client.__init__() got an unexpected keyword argument 'app'`).
  - Blocker 2 (Windows OS specific): Unclosed SQLite connections in `app/database/db.py` hold file handles open, causing `test_db.unlink()` in test fixture teardown to crash with `PermissionError: [WinError 32]`.
- **Wrapped Execution (with httpx compatibility shim and connection closing):**
  - Command: `python scratch/run_bhiv_tests.py`
  - Total Tests: 39
  - Passed: **39**
  - Failed: 0
  - Execution Time: 3.29s

---

## 11. Evidence Audit

| Claim / Capability | Claimed In | Actual Evidence Type | Status | Evidence Detail |
|---|---|---|---|---|
| Repo 1: 47 Tests Pass | `FINAL_DELIVERABLE_REPORT.md` | TEST / EXECUTION | PARTIALLY VERIFIED | Fails on clean run due to import structure; passes 47/47 when `src` is aliased. |
| Repo 1: Duplicate Prevention | `REVIEW_INDEX.md` | EXECUTION / OUTPUT | VERIFIED | `test_evidence_store.py` and scratch failure test confirm duplicate rejection. |
| Repo 1: Runtime OpenFDA Retrieval | `FINAL_DELIVERABLE_REPORT.md` | EXECUTION | PARTIALLY VERIFIED | Live query works via requests, but direct CLI execution fails on import error. |
| Repo 2: 39 Tests Pass | `README.md`, `task4_review_packet.md` | TEST / EXECUTION | PARTIALLY VERIFIED | Fails on Windows / modern httpx; passes 39/39 with compatibility wrapper. |
| Repo 2: FHIR R4 LOINC Export | `task4_review_packet.md` | CODE / TEST | VERIFIED | `/fhir/observation/{id}` outputs valid LOINC `8867-4` JSON structure. |
| Repo 2: Replay Determinism | `README.md` | CODE / TEST | VERIFIED | `/replay/{id}` successfully recalculates rules and verifies matching outcome. |
| Repo 2: Ingestion Deduplication | Implied by "Production DB" | CODE / EXECUTION | **REFUTED** | Code inspection and failure test 4 prove duplicate rows are inserted without check. |

---

## 12. Reproducibility Assessment

> **Key Audit Question:** Can another developer reproduce the same result using only the repository, documented dependencies, supplied inputs, and documented execution procedure?

### Repository 1 Assessment:
- **Status:** **BLOCKED**
- **Blocker:** Following the documented instructions (`pytest -q` or `python app.py`) immediately throws `ModuleNotFoundError: No module named 'src'`.
- **Reproduction Prerequisites:**
  - Python 3.10+
  - Dependencies: `requests`, `pytest` (no `requirements.txt` was provided in Repo 1).
  - Directory restructuring: Files must be moved into a `src/` directory or an explicit `PYTHONPATH` alias must be configured.

### Repository 2 Assessment:
- **Status:** **PARTIALLY REPRODUCED**
- **Blocker:** Following documented instructions (`pip install -r requirements.txt; python -m pytest tests/test_task4.py -v`) on Windows with unpinned transitive httpx fails on collection and teardown.
- **Reproduction Prerequisites:**
  - Python 3.11 / 3.12
  - Dependencies: `requirements.txt` with `httpx==0.27.0` explicitly pinned.
  - OS Requirement: Must be executed on Linux/POSIX (or inside Docker) to avoid Windows SQLite file locking during test database unlinking.

---

## 13. Output Integrity

### Repo 1 Output (`evidence_output.json`):
- **Format:** JSON array of objects.
- **Schema Compliance:** Consistent field names (`evidence_id`, `drug_name`, `adverse_event`, `source`, `source_record_id`, `provenance`).
- **Consumption Readiness:** Readily parsable by downstream tools, but consumers must handle `drug_name: null` and `adverse_event: null` because schema validation does not require them.

### Repo 2 Output (REST Responses, FHIR R4, Audit Log):
- **Format:** JSON responses adhering to Pydantic models.
- **Standards Compliance:** FHIR export strictly adheres to HL7 FHIR R4 Observation resource definitions with standard LOINC codes and UCUM units.
- **Contract Clarity:** Documented via interactive OpenAPI Swagger UI at `/docs`.

---

## 14. Integration Readiness

### Repository 1: **BLOCKED**
- **Input Interface:** CLI `input()` prompt; no programmatic HTTP/gRPC API.
- **Output Interface:** JSON file dump (`evidence_output.json`).
- **Blockers for Integration:**
  1. No package structure (`src/` import crash).
  2. Hardcoded relative output path `EVIDENCE_FILE = "evidence_output.json"` prevents concurrent multi-threaded execution or deployment as a shared library.
  3. External unauthenticated network dependency on OpenFDA API without local fallback caching.

### Repository 2: **PARTIALLY INTEGRATION READY**
- **Input Interface:** Authenticated REST API (`POST /dataset/upload`, `POST /signal/classify`).
- **Output Interface:** JSON, FHIR R4 Observation/Bundle (`GET /fhir/observation/{id}`), Prometheus text (`/metrics`).
- **Security:** `X-API-Key` authentication middleware.
- **Blockers for Integration:**
  1. Ingestion lacks duplicate prevention.
  2. Windows SQLite database locking prevents clean automated CI/CD runs on Windows runners.
  3. No rate limiting or batching controls on dataset uploads.

---

## 15. Cross-Repository Findings & Convergence Opportunities

### 1. Independent Implementations:
- Repo 1 independently implements pharmacovigilance data retrieval and entity resolution.
- Repo 2 independently implements multi-vital signal classification, replay auditing, and FHIR export.

### 2. Overlapping & Duplicated Capabilities:
- Both repositories implement custom provenance schemas that capture similar metadata under differing keys:
  - Repo 1: `provenance.source_system`, `underlying_dataset`, `retrieval_method`, `transformation`.
  - Repo 2: `provenance.dataset_name`, `dataset_version`, `dataset_url`, `institution`, `license`.
- Both repositories implement monitoring/observability through disjoint channels (Repo 1 uses standard python logging; Repo 2 uses SQLite audit logging + Prometheus metrics).

### 3. Convergence Opportunities (Future Shared Services):
- **Unified Provenance & Deduplication Service:** Port Repo 1's deterministic SHA-256 fingerprinting logic into Repo 2's signal ingestion pipeline to solve Repo 2's duplicate vulnerability.
- **Standardized FHIR Ingestion & Export:** Extend Repo 2's FHIR export module to convert Repo 1's drug adverse events into HL7 FHIR `AdverseEvent` resources.
- **Shared Audit Trail:** Route Repo 1's pipeline events into Repo 2's `audit_log` relational table.

---

## 16. Required Status Classification Summary

```text
================================================================================
CAPABILITY EVALUATION STATUS REGISTER
================================================================================
Repo 1 - OpenFDA Evidence Retrieval:    PARTIALLY IMPLEMENTED (Network dependent)
Repo 1 - Entity Resolution:            REPRODUCIBLE (Algorithmic string match)
Repo 1 - Evidence Normalization:        REPRODUCIBLE (Contains multi-drug index flaw)
Repo 1 - Evidence Deduplication:        REPRODUCIBLE (SHA-256 & Source ID)
Repo 1 - Pipeline CLI (app.py):         BLOCKED (ModuleNotFoundError: 'src')
Repo 1 - Test Suite:                   BLOCKED out-of-box (Passes 47/47 when wrapped)
Repo 1 - Overall Integration:           BLOCKED

Repo 2 - Multi-Signal Ingestion:       REPRODUCIBLE (CSV & JSON)
Repo 2 - Threshold Classification:     REPRODUCIBLE (Deterministic rules)
Repo 2 - Signal Replay Service:        REPRODUCIBLE (Verified determinism)
Repo 2 - FHIR R4 Export:               REPRODUCIBLE (LOINC coded)
Repo 2 - Audit Logging & Metrics:      REPRODUCIBLE (SQLite & Prometheus)
Repo 2 - Ingestion Deduplication:      DOCUMENTED ONLY (Not implemented)
Repo 2 - Test Suite:                   PARTIALLY REPRODUCIBLE (Windows/httpx blocked)
Repo 2 - Overall Integration:          PARTIALLY INTEGRATION READY
================================================================================
```

---

## 17. Findings Register

### Finding AUDIT-PV-001: Missing Package Hierarchy (`src`)
- **Repository:** Repo 1 (Pharmacovigilance Pipeline)
- **Capability:** Pipeline Execution & Testing
- **Category:** RUNTIME / REPRODUCIBILITY
- **Observation:** Direct execution of `python app.py` and `pytest` fails immediately with `ModuleNotFoundError: No module named 'src'`.
- **Evidence:** Terminal execution outputs `ModuleNotFoundError: No module named 'src'`. All source code files reside in the root directory while importing from `src.*`.
- **Impact:** Any developer cloning the repository cannot run the pipeline or execute the test suite without altering the repository structure or setting external environment variables.
- **Reproduction:** Run `pytest` or `python app.py` in the root of Repo 1.
- **Expected:** Pipeline executes or tests run cleanly out of the box as claimed in `FINAL_DELIVERABLE_REPORT.md`.
- **Actual:** Execution terminates with exit code 1.
- **Status:** BLOCKED
- **Recommended Action:** Place all source modules inside a `src/` directory, or update all import statements to root-relative imports.

---

### Finding AUDIT-PV-002: Multi-Drug Regimen Entity Resolution Misattribution
- **Repository:** Repo 1 (Pharmacovigilance Pipeline)
- **Capability:** Evidence Normalization
- **Category:** DATA / AI/ML
- **Observation:** Normalizer selects `drugs[0]` unconditionally from OpenFDA FAERS patient drug arrays.
- **Evidence:** [normalize_evidence.py](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main/normalize_evidence.py#L51-L57):
  ```python
  if drugs:
      first_drug = drugs[0]
      if isinstance(first_drug, dict):
          drug_name = first_drug.get("medicinalproduct")
  ```
  In `evidence_output.json`, record `EV-000002` resulted from an "ibuprofen" search, but was assigned `drug_name="BENLYSTA"` and marked `entity_resolution_status="UNRESOLVED"` because Benlysta appeared first in the patient's multi-drug report.
- **Impact:** Adverse events are attributed to the wrong drug in multi-drug therapy cases, corrupting safety evidence data integrity.
- **Reproduction:** Inspect record `EV-000002` in `evidence_output.json`.
- **Expected:** Normalizer iterates through `patient.drug` to find the drug matching the search query.
- **Actual:** Normalizer selects index 0, misattributing the drug and failing entity resolution.
- **Status:** IMPLEMENTED (Flawed analytical logic)
- **Recommended Action:** Search the `drugs` list for a match with `requested_drug_name` before falling back to index 0, or record all concomitant medications.

---

### Finding AUDIT-PV-003: Permissive Schema Allows Empty Drug and Reaction Fields
- **Repository:** Repo 1 (Pharmacovigilance Pipeline)
- **Capability:** Evidence Schema Validation
- **Category:** SCHEMA / VALIDATION
- **Observation:** Validation function `validate_evidence()` does not require `drug_name` or `adverse_event`.
- **Evidence:** In `evidence_schema.py`, lines 58-126 validate `evidence_id`, `source`, `evidence_type`, `provenance`, etc., but omit checks for `record.drug_name` and `record.adverse_event`. Controlled test in `test_audit_suite.py` passed validation with both fields set to `None`.
- **Impact:** Empty records without any drug name or adverse event can be successfully validated and persisted into `evidence_output.json`.
- **Reproduction:** Execute `validate_evidence(EvidenceRecord(..., drug_name=None, adverse_event=None))`.
- **Expected:** Validation rejects records lacking primary clinical evidence fields.
- **Actual:** Returns `valid=True`, `errors=[]`.
- **Status:** IMPLEMENTED
- **Recommended Action:** Add mandatory non-empty checks for `drug_name` and `adverse_event` in `validate_evidence()`.

---

### Finding AUDIT-MH-001: Transitive HTTPX Incompatibility with Starlette TestClient
- **Repository:** Repo 2 (BHIV MHEALTH)
- **Capability:** Automated Testing
- **Category:** TESTING / REPRODUCIBILITY
- **Observation:** `pytest tests/test_task4.py` fails with 39 `TypeError` errors when `httpx >= 0.28.0` is installed.
- **Evidence:** Starlette 0.27.0 `TestClient` invokes `super().__init__(app=self.app, ...)`. In `httpx 0.28.0+`, the `app` keyword argument was removed from `Client.__init__()`.
- **Impact:** Standard test execution fails immediately on fresh installations where `httpx` is resolved to latest version.
- **Reproduction:** Run `pytest tests/test_task4.py` in an environment with `httpx 0.28.1` and `starlette 0.27.0`.
- **Expected:** All 39 tests collect and execute cleanly.
- **Actual:** Test collection crashes with `TypeError: Client.__init__() got an unexpected keyword argument 'app'`.
- **Status:** BLOCKED
- **Recommended Action:** Pin `httpx==0.27.0` strictly in `requirements.txt` or upgrade `starlette`/`fastapi`.

---

### Finding AUDIT-MH-002: SQLite Connection Handle Leaks Lock Database on Windows
- **Repository:** Repo 2 (BHIV MHEALTH)
- **Capability:** Database & Testing Layer
- **Category:** RUNTIME / TESTING
- **Observation:** Test fixture teardown fails on Windows with `PermissionError: [WinError 32]`.
- **Evidence:** In [app/database/db.py](file:///c:/Pratik_Bhuwad/Biotech/bhiv-MHEALTH-Production-main/app/database/db.py#L18-L28), `with get_conn() as conn:` handles transactions but does not call `conn.close()`. On Windows, open SQLite handles prevent `test_db.unlink()` from deleting the test database file during fixture cleanup.
- **Impact:** Automated regression tests cannot execute to completion on Windows platforms without raising teardown errors.
- **Reproduction:** Run tests on Windows using standard `pytest`.
- **Expected:** Fixture tears down and unlinks temporary database cleanly.
- **Actual:** Throws `PermissionError: [WinError 32] The process cannot access the file because it is being used by another process`.
- **Status:** BLOCKED (on Windows)
- **Recommended Action:** Ensure all SQLite database connections are explicitly closed after query execution or provide a connection teardown pool.

---

### Finding AUDIT-MH-003: Ingestion Lacks Signal Deduplication
- **Repository:** Repo 2 (BHIV MHEALTH)
- **Capability:** Data Ingestion
- **Category:** DATA / INTEGRITY
- **Observation:** System generates a new `uuid.uuid4()` for every ingested signal row and does not detect duplicate records.
- **Evidence:** In [app/ingestion/ingest.py](file:///c:/Pratik_Bhuwad/Biotech/bhiv-MHEALTH-Production-main/app/ingestion/ingest.py#L37-L90), `_build()` does not check existing records and assigns a fresh UUID. Controlled failure testing confirmed that duplicate rows within the same CSV, as well as uploading the exact same CSV twice, inserted all records repeatedly.
- **Impact:** Repeated sensor telemetry transmissions or re-uploaded datasets lead to database bloat and duplicated downstream alerts.
- **Reproduction:** POST `test-signals.csv` twice to `/dataset/upload`. Both requests return `success: 2`.
- **Expected:** Duplicate records or repeated datasets should be identified and deduplicated or rejected.
- **Actual:** All duplicate rows are stored as distinct signals.
- **Status:** DOCUMENTED ONLY
- **Recommended Action:** Generate a deterministic signal hash based on `(subject_id, recorded_at, signal_type, source, payload_hash)` and enforce unique indexing.

---

### Finding AUDIT-MH-004: Synthetic Data Silently Presented as Real Clinical Evidence
- **Repository:** Repo 2 (BHIV MHEALTH)
- **Capability:** Data Ingestion & FHIR Export
- **Category:** DATA / PROVENANCE
- **Observation:** Test signals (`subj-01`, `subj-02`) lack an `is_synthetic` flag and are exported as clinical FHIR R4 Observations.
- **Evidence:** `DatasetProvenance` model lacks synthetic indicators. `test-signals.csv` maps to `source: "wearable"` (`WEARABLE_DEVICE`). When exported via `/fhir/observation/{id}`, it generates a FHIR Observation with status `final` and standard LOINC code without disclosing that the data is synthetic.
- **Impact:** Downstream clinical aggregators cannot distinguish synthetic test signals from real patient vitals.
- **Reproduction:** Ingest `test-signals.csv` and inspect `GET /fhir/observation/{signal_id}`.
- **Expected:** Synthetic data must be explicitly labeled and carry an `is_synthetic: true` extension in FHIR exports.
- **Actual:** Exported as an ordinary vital signs observation with `subject: "Patient/unknown"`.
- **Status:** PARTIALLY IMPLEMENTED
- **Recommended Action:** Add `is_synthetic: bool = False` to `DatasetProvenance` and include a corresponding FHIR extension.

---

## 18. Disagreement Protocol

```text
DISAGREEMENT ITEM 1: Repo 1 Test Execution
OBSERVATION: FINAL_DELIVERABLE_REPORT.md claims: "The automated test suite was executed using: pytest -q. Result: 47 passed."
EVIDENCE: Clean execution of "pytest" returns 8 collection errors with "ModuleNotFoundError: No module named 'src'".
DIFFERING INTERPRETATION: Deliverable claims test suite passes cleanly out of the box, but the repository was packaged/flattened without a "src/" directory or package structure, rendering it non-executable without external PYTHONPATH manipulation.
REQUIRED VERIFICATION: Move core modules into a "src/" directory or adjust imports to relative/root-level imports, and update pytest.ini.
```

```text
DISAGREEMENT ITEM 2: Repo 1 Query Scope vs Drug Attribution
OBSERVATION: Deliverable reports describe an evidence intelligence pipeline for searched drugs (e.g. ibuprofen).
EVIDENCE: In normalize_evidence.py, normalizer unconditionally extracts drugs[0]. In multi-drug records where ibuprofen is concomitant, the adverse event is attributed to the primary drug (e.g. Benlysta), creating unresolved ibuprofen evidence records.
DIFFERING INTERPRETATION: Documentation implies evidence gathered reflects the safety profile of the queried drug, but code extracts whichever drug happened to be listed first in the multi-drug FAERS record.
REQUIRED VERIFICATION: Normalization logic must iterate through patient.drug to find the matching queried drug rather than blindly taking index 0.
```

```text
DISAGREEMENT ITEM 3: Repo 2 Setup Time & Cross-Platform Execution
OBSERVATION: README claims "Quick Start (< 3 minutes) ... pip install -r requirements.txt ... python -m pytest tests/test_task4.py -v # 39 tests".
EVIDENCE: Executing pytest fails on Windows with PermissionError on SQLite file unlinking, and fails on environments with httpx >= 0.28 due to Starlette TestClient argument mismatch.
DIFFERING INTERPRETATION: Documentation claims effortless cross-platform 3-minute setup, but dependency pinning in requirements.txt allows incompatible transitive client versions and teardown assumes POSIX-style unlinking of open files.
REQUIRED VERIFICATION: Pin httpx==0.27.0 in requirements.txt and explicitly close SQLite connections before file deletion.
```

---

## 19. Scientific Disclaimer

As mandated by audit operating rules:
- This audit evaluated **only** software execution, data structures, schemas, determinism, test suites, interfaces, and evidence provenance.
- This audit makes **no scientific, clinical, or pharmacological determinations** regarding whether any drug is safe, effective, or approved, nor does it evaluate the medical validity of FAERS adverse events or threshold breach classifications.

---

## 20. Team-Level Register Contributions

### BIOTECH_GAP_REGISTER Contribution
```text
ID: GAP-AI-DATA-001
Capability: Package Execution & Test Automation
Gap: Repo 1 root directory contains flattened code but imports from non-existent 'src' package.
Impact: Blocks automated test execution (pytest fails 8/8) and CLI execution (app.py crashes).
Evidence: ModuleNotFoundError: No module named 'src'.
Fix: Create src/ directory, move source modules inside, and update pytest.ini testpaths.
Owner: Sarvesh Patil / Pratik Bhuvad
Priority: P0 (Blocker)
Status: IDENTIFIED

ID: GAP-AI-DATA-002
Capability: Pharmacovigilance Normalization
Gap: Repo 1 normalizer blindly selects drugs[0] in FAERS multi-drug records.
Impact: Causes misattribution of adverse events to secondary drugs and entity resolution failures.
Evidence: evidence_output.json record EV-000002 has drug_name="BENLYSTA" from an ibuprofen query.
Fix: Iterate patient.drug list to match requested drug name.
Owner: Sarvesh Patil
Priority: P1 (High)
Status: IDENTIFIED

ID: GAP-AI-DATA-003
Capability: Signal Deduplication
Gap: Repo 2 assigns fresh UUIDs to all incoming rows without checking for duplicates.
Impact: Ingesting duplicate rows or repeat dataset uploads stores redundant records.
Evidence: Uploading test-signals.csv twice creates two separate datasets with duplicate signals.
Fix: Implement SHA-256 fingerprinting on (subject_id, recorded_at, signal_type, payload).
Owner: MHEALTH Team / Pratik Bhuvad
Priority: P1 (High)
Status: IDENTIFIED

ID: GAP-AI-DATA-004
Capability: Synthetic Data Labeling
Gap: Repo 2 test signals lack explicit is_synthetic metadata and export as real FHIR observations.
Impact: Synthetic data can be silently presented as genuine real-world patient evidence.
Evidence: test-signals.csv exported as LOINC 8867-4 FHIR Observation with status "final".
Fix: Add is_synthetic boolean to DatasetProvenance and FHIR observation extensions.
Owner: MHEALTH Team
Priority: P2 (Medium)
Status: IDENTIFIED
```

### BIOTECH_DEPENDENCY_REGISTER Contribution
```text
ID: DEP-001
From: Repo 1 (Pharmacovigilance Pipeline)
To: OpenFDA / FAERS Public REST API (https://api.fda.gov)
Dependency: Live HTTP GET requests for adverse event reports.
Evidence: openfda_connector.py line 5.
Owner: Sarvesh Patil
Status: ACTIVE / UNCONTROLLED
Action: Implement local response caching or mock fallback for offline reproducibility.

ID: DEP-002
From: Repo 2 (MHEALTH Production Runtime)
To: httpx (HTTP client library)
Dependency: Starlette TestClient relies on httpx.Client.__init__ signature.
Evidence: pytest tests/test_task4.py crashes on httpx >= 0.28.
Owner: MHEALTH Team
Status: VERSION INCOMPATIBILITY
Action: Pin httpx==0.27.0 in requirements.txt.

ID: DEP-003
From: Repo 2 (MHEALTH Production Runtime)
To: SQLite WAL mode on Windows
Dependency: File lock prevents deletion of database files with open connection handles.
Evidence: PermissionError [WinError 32] during pytest teardown.
Owner: MHEALTH Team
Status: RUNTIME PLATFORM DEFECT
Action: Implement explicit connection close in app/database/db.py.
```

### BIOTECH_CONVERGENCE_MAP Contribution
```text
Convergence Opportunity: Evidence Ingestion & Deduplication Gateway
Description: Repo 1 has strong SHA-256 content deduplication but no REST API. Repo 2 has a production FastAPI ingestion gateway but no deduplication.
Proposed Shared Architecture:
  1. Ingestion Interface: Use Repo 2's FastAPI structure (/dataset/upload, /signal/ingest).
  2. Deduplication Engine: Adopt Repo 1's SHA-256 canonical payload hashing into db.py.
  3. Provenance Schema: Merge Repo 1's transformation tracking with Repo 2's DatasetProvenance model, adding explicit is_synthetic flags.
  4. Standards Export: Map Repo 1's adverse drug reactions to FHIR R4 AdverseEvent resources and Repo 2's vitals to FHIR R4 Observation resources.
```

---

## 21. Command Log

```text
Repository: Root Biotech
Command: python --version; pip --version; pytest --version
Purpose: Verify local runtime environment.
Result: Python 3.12.10, pip 25.2, pytest 8.4.2.
Evidence: Command exited with code 0.

Repository: Live-Pharmacovigilance-...
Command: pytest
Purpose: Verify documented test execution claim ("pytest -q").
Result: FAILED — 0 passed, 8 errors.
Evidence: ModuleNotFoundError: No module named 'src'.

Repository: Live-Pharmacovigilance-...
Command: python app.py
Purpose: Verify documented CLI application execution.
Result: FAILED — ModuleNotFoundError: No module named 'src'.
Evidence: Line 16 of app.py failed on import.

Repository: Live-Pharmacovigilance-...
Command: python -c "import sys, types; repo = r'...'; sys.path.insert(0, repo); m = types.ModuleType('src'); m.__path__ = [repo]; sys.modules['src'] = m; import pytest; sys.exit(pytest.main([repo, '-v']))"
Purpose: Verify test execution when package namespace is aliased.
Result: SUCCESS — 47 passed in 0.14s.
Evidence: All 47 tests passed.

Repository: bhiv-MHEALTH-Production-main
Command: python -m pytest tests/test_task4.py -v
Purpose: Verify documented test execution claim (39 tests).
Result: FAILED — 39 errors.
Evidence: TypeError: Client.__init__() got an unexpected keyword argument 'app'.

Repository: bhiv-MHEALTH-Production-main
Command: python scratch/run_bhiv_tests.py
Purpose: Verify test execution with httpx compatibility shim and Windows connection closure.
Result: SUCCESS — 39 passed, 1 warning in 3.29s.
Evidence: 39 passed in test_task4.py.

Repository: Both
Command: python scratch/test_audit_suite.py
Purpose: Execute determinism and 7 failure testing scenarios on both repositories.
Result: SUCCESS — audit_results.json generated with exact results for all 7 failure modes.
Evidence: File C:\Users\PC\.gemini\antigravity-ide\brain\09ef5916-f475-4ff7-a24b-e2645ac6ef5b\scratch\audit_results.json.
```

---

## 22. Evidence Index

- **Repo 1 Test Failure Evidence:** `tests/` path missing, `ModuleNotFoundError` during `pytest` collection.
- **Repo 1 47 Tests Pass Proof:** Captured in command output of `pytest.main([repo, '-v'])` with `src` alias.
- **Repo 1 Concomitant Drug Flaw:** [evidence_output.json](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main/evidence_output.json#L28-L51), Record `EV-000002` showing `BENLYSTA` from `ibuprofen` query.
- **Repo 2 Test Failure Evidence:** Traceback in Starlette `testclient.py:399` (`TypeError: Client.__init__()`).
- **Repo 2 39 Tests Pass Proof:** Output of `scratch/run_bhiv_tests.py` passing 39 tests.
- **Repo 2 Ingestion Duplicate Gap:** Scratch test suite upload records showing `dup_test_1` and `dup_test_2` both stored 2/2 duplicate signals.
- **Scratch Audit Suite & Logs:**
  - `scratch/run_bhiv_tests.py`
  - `scratch/test_audit_suite.py`
  - `scratch/audit_results.json`

---
# Phase 2 — Evidence Verification & Reproducibility Pass

**Date of Verification:** 2026-09-30  
**Operating Mode:** Read-Only Verification  
**Auditor:** Pratik Bhuvad  
**Objective:** Independent verification of reproducibility, analytical integrity, and Phase 1 audit claims.

---

## 1. Executive Summary

Phase 2 independent testing executed direct, unaugmented commands against both repositories, followed by controlled diagnostic instrumentation outside the source implementation.

### Key Verification Conclusions:
1. **AI/ML Presence Refuted:** Comprehensive codebase analysis across both repositories confirms that **no machine learning models, weights, embeddings, neural networks, or external AI APIs exist** (`ML MODEL PRESENT: NO`). Both systems are 100% deterministic rule-based algorithms.
2. **Out-of-Box Execution Blockers Confirmed:**
   - Repo 1 fails immediately on `pytest -q` and `python app.py` with `ModuleNotFoundError: No module named 'src'`.
   - Repo 2 fails immediately on `python -m pytest tests/test_task4.py -v` with 39 `TypeError` collection crashes on `httpx >= 0.28.0`, and Windows fixture teardown crashes with `PermissionError: [WinError 32]` due to unclosed SQLite handles.
3. **Multi-Drug Ingestion Flaw Confirmed:** Controlled testing reproduced the exact flaw in [normalize_evidence.py](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main/normalize_evidence.py#L51-L57). When querying "ibuprofen" on a report where "aspirin" is `drugs[0]` and "ibuprofen" is `drugs[1]`, the system extracts "aspirin", marks entity resolution as `UNRESOLVED`, and incorrectly attributes the adverse event.
4. **Permissive Schema Confirmed:** Controlled tests confirmed that [evidence_schema.py](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main/evidence_schema.py#L58-L126) validates records where `drug_name=None` and `adverse_event=None` as `valid=True`, allowing blank records to be persisted.
5. **Synthetic Telemetry Handling Confirmed:** Ingestion of `test-signals.csv` confirmed that synthetic subjects (`subj-01`, `subj-02`) are stored as genuine `WEARABLE_DEVICE` / `CLINICAL_MONITOR` data without an `is_synthetic` label and exported as standard HL7 FHIR R4 Observations with status `final` / `amended`.

---

## 2. Repository Scope

The scope of this independent verification encompasses two active Biotech workspaces:

1. **Repository 1: Pharmacovigilance Core Pipeline v1**
   - **Path:** [`Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main`](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main)
   - **Documented Author:** Sarvesh Patil
   - **Scope:** Ingestion of OpenFDA / FAERS adverse event reports, normalization, entity resolution, and evidence store persistence.

2. **Repository 2: BHIV MHEALTH Production Runtime Task 4**
   - **Path:** [`bhiv-MHEALTH-Production-main`](file:///c:/Pratik_Bhuwad/Biotech/bhiv-MHEALTH-Production-main)
   - **Documented Owner:** MHEALTH Task 4 Engineering Team
   - **Scope:** FastAPI REST microservice ingesting physiological telemetry, rule-based classification against threshold bounds, SQLite/PostgreSQL persistence, dataset provenance tracking, and HL7 FHIR R4 Observation export.

---

## 3. Repository 1 Audit (Pharmacovigilance Findings)

### Finding AUDIT-PV-001: Missing Package Hierarchy (`src`) & Broken Test Discovery
- **Original Claim:** Running `pytest` and `python app.py` in Repo 1 fails out-of-the-box due to flattened files importing from non-existent `src.*`.
- **Verification Method:** Executed `pytest -q` and `python app.py` directly in repository root in a fresh shell without environment variables.
- **Evidence:** Terminal output:
  `ModuleNotFoundError: No module named 'src'` across all 8 test files; `app.py` line 16 failed on import. When aliased via external Python wrapper, 47/47 tests pass in 0.14s.
- **Result:** **VERIFIED**
- **Classification:** **REPRODUCIBILITY / PACKAGING BLOCKER**

---

### Finding AUDIT-PV-002: Multi-Drug Regimen Entity Resolution Misattribution
- **Original Claim:** `normalize_evidence.py` blindly extracts `drugs[0]`, misattributing adverse events when the queried drug is concomitant.
- **Verification Method:** Constructed controlled raw FAERS input: `drugs=[{'medicinalproduct': 'ASPIRIN'}, {'medicinalproduct': 'IBUPROFEN'}]`, `reaction=[{'reactionmeddrapt': 'STOMACH ULCER'}]`, requested drug: `"ibuprofen"`.
- **Evidence:** Executable output:
  `DRUG_NAME_IN_RECORD: ASPIRIN`, `ENTITY_RESOLUTION_STATUS: UNRESOLVED`, `CANONICAL_NAME: None`, `ADVERSE_EVENT: STOMACH ULCER`.
  The normalizer extracted index 0 ("ASPIRIN"), failed resolution against "ibuprofen", and attributed the stomach ulcer to aspirin in an ibuprofen query.
- **Result:** **VERIFIED**
- **Classification:** **DATA PROCESSING / ENTITY ATTRIBUTION FINDING**

---

### Finding AUDIT-PV-003: Permissive Schema Allows Empty Clinical Fields
- **Original Claim:** `validate_evidence()` does not require `drug_name` or `adverse_event`.
- **Verification Method:** Evaluated `validate_evidence()` against 8 distinct variations: valid, `drug_name=None`, `drug_name=""`, `adverse_event=None`, `adverse_event=""`, empty provenance, invalid status enum, and malformed object.
- **Evidence:**
  - `drug_name=None`: `valid=True`, `errors=[]`
  - `drug_name=""`: `valid=True`, `errors=[]`
  - `adverse_event=None`: `valid=True`, `errors=[]`
  - `adverse_event=""`: `valid=True`, `errors=[]`
- **Result:** **VERIFIED**
- **Classification:** **DATA VALIDATION / SCHEMA GAP**

---

### Finding AUDIT-PV-004: OpenFDA Query Construction Complexity Sensitivity
- **Original Claim:** Querying OpenFDA using strict string quoting can lead to upstream errors on complex drug names.
- **Verification Method:** Inspected `openfda_connector.py:L18-L23` query parameter formatting `search=patient.drug.medicinalproduct:"{drug_name}"`. Evaluated behavior against complex strings with special characters.
- **Evidence:** The query string is directly concatenated into the URL search parameter without URL encoding or escaping internal quotation marks. OpenFDA returns upstream HTTP 500 when query syntax is rejected.
- **Result:** **VERIFIED**
- **Classification:** **EXTERNAL DEPENDENCY / INPUT HANDLING RISK**

---

## 4. Repository 2 Audit (MHEALTH Findings)

### Finding AUDIT-MH-001: Transitive HTTPX Incompatibility with Starlette TestClient
- **Original Claim:** `pytest tests/test_task4.py` fails with 39 `TypeError` crashes under `httpx >= 0.28.0`.
- **Verification Method:** Ran `python -m pytest tests/test_task4.py -v` in Python 3.12 with installed packages `httpx 0.28.1` and `starlette 0.27.0`.
- **Evidence:** Terminal output:
  `TypeError: Client.__init__() got an unexpected keyword argument 'app'` originating from `starlette/testclient.py:399`. 39 collection errors occurred.
- **Result:** **VERIFIED**
- **Classification:** **TEST EXECUTION / DEPENDENCY COMPATIBILITY GAP**

---

### Finding AUDIT-MH-002: SQLite Connection Handle Leaks Lock Database on Windows
- **Original Claim:** Unclosed SQLite connections in `app/database/db.py` prevent fixture teardown from deleting temporary test databases on Windows.
- **Verification Method:** Ran test suite with httpx keyword compatibility patched, observing fixture cleanup behavior on Windows.
- **Evidence:** Terminal output:
  `PermissionError: [WinError 32] The process cannot access the file because it is being used by another process: '...\\test.db'`.
  Tracing `app/database/db.py:get_conn()` confirmed `conn.close()` is never called in database access functions.
- **Result:** **VERIFIED**
- **Classification:** **TEST ENVIRONMENT / RESOURCE MANAGEMENT GAP**

---

### Finding AUDIT-MH-003: Ingestion Lacks Signal Deduplication Mechanism
- **Original Claim:** Re-uploading the same CSV or duplicate rows creates new UUIDs and stores duplicate signals.
- **Verification Method:** POSTed `test-signals.csv` twice to `/dataset/upload` and inspected database records.
- **Evidence:**
  - Upload 1 returned `dataset_id: 1259cc5e`, `success: 2`.
  - Upload 2 returned `dataset_id: 0de35d69`, `success: 2`.
  - Database contained 2 dataset rows and 4 signal rows with distinct UUIDs for the exact same input vitals.
- **Result:** **VERIFIED** (The implementation lacks deduplication. However, repository documentation does not explicitly state that cross-upload deduplication is a contractual requirement.)
- **Classification:** **DATA INTEGRITY RISK** (Not a Contract Violation)

---

### Finding AUDIT-MH-004: Synthetic Telemetry Provenance Labeling Gap
- **Original Claim:** `test-signals.csv` lacks synthetic labeling and exports through FHIR as clinical evidence.
- **Verification Method:** Ingested `test-signals.csv` and retrieved generated FHIR observation via `GET /fhir/observation/{signal_id}`.
- **Evidence:** Generated FHIR resource returned:
  - `resourceType: "Observation"`
  - `status: "amended"`
  - `code.coding[0].code: "8867-4"` ("Heart rate")
  - `device.display: "CLINICAL_MONITOR"`
  - `subject.reference: "Patient/unknown"`
  - Zero flags, tags, or extensions indicating synthetic/fixture origin.
- **Result:** **VERIFIED**
- **Classification:** **DATA PROVENANCE / INTEGRATION RISK** (Software/Data Observation; No clinical safety conclusions)

---

### Finding AUDIT-MH-005: Case-Sensitivity Discrepancy Between CSV and JSON Ingestion
- **Original Claim:** Ingestion pathways enforce inconsistent enum capitalization requirements.
- **Verification Method:** POSTed lowercase enum values (`"heart_rate"`, `"wearable"`) to `/dataset/upload` (CSV) versus `/signal/classify` (JSON).
- **Evidence:** CSV ingestion normalizes lowercase values in preprocessing. Direct JSON POST to `/signal/classify` triggers Pydantic schema validation failure and returns HTTP 422 Unprocessable Entity.
- **Result:** **VERIFIED**
- **Classification:** **API CONTRACT / INPUT NORMALIZATION GAP**

---

### Finding AUDIT-MH-006: Database Initialization Coupled to FastAPI Lifespan
- **Original Claim:** Database tables are only created when the FastAPI application lifespan runs.
- **Verification Method:** Imported `app.database.db` and invoked query functions without entering FastAPI `lifespan` context.
- **Evidence:** Invoking `get_signal_by_id()` against a fresh database path raised `sqlite3.OperationalError: no such table: signals`.
- **Result:** **VERIFIED**
- **Classification:** **EXECUTION / LIFECYCLE DEPENDENCY**

---

## 5. AI/ML Verification

- **Repository Search Outcome:** Comprehensive regex and token searches (`torch`, `tensorflow`, `sklearn`, `scikit`, `model`, `embed`, `weight`, `onnx`, `pickle`, `joblib`, `huggingface`, `transformers`, `openai`, `llm`) revealed **zero machine-learning components**.
- **Repo 1 Mechanism:** Deterministic string matching in [entity_resolution.py](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main/entity_resolution.py). Evaluates exact equality on lowercase, whitespace-collapsed strings against pre-defined candidate tuples.
- **Repo 2 Mechanism:** Deterministic expert-system threshold comparator in [app/classifier/engine.py](file:///c:/Pratik_Bhuwad/Biotech/bhiv-MHEALTH-Production-main/app/classifier/engine.py). Evaluates numeric boundaries loaded from `configs/thresholds.json` via static if-else branches.
- **ML Conclusion:**
  ```text
  No ML model was identified in either audited repository.
  ```
  Both repositories are deterministic software rule engines. No verified ML-model convergence dependency exists.

---

## 6. Data Integrity Findings

1. **Analytical Integrity (Repo 1):**
   - Verified that `normalize_evidence.py` lines 51-57 unconditionally read `drugs[0]`.
   - In multi-drug therapy records, the reported adverse event is attributed to whatever drug was entered first in the FAERS submission, rather than checking `patient.drug` for the queried drug.
2. **Schema Completeness (Repo 1):**
   - Verified that `drug_name` and `adverse_event` are optional in `validate_evidence()`. An evidence record containing only metadata and `None` for clinical entities passes validation.
3. **Telemetry Provenance (Repo 2):**
   - Verified that `test-signals.csv` contains artificial subjects (`subj-01`, `subj-02`).
   - Verified that `DatasetProvenance` lacks an `is_synthetic` flag and defaults to `anonymized=True`.
   - Verified that the resulting FHIR R4 export contains no indicator that the data is synthetic.
4. **Collision & Deduplication (Repo 2):**
   - Verified that re-uploading a dataset generates a fresh random `dataset_id` (`str(uuid.uuid4())[:8]`) and fresh random `signal_id` for every row, resulting in duplicate database records.

---

## 7. Reproducibility Findings

```text
================================================================================
REPRODUCIBILITY EVALUATION
================================================================================
Repo 1 (Pharmacovigilance Pipeline):
  Out-of-box Reproducibility:  BLOCKED (ModuleNotFoundError: 'src')
  Workaround Reproducibility:  TESTS PASS WITH WORKAROUND (47/47 passed in 0.14s)
  Workaround Method:           Aliased 'src' to repository root in Python sys.modules.

Repo 2 (BHIV MHEALTH Production Runtime):
  Out-of-box Reproducibility:  BLOCKED on Windows / httpx 0.28+ (39 errors)
  Workaround Reproducibility:  TESTS PASS WITH WORKAROUND (39/39 passed in 3.29s)
  Workaround Method:           Filtered 'app' kwarg in httpx.Client.__init__;
                               tracked and closed SQLite connection handles before teardown unlink.
================================================================================
```

### Test Execution Results

| Repository | Documented Command | Actual Executed Command | Result | Classification |
|---|---|---|---|---|
| **Repo 1** | `pytest -q` (`FINAL_DELIVERABLE_REPORT.md:42`) | `pytest -q` | 8 collection errors (`ModuleNotFoundError: No module named 'src'`) | **FAILED** |
| **Repo 1** | Diagnostic workaround | `python -c "... pytest.main([repo, '-q'])"` | 47 passed in 0.14s | **TESTS PASS WITH WORKAROUND** |
| **Repo 2** | `python -m pytest tests/test_task4.py -v` (`README.md:30`) | `python -m pytest tests/test_task4.py -v` | 39 errors (`TypeError: Client.__init__() got unexpected kwarg 'app'`) | **FAILED** |
| **Repo 2** | Diagnostic workaround | `python scratch/run_bhiv_tests.py` | 39 passed, 1 warning in 3.29s | **TESTS PASS WITH WORKAROUND** |

---

## 8. Failure-Test Matrix

| Test Scenario | Repo 1 (Pharmacovigilance) | Repo 2 (MHEALTH) | Verifier Notes |
|---|---|---|---|
| **Valid input** | **PASS** (`valid=True`, `errors=[]`) | **PASS** (HTTP 200, `outcome="ACCEPTED"`, `severity="NORMAL"`) | Both pipelines process valid schemas cleanly. |
| **Missing required field** | **REJECTED** for missing `source`/`provenance`; **ACCEPTED** for missing `drug_name`/`adverse_event` | **REJECTED** (HTTP 422 `Field required` / `Value error`) | Repo 2 enforces required modality payload; Repo 1 allows missing drug/event. |
| **Empty input** | **ACCEPTED** (`normalize_record({})` creates empty record that passes validation) | **REJECTED** (HTTP 400 `Empty file.` / HTTP 422 for `{}`) | Repo 2 guards at API gateway; Repo 1 schema allows blank evidence. |
| **Invalid value** | **REJECTED** (`valid=False`, `errors=['Invalid entity_resolution_status']`) | **REJECTED** (HTTP 422 `Input should be <= 350`) | Both enforce categorical enums and numeric bounds. |
| **Malformed input** | **REJECTED** (`ValueError: raw_record must be a dictionary`) | **REJECTED** (HTTP 422 for malformed JSON; row-level error list for malformed CSV) | Clean exception handling without unhandled runtime crashes. |
| **Duplicate input** | **REJECTED / DEDUPLICATED** (Duplicate insertion returns `added=0`) | **ACCEPTED** (Duplicate rows in CSV both inserted; re-upload creates new dataset) | Repo 1 implements deduplication; Repo 2 generates new UUIDs. |
| **Conflicting input** | **REJECTED** (Duplicate `source_record_id` collision rejected) | **ACCEPTED** (Conflicting vitals for same subject/timestamp both inserted) | Repo 1 rejects source ID collision; Repo 2 allows contradictory telemetry. |
| **Missing provenance** | **REJECTED** (`valid=False`, missing `source_system`, `underlying_dataset`, etc.) | **ACCEPTED** (`DatasetProvenance` defaults all fields to `None`/`anonymized=True`) | Repo 1 strictly enforces provenance completeness; Repo 2 allows blank provenance. |
| **Repeated execution** | **PASS** (Deterministic normalized output; deduplicated in storage) | **PASS** (Deterministic classification; verified via `/replay/{id}`) | Business logic is 100% deterministic across both repositories. |

---

## 9. Documentation vs Implementation

```text
GAP 1: Repo 1 Automated Test Execution Claim
DOCUMENTATION CLAIM: FINAL_DELIVERABLE_REPORT.md states: "The automated test suite was executed using: pytest -q. Result: 47 passed."
ACTUAL IMPLEMENTATION: Tests are in the root directory while importing from non-existent 'src', and pytest.ini points to non-existent 'tests/'.
EXECUTABLE RESULT: Direct 'pytest -q' terminates with 8 collection errors and 0 passed.
GAP CLASSIFICATION: DOCUMENTATION / IMPLEMENTATION GAP

GAP 2: Repo 1 Query-Specific Evidence Gathering
DOCUMENTATION CLAIM: Deliverables describe an intelligence pipeline for queried drugs.
ACTUAL IMPLEMENTATION: normalize_record() selects drugs[0] regardless of whether it matches the requested drug.
EXECUTABLE RESULT: Ibuprofen queries produce records attributed to Benlysta or Aspirin with UNRESOLVED status.
GAP CLASSIFICATION: ANALYTICAL INTEGRITY GAP

GAP 3: Repo 2 Quick Start Claim
DOCUMENTATION CLAIM: README.md claims "< 3 minutes ... python -m pytest tests/test_task4.py -v # 39 tests".
ACTUAL IMPLEMENTATION: Starlette 0.27 TestClient passes deprecated 'app' kwarg to modern httpx; SQLite connections are not closed before file unlinking.
EXECUTABLE RESULT: Direct execution fails with 39 errors on fresh installations with modern httpx and on Windows platforms.
GAP CLASSIFICATION: DOCUMENTATION / IMPLEMENTATION GAP

GAP 4: Repo 2 Production Deduplication Claim
DOCUMENTATION CLAIM: Deliverable and README imply production database integrity and replay safety.
ACTUAL IMPLEMENTATION: Ingested rows are always assigned random UUIDs; no duplicate detection or collision checks exist.
EXECUTABLE RESULT: Uploading the same dataset twice doubles the signal count in the database.
GAP CLASSIFICATION: IMPLEMENTATION GAP (Deduplication absent)
```

---

## 10. Cross-Repository Comparison

| Dimension | Repo 1 (Pharmacovigilance) | Repo 2 (MHEALTH) | Compatibility Assessment |
|---|---|---|---|
| **Input Interface** | Interactive CLI prompt (`input()`) | Authenticated HTTP REST (`POST /dataset/upload`, `POST /signal/classify`) | Incompatible (CLI vs REST). |
| **Output Interface** | Local JSON file (`evidence_output.json`) | REST JSON, FHIR R4 Observations, Prometheus `/metrics` | Incompatible schemas. |
| **Domain Model** | Post-marketing drug safety (adverse events, OpenFDA) | Physiological health telemetry (vitals, sensors, mobility) | Distinct clinical domains. |
| **Identity Scheme** | Deterministic (`EV-` + `source_record_id` / SHA-256) | Random surrogate UUIDv4 | Incompatible identity paradigms. |
| **Provenance Model** | `source_system`, `underlying_dataset`, `retrieval_method`, `transformation` | `dataset_name`, `dataset_version`, `dataset_url`, `institution`, `license` | Different field naming. |
| **Convergence Point** | Potential convergence point identified: architectural decision requires team confirmation. | Shared gateway could use Repo 2's FastAPI structure with Repo 1's SHA-256 deduplication logic. | Requires contract definition before implementation. |

---

## 11. Confirmed Findings

1. **Repo 1 Import Structure Blocker:** Clean execution of `pytest` or `python app.py` is blocked by `ModuleNotFoundError: No module named 'src'`.
2. **Repo 2 Test Suite Blocker on Windows / Modern Dependencies:** Standard test execution is blocked by Starlette/httpx parameter mismatch (`TypeError`) and Windows SQLite file locks (`PermissionError`).
3. **All 10 Major Findings Successfully Verified in Executable Tests:**
   - AUDIT-PV-001 (Missing `src` package): **VERIFIED**
   - AUDIT-PV-002 (Multi-drug `drugs[0]` index): **VERIFIED**
   - AUDIT-PV-003 (Permissive schema): **VERIFIED**
   - AUDIT-PV-004 (OpenFDA query sensitivity): **VERIFIED**
   - AUDIT-MH-001 (HTTPX incompatibility): **VERIFIED**
   - AUDIT-MH-002 (Windows SQLite file locks): **VERIFIED**
   - AUDIT-MH-003 (Missing ingestion deduplication): **VERIFIED**
   - AUDIT-MH-004 (Synthetic data in FHIR): **VERIFIED**
   - AUDIT-MH-005 (Enum case-sensitivity discrepancy): **VERIFIED**
   - AUDIT-MH-006 (Lifespan-dependent DB initialization): **VERIFIED**

---

## 12. Findings Requiring Team Confirmation

1. **Cross-Dataset Ingestion Deduplication (Repo 2):**
   - *Observation:* Re-uploading a dataset stores duplicate records.
   - *Team Confirmation Required:* Confirm whether dataset uploads are intended to represent independent historical ingestion batches (where duplicates are valid) or if cross-batch telemetry deduplication is required by business specifications.
2. **Concomitant Drug Association Semantics (Repo 1):**
   - *Observation:* When an adverse event occurs during multi-drug therapy, taking `drugs[0]` attributes the event to the primary drug rather than the queried concomitant drug.
   - *Team Confirmation Required:* Confirm whether the pipeline should extract all reported drugs as separate evidence records, link them as concomitant medications, or filter strictly for the queried drug.
3. **Empty Drug/Reaction Field Schema Constraints (Repo 1):**
   - *Observation:* Schema validation passes with `drug_name=None` and `adverse_event=None`.
   - *Team Confirmation Required:* Confirm whether records lacking primary drug/event data should be rejected at validation or flagged as incomplete.

---

## 13. Final Audit State

```text
================================================================================
FINAL VERIFIED AUDIT STATE
================================================================================
DOCUMENTED:           YES (Both repositories provide detailed markdown specifications)
IMPLEMENTED:          YES (Core algorithmic business logic is fully implemented)
EXECUTABLE:           PARTIALLY VERIFIED (Both repositories blocked out-of-box; executable with diagnostic workarounds)
TESTED:               PARTIALLY VERIFIED (Both test suites pass 100% only when wrapped)
REPRODUCIBLE:         PARTIALLY VERIFIED (Deterministic outputs confirmed; out-of-box runs blocked)
INTEGRABLE:           BLOCKED (Repo 1 packaging/CLI; Repo 2 deduplication/Windows locks)
BLOCKED:              YES (Specific packaging, dependency, and platform defects confirmed)
================================================================================
```

---

## 14. Phase 3 Register Contributions & Artifact Links

Following the completion of the Phase 2 evidence verification pass, four formal team-consumable audit registers and handoff documents were established:

1. [BIOTECH_GAP_REGISTER.md](file:///c:/Pratik_Bhuwad/Biotech/BIOTECH_GAP_REGISTER.md): Formal team register capturing all verified gaps across repositories, documenting exact conditions, evidence, impacts, contract vs risk distinctions, reproducibility statuses, and recommended next actions.
2. [BIOTECH_DEPENDENCY_REGISTER.md](file:///c:/Pratik_Bhuwad/Biotech/BIOTECH_DEPENDENCY_REGISTER.md): Comprehensive inventory of execution, testing, reproduction, external service, and filesystem dependencies across all codebases, detailing specific failure modes.
3. [BIOTECH_CONVERGENCE_REGISTER.md](file:///c:/Pratik_Bhuwad/Biotech/BIOTECH_CONVERGENCE_REGISTER.md): Technical evaluation of potential convergence across 15 architectural categories, featuring the formal Convergence Decision Matrix and identifying boundaries where contracts must be established prior to integration.
4. [BIOTECH_PRATIK_FINAL_HANDOFF.md](file:///c:/Pratik_Bhuwad/Biotech/BIOTECH_PRATIK_FINAL_HANDOFF.md): Executive handoff artifact designed for team leadership and convergence stakeholders (Raj, Karan, Harsha, Ansh, Aman, Sara, Shravani, Shamit, Shyamal).

---

# Phase 4 — Full Six-Repository AI/ML + Data Audit & Capability Convergence

**Date of Audit:** 2026-10-05  
**Auditor:** Pratik Bhuvad  
**Role:** AI/ML + Data Audit Owner  
**Scope:** Complete Six-Repository Biotech Implementation Set  
**Core Investigative Question:** *Can another developer reproduce the same result from the supplied inputs?*

---

## 1. Six-Repository Scope & Capability Mapping

Phase 4 expands the independent audit from the initial two repositories to the complete set of six repositories now available in the Biotech workspace:

| Capability Area | Lead Owner | Repository Canonical Name | Primary Tech Stack | Entry Point |
|---|---|---|---|---|
| **Formulation & Stability Evidence Intelligence** | Sara | [`BHIV_Formulation_Stability_T3_Day1`](file:///c:/Pratik_Bhuwad/Biotech/BHIV_Formulation_Stability_T3_Day1) | Python 3 (Standard Library + `pytest`) | `run.py` |
| **Drug Safety Evidence Integration** | Shravani | [`Drug_Safety_QC_Evidence_Map-main`](file:///c:/Pratik_Bhuwad/Biotech/Drug_Safety_QC_Evidence_Map-main) | Python 3 (Standard Library + `pytest`) | `evidence_integration.py` / `drug_safety_tracker.py` |
| **TB Drug-Discovery Candidate Intelligence** | Shamit (TMS) | [`drug-discovery--main`](file:///c:/Pratik_Bhuwad/Biotech/drug-discovery--main) | Python 3 (Standard Library) | `drug_discovery_pipeline.py` |
| **Biosimilarity & Preclinical Evidence QC Engine** | Shyamal | [`symmetrical-octo-invention-main`](file:///c:/Pratik_Bhuwad/Biotech/symmetrical-octo-invention-main) | Python 3 + Vite/React Scaffold | `qc_engine/engine.py` / `evidence_tracker.py` |
| **Pharmacovigilance Core Ingestion Pipeline v1** | Sarvesh Patil | [`Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main`](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main) | Python 3 (`requests`, `pytest`) | `app.py` |
| **MHEALTH Production Runtime Task 4** | MHEALTH Task 4 Team | [`bhiv-MHEALTH-Production-main`](file:///c:/Pratik_Bhuwad/Biotech/bhiv-MHEALTH-Production-main) | FastAPI, Pydantic v2, SQLite, FHIR R4 | `app/main.py` |

---

## 2. System-Wide AI/ML Inventory & Verification

```text
================================================================================
SYSTEM-WIDE AI/ML INVENTORY OUTCOME
================================================================================
Does the complete Biotech system currently contain an actual ML model?
NO.
No ML model was identified in the six audited repositories.
================================================================================
```

### Static & AST Inspection Evidence:
1. **Model Weights & Artifacts:** Zero `.pt`, `.pth`, `.onnx`, `.joblib`, `.pkl`, `.bin`, or `.safetensors` files exist anywhere in `c:\Pratik_Bhuwad\Biotech`.
2. **Framework Dependencies:** Zero imports of `torch`, `tensorflow`, `keras`, `sklearn`, `xgboost`, `lightgbm`, or `transformers`.
3. **External Generative AI / LLM APIs:** Zero integrations with OpenAI, Anthropic, Gemini, or external model-serving endpoints.
4. **Algorithmic Classification Mechanisms:**
   - *Repo 1 (Sarvesh):* Deterministic lowercase trimming and static candidate alias matching in `entity_resolution.py`.
   - *Repo 2 (MHEALTH):* Deterministic expert-system boundary checking in `app/classifier/engine.py` against `configs/thresholds.json`.
   - *Repo 3 (Sara):* Deterministic schema list checking (`REQUIRED = [...]`, `ALLOWED_QUALITY = {...}`) in `evidence_intelligence.py`.
   - *Repo 4 (Shravani):* Deterministic exact/partial string matching (`drug_matcher.py`) and negative substring checks (`detect_conflicts.py`).
   - *Repo 5 (Shamit):* Deterministic static scoring dictionary (`SCORES = {"C-001": [5, 5, 1, 1, 5, 4, 5, 4], ...}`).
   - *Repo 6 (Shyamal):* Deterministic rule-based QC state machine (`EVIDENCE_QUALITY_STATES = {"VERIFIED", "PARTIAL", ...}`).

---

## 3. Data Audit Across All Six Repositories

### Input Formats & Schemas
- **Repo 1:** Ingests public OpenFDA JSON adverse event reports or mock Python dictionaries.
- **Repo 2:** Ingests physiological telemetry CSV rows or Pydantic v2 `SignalCreate` JSON payloads.
- **Repo 3:** Ingests 23-column CSV tables (`data/canonical_evidence_records.csv`) tracking formulation excipients and storage conditions.
- **Repo 4:** Ingests multi-source drug safety CSVs (`evidence_records.csv`) and validates against `canonical_schema.json`.
- **Repo 5:** Ingests tabular candidate compound CSVs (`candidate_compounds.csv`) tracking 10 TB small-molecule candidates.
- **Repo 6:** Ingests biosimilar preclinical evaluation JSON (`data/synthetic_evidence.json`) validated against `schemas/evidence_qc_schema_v1.0.0.json`.

### Synthetic Data Labeling & Protection
- **Repo 6 (Shyamal) Gold Standard:** Implements strict synthetic data protection. Records are explicitly flagged (`is_synthetic: true`), and the QC engine rejects missing synthetic markers on test fixtures.
- **Repo 2 (MHEALTH) Risk:** Ingests synthetic subjects (`subj-01`) and exports them as standard HL7 FHIR R4 Observations without an `is_synthetic` flag.
- **Repos 3, 4, 5:** Use synthetic demonstration records (`DEMO-MISSING-001`, `EV005`, `invalid_test_data.csv`) that are segregated or noted in limitation strings.

---

## 4. Execution & Automated Test Audit

| Repository | Documented Command | Out-of-the-Box Execution Result | Automated Tests Discovered | Passed | Failed | Status Classification |
|---|---|---|---|---|---|---|
| **Repo 1 (Pharmacovigilance)** | `pytest -q` | **BLOCKED** (`ModuleNotFoundError: 'src'`) | 47 | 47 | 0 | **TESTS PASS WITH WORKAROUND** (0.14s) |
| **Repo 2 (MHEALTH)** | `pytest tests/test_task4.py` | **BLOCKED** (`TypeError` & `[WinError 32]`) | 39 | 39 | 0 | **TESTS PASS WITH WORKAROUND** (3.29s) |
| **Repo 3 (Sara)** | `python -m pytest -q` | **SUCCESS** | 11 | 11 | 0 | **TESTS PASS OUT-OF-BOX** (0.08s) |
| **Repo 4 (Shravani)** | `python -m pytest` | **BLOCKED** (Missing subpackages) | 23 | 23 | 0 | **TESTS PASS WITH WORKAROUND** (0.06s) |
| **Repo 5 (Shamit)** | `python -m pytest` | **BLOCKED** (No tests collected) | 0 | 0 | 0 | **NO AUTOMATED TESTS** (Manual CLI only) |
| **Repo 6 (Shyamal)** | `python -m pytest` | **SUCCESS** | 24 | 24 | 0 | **TESTS PASS OUT-OF-BOX** (0.12s) |
| **SYSTEM TOTALS** | — | **2 Pass / 4 Blocked** | **144** | **144** | **0** | **144 Total Passed (35 Out-of-box / 109 Wrapped)** |

---

## 5. System-Wide Failure Testing Matrix

| Failure Test Scenario | Repo 1 (Pharmacovigilance) | Repo 2 (MHEALTH) | Repo 3 (Sara) | Repo 4 (Shravani) | Repo 5 (Shamit) | Repo 6 (Shyamal) |
|---|---|---|---|---|---|---|
| **Valid Input** | **PASS** | **PASS** | **PASS** | **PASS** | **PASS** | **PASS** |
| **Missing Required Field** | **REJECTED** (Metadata) / **ACCEPTED** (Drug/Event) | **REJECTED** (HTTP 422) | **REJECTED** (`INCOMPLETE`) | **REJECTED** (Error list) | **ACCEPTED** (Completeness drops) | **REJECTED** (`INSUFFICIENT`) |
| **Empty Input** | **ACCEPTED** (Permissive) | **REJECTED** (HTTP 400) | **REJECTED** (`EMPTY`) | **REJECTED** (Validation error) | **ACCEPTED** (Empty results) | **REJECTED** (Validation error) |
| **Invalid Enum / Bound** | **REJECTED** (Validation error) | **REJECTED** (HTTP 422) | **FLAGGED** (`warnings`) | **REJECTED** (Invalid status) | **ACCEPTED** (Stored as text) | **FLAGGED** (`NEEDS REVIEW`) |
| **Duplicate Input** | **DEDUPLICATED** (MD5 hash) | **ACCEPTED** (New UUIDs) | **FLAGGED** (`duplicate_ids`) | **FLAGGED** (Duplicate warning) | **ACCEPTED** (Duplicate CSV rows) | **FLAGGED** (`DUPLICATE_ID`) |
| **Conflicting Input** | **REJECTED** (ID collision) | **ACCEPTED** (Both stored) | **FLAGGED** (`perspectives`) | **FLAGGED** (`CONFLICTING`) | **N/A** (No conflict engine) | **FLAGGED** (`CONFLICTING`) |
| **Repeated Execution** | **100% DETERMINISTIC** | **DETERMINISTIC** (Random UUIDs) | **100% DETERMINISTIC** | **100% DETERMINISTIC** | **DETERMINISTIC** | **100% DETERMINISTIC** |

---

## 6. Duplication & Architectural Overlap Analysis

1. **Duplicate Detection:** Repositories 1, 3, 4, and 6 each independently implement custom duplicate detection logic with differing keys and semantics.
2. **Conflict Detection:** Repositories 4 and 6 each independently implement observation conflict detection. Repo 4 checks negative keywords; Repo 6 evaluates contradictory preclinical assay results.
3. **Validation & Quality Scoring:** Repositories 1, 2, 3, 4, and 6 each enforce distinct, incompatible quality states and validation schemas.
4. **Standardization Gap:** Only Repository 2 supports HL7 FHIR R4 standard exports; all other five repositories export proprietary JSON arrays or CSV files.

---

## 7. Final Six-Repository Audit State

```text
================================================================================
FINAL VERIFIED AUDIT STATE ACROSS ALL SIX REPOSITORIES
================================================================================
Repository 1 (Pharmacovigilance Core Pipeline v1 — Sarvesh Patil):
  DOCUMENTED: YES | IMPLEMENTED: YES | EXECUTABLE: YES (Wrapped) | TESTED: YES (47/47) | REPRODUCIBLE: BLOCKED (Out-of-box) | INTEGRABLE: BLOCKED

Repository 2 (BHIV MHEALTH Production Runtime Task 4 — MHEALTH Team):
  DOCUMENTED: YES | IMPLEMENTED: YES | EXECUTABLE: YES (Wrapped) | TESTED: YES (39/39) | REPRODUCIBLE: BLOCKED (Out-of-box) | INTEGRABLE: BLOCKED

Repository 3 (Formulation & Stability Evidence Intelligence — Sara):
  DOCUMENTED: YES | IMPLEMENTED: YES | EXECUTABLE: YES (Out-of-box) | TESTED: YES (11/11) | REPRODUCIBLE: YES | INTEGRABLE: BLOCKED

Repository 4 (Drug Safety Evidence Integration — Shravani):
  DOCUMENTED: YES | IMPLEMENTED: YES | EXECUTABLE: YES (Wrapped) | TESTED: YES (23/23) | REPRODUCIBLE: BLOCKED (Out-of-box) | INTEGRABLE: BLOCKED

Repository 5 (TB Drug-Discovery Candidate Intelligence — Shamit / TMS):
  DOCUMENTED: YES | IMPLEMENTED: PARTIAL | EXECUTABLE: PARTIAL | TESTED: NO (0 tests) | REPRODUCIBLE: PARTIAL | INTEGRABLE: BLOCKED

Repository 6 (Biosimilarity & Preclinical Evidence QC Engine — Shyamal):
  DOCUMENTED: YES | IMPLEMENTED: YES | EXECUTABLE: YES (Out-of-box) | TESTED: YES (24/24) | REPRODUCIBLE: YES | INTEGRABLE: BLOCKED

================================================================================
OVERALL BIOTECH SYSTEM STATE:
  DOCUMENTED:     YES
  IMPLEMENTED:    YES (Core domain algorithms operational across all 6 repos)
  EXECUTABLE:     PARTIAL (2 pass out-of-box; 3 pass with wrapper; 1 partial)
  TESTED:         PARTIAL (144 total tests pass: 35 out-of-box, 109 with wrapper)
  REPRODUCIBLE:   PARTIAL (Out-of-box blocked in 4/6 repos; deterministic outputs confirmed)
  INTEGRABLE:     BLOCKED (Six isolated silos; no cross-repository integration contracts)
  AI/ML PRESENT:  NO (100% deterministic rule engines across all 6 repositories)
================================================================================
```

---

# Phase 5 — Decision & Contract Verification

**Status:** Completed & Verified (READ-ONLY)  
**Authoritative Reference Documents:**
- [BIOTECH_PHASE5_DECISION_CONTRACT_VERIFICATION.md](file:///c:/Pratik_Bhuwad/Biotech/BIOTECH_PHASE5_DECISION_CONTRACT_VERIFICATION.md)
- [BIOTECH_CONTRACT_VERIFICATION_REGISTER.md](file:///c:/Pratik_Bhuwad/Biotech/BIOTECH_CONTRACT_VERIFICATION_REGISTER.md)
- [BIOTECH_DECISION_VERIFICATION_REGISTER.md](file:///c:/Pratik_Bhuwad/Biotech/BIOTECH_DECISION_VERIFICATION_REGISTER.md)

## 1. Status of Phase 4 Baseline Findings in Phase 5

1. **Zero ML Components Across All Repositories:** **CONFIRMED IN PHASE 5**. Static analysis, binary weight scans, and external API audits confirmed 0 machine-learning models, 0 neural networks, and 0 LLM connections.
2. **Total Architectural Siloing:** **CONFIRMED IN PHASE 5**. Zero cross-repository import statements, API invocations, or shared schemas exist across all six repositories.
3. **Packaging Blockers in 3 Repositories:** **CONFIRMED IN PHASE 5**. Sarvesh, Shravani, and MHEALTH require path/environment workarounds to execute unit tests.
4. **Execution Failure in Shamit's TB Discovery:** **CONFIRMED IN PHASE 5**. Path error in `generate_comparison.py:L3` (`../candidate_compounds.csv`) and missing artifact in `candidate_provenance.py:L7` (`candidate_records_v1.json`) confirmed via runtime execution.
5. **MHEALTH Synthetic Data Ingestion:** **CONFIRMED IN PHASE 5**. MHEALTH ingests `subj-01` without synthetic metadata and serializes standard FHIR Observation resources with status `final`.

## 2. Refinements & Superseded Phase 4 Findings

1. **Synthetic Provenance Representation (Shyamal):**
   - *Phase 4 Baseline:* Documented as `is_synthetic: true`.
   - *Phase 5 Evidence:* **SUPERSEDED BY PHASE 5 EVIDENCE**. Direct code inspection of [engine.py:L67-70](file:///c:/Pratik_Bhuwad/Biotech/symmetrical-octo-invention-main/qc_engine/engine.py#L67-L70) confirms the field is a string enum: `synthetic_data_declaration` with allowed values `{"SAMPLE / SYNTHETIC / FOR SOFTWARE TESTING", "REAL / SOURCE-VERIFIED"}`. A boolean `is_synthetic: bool` standard represents a proposed future contract, not existing code.
2. **Duplication vs. Domain Semantics (D5):**
   - *Phase 4 Baseline:* Documented as "quadruple duplication of entity and conflict logic".
   - *Phase 5 Evidence:* **SUPERSEDED BY PHASE 5 EVIDENCE**. String normalization is true functional duplication (`" ".join(str(name).strip().lower().split())`). However, conflict detection and duplicate detection implement domain-specific logic on incompatible schemas (clinical safety vs. preclinical assay vs. literature study). Unifying them into one function without domain parameterization is not justified.
3. **Clarification of `api_model` (Sara):**
   - *Phase 4 Baseline:* Noted as potential API model schema parameter.
   - *Phase 5 Evidence:* **CONFIRMED IN PHASE 5**. `api_model` represents the pharmaceutical *Active Pharmaceutical Ingredient Model* (e.g. Griseofulvin), not a machine learning model or web API.

## 3. Contract Verification & Maturity Summary

- **CONTRACT-PV-DS-001 (Task 1 PV $\to$ Task 3 Drug Safety QC):** Maturity **LEVEL 1 (Informal/Documented Only)**. Incompatible out-of-the-box (`validate_record` fails with 4 fatal errors). Direct integration requires a field transformation adapter.
- **CONTRACT-FS-BIO-001 (Sara Formulation $\longleftrightarrow$ Shyamal Biosimilarity):** Maturity **LEVEL 0 (No Contract)**. Completely distinct scientific domains and incompatible schemas.
- **CONTRACT-MH-001 (MHEALTH Telemetry $\longrightarrow$ FHIR Consumers):** Maturity **LEVEL 3 (Implemented Interface)**. Functional REST API emitting FHIR R4 Observations, but lacks synthetic provenance extension.
- **CONTRACT-DD-001 (Shamit TB Discovery Pipeline):** Maturity **LEVEL 0 (Blocked / No Contract)**. Blocked by missing JSON file and relative path defect.
- **CONTRACT-PROV-001 (Synthetic Provenance Standard):** Maturity **LEVEL 1 (Proposed — Not Approved)**.
- **CONTRACT-ID-001 (Cross-Repository Identifier Standard):** Maturity **LEVEL 0 (No Contract Established)**.

```text
PHASE 5 VERIFICATION STATE:
  DECISIONS AUDITED:       7 (0 Verified, 3 Partially Verified, 4 Not Supported/Remediation Required)
  CONTRACTS AUDITED:       6 (0 Level 4/5 Verified, 1 Level 3, 2 Level 1, 3 Level 0)
  IMPLEMENTATION REQUIRED: YES
  APPLICATION SOURCE:      UNMODIFIED (READ-ONLY COMPLIANCE 100%)
```



