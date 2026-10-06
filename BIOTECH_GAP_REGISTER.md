# BIOTECH — TEAM GAP REGISTER
**Auditor:** Pratik Bhuvad  
**Role:** AI/ML + Data Audit Owner — Independent verification of reproducibility and analytical/data integrity  
**Date:** 2026-10-05  
**Status:** Phase 4 Six-Repository System-Wide Audit (Authoritative & Verified)

---

## 1. Executive Summary & Overview

This Gap Register documents all verified defects, analytical risks, packaging blockers, and lifecycle gaps discovered during the independent AI/ML and Data Audit across the complete six-repository Biotech implementation set:
1. `Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main` (Pharmacovigilance Pipeline v1 — Sarvesh Patil)
2. `bhiv-MHEALTH-Production-main` (BHIV MHEALTH Production Runtime Task 4 — MHEALTH Task 4 Team)
3. `BHIV_Formulation_Stability_T3_Day1` (Formulation & Stability Evidence Intelligence — Sara)
4. `Drug_Safety_QC_Evidence_Map-main` (Drug Safety Evidence Integration — Shravani)
5. `drug-discovery--main` (TB Drug-Discovery Candidate Intelligence — Shamit / TMS)
6. `symmetrical-octo-invention-main` (Biosimilarity & Preclinical Evidence QC — Shyamal)

Every entry in this register traces directly to executable proof and observed repository behaviors established in [BIOTECH_PRATIK_INDEPENDENT_AUDIT.md](file:///c:/Pratik_Bhuwad/Biotech/BIOTECH_PRATIK_INDEPENDENT_AUDIT.md). Gaps are categorized, evaluated for contractual compliance versus operational risk, and linked to actionable recommendations for team resolution.

---

## 2. Team Gap Summary Table

| Gap ID | Source Finding | Repository | Capability | Category | Observed Condition | Contract Status | Reproducibility | Priority | Owner | Next Action |
|---|---|---|---|---|---|---|---|---|---|---|
| **GAP-PV-001** | AUDIT-PV-001 | Repo 1 (Pharmacovigilance) | Task 1 Core Pipeline | Packaging / Runtime | Source and test files flattened in root; modules import from non-existent `src.*`. | DOCUMENTED CONTRACT VIOLATED | REPRODUCIBLE (Fails out-of-box) | TEAM TO PRIORITIZE | Sarvesh Patil (Documented Developer) | Move modules into `src/` or adopt root-relative imports; update `pytest.ini`. |
| **GAP-PV-002** | AUDIT-PV-002 | Repo 1 (Pharmacovigilance) | Task 1 Core Pipeline | Analytical / Normalization | `normalize_record()` unconditionally selects `drugs[0]` in multi-drug FAERS reports. | ANALYTICAL LOGIC FLAW | REPRODUCIBLE (Verified with controlled input) | TEAM TO PRIORITIZE | Sarvesh Patil (Documented Developer) | Match `patient.drug` against `requested_drug_name` before index fallback. |
| **GAP-PV-003** | AUDIT-PV-003 | Repo 1 (Pharmacovigilance) | Task 1 Core Pipeline | Validation / Schema | `validate_evidence()` allows `drug_name=None` and `adverse_event=None` to pass validation. | VALIDATION GAP | REPRODUCIBLE (Verified across 8 variations) | TEAM TO PRIORITIZE | Sarvesh Patil (Documented Developer) | Add mandatory non-empty validation checks for `drug_name` and `adverse_event`. |
| **GAP-PV-004** | AUDIT-PV-004 | Repo 1 (Pharmacovigilance) | Task 1 Core Pipeline | External Dependency | Quoting syntax `patient.drug.medicinalproduct:"<name>"` sensitive to complex strings. | INPUT HANDLING RISK | REPRODUCIBLE (Quoting behavior verified) | TEAM TO PRIORITIZE | Sarvesh Patil (Documented Developer) | Implement URL encoding and fallback unquoted search on OpenFDA 500 errors. |
| **GAP-MH-001** | AUDIT-MH-001 | Repo 2 (MHEALTH) | Task 4 Runtime / API | Test Execution | Starlette 0.27 `TestClient` passes obsolete `app` keyword argument to `httpx >= 0.28.0`. | DEPENDENCY COMPATIBILITY GAP | REPRODUCIBLE (39 errors on fresh httpx) | TEAM TO PRIORITIZE | MHEALTH Task 4 Team | Pin `httpx==0.27.0` in `requirements.txt` or upgrade `starlette`/`fastapi`. |
| **GAP-MH-002** | AUDIT-MH-002 | Repo 2 (MHEALTH) | Task 4 Runtime / API | Platform / Resource | `app/database/db.py` leaks open SQLite connection handles, locking database on Windows. | RESOURCE MANAGEMENT GAP | REPRODUCIBLE (PermissionError on Windows) | TEAM TO PRIORITIZE | MHEALTH Task 4 Team | Explicitly close SQLite connections after query execution in `db.py`. |
| **GAP-MH-003** | AUDIT-MH-003 | Repo 2 (MHEALTH) | Task 4 Runtime / API | Data Integrity | Ingesting duplicate rows or repeat CSV uploads creates new UUIDs without deduplication. | DATA INTEGRITY RISK (Not a contract breach) | REPRODUCIBLE (Verified via duplicate uploads) | TEAM TO PRIORITIZE | MHEALTH Task 4 Team | Implement natural key hashing on `(subject_id, recorded_at, signal_type, payload)`. |
| **GAP-MH-004** | AUDIT-MH-004 | Repo 2 (MHEALTH) | Task 4 Runtime / API | Provenance / Standards | Synthetic test subjects (`subj-01`) exported through FHIR R4 Observations without synthetic flag. | PROVENANCE / INTEGRATION RISK | REPRODUCIBLE (Verified via FHIR export) | TEAM TO PRIORITIZE | MHEALTH Task 4 Team | Add `is_synthetic: bool` to `DatasetProvenance` and emit FHIR extension. |
| **GAP-MH-005** | AUDIT-MH-005 | Repo 2 (MHEALTH) | Task 4 Runtime / API | API Contract | CSV ingestion accepts lowercase enums (`wearable`), but JSON `/signal/classify` requires uppercase. | INPUT NORMALIZATION GAP | REPRODUCIBLE (HTTP 422 on lowercase JSON) | TEAM TO PRIORITIZE | MHEALTH Task 4 Team | Add pre-validation string normalizer (case-folding) in Pydantic schema. |
| **GAP-MH-006** | AUDIT-MH-006 | Repo 2 (MHEALTH) | Task 4 Runtime / API | Execution / Lifecycle | Calling DB functions without entering FastAPI `lifespan` fails with `no such table: signals`. | LIFECYCLE DEPENDENCY | REPRODUCIBLE (Verified outside context) | TEAM TO PRIORITIZE | MHEALTH Task 4 Team | Ensure `init_db()` is called defensively during module load or standalone CLI. |
| **GAP-FS-001** | AUDIT-FS-001 | `BHIV_Formulation_Stability_T3_Day1` | Sara (Formulation & Stability) | Data / Character Encoding | Non-breaking en-dash in `"Moderate–High"` string triggers encoding replacement on standard Windows console streams. | DATA QUALITY RISK | REPRODUCIBLE (Observed in `run.py run` output) | TEAM TO PRIORITIZE | Sara | Standardize hyphenation to ASCII hyphen-minus (`"Moderate-High"`). |
| **GAP-FS-002** | AUDIT-FS-002 | `BHIV_Formulation_Stability_T3_Day1` | Sara (Formulation & Stability) | CLI / Usability | `run.py search` with unknown arguments lacks graceful error guidance. | USABILITY / CLI HANDLING | REPRODUCIBLE | TEAM TO PRIORITIZE | Sara | Add input validation and help tips in `argparse` configuration. |
| **GAP-DS-001** | AUDIT-DS-001 | `Drug_Safety_QC_Evidence_Map-main` | Shravani (Drug Safety Integration) | Packaging / Runtime | Source files flattened in root; modules import from non-existent packages (`conflicts`, `matching`, `duplicates`, etc.). | DOCUMENTED CONTRACT VIOLATED | REPRODUCIBLE (6 collection errors on `pytest`) | TEAM TO PRIORITIZE | Shravani | Reconstruct subpackages or change imports to root-relative syntax. |
| **GAP-DS-002** | AUDIT-DS-002 | `Drug_Safety_QC_Evidence_Map-main` | Shravani (Drug Safety Integration) | Documentation / Execution | Documented CLI command references `sample_data/evidence_records.csv`, but directory does not exist. | DOCUMENTATION DEFECT | REPRODUCIBLE (`FileNotFoundError` out-of-box) | TEAM TO PRIORITIZE | Shravani | Update documentation command to `evidence_records.csv` or create `sample_data/`. |
| **GAP-DS-003** | AUDIT-DS-003 | `Drug_Safety_QC_Evidence_Map-main` | Shravani (Drug Safety Integration) | Analytical / Logic | `detect_conflicts.py` relies on simplistic negative string matching; does not detect semantic numeric contradictions. | ANALYTICAL LIMITATION | REPRODUCIBLE | TEAM TO PRIORITIZE | Shravani | Implement structured parameter-level conflict detection rules. |
| **GAP-DD-001** | AUDIT-DD-001 | `drug-discovery--main` | Shamit (TB Drug Discovery) | Execution / Path Dependency | `generate_comparison.py` hardcodes `CSV_FILE = "../candidate_compounds.csv"`; fails with `FileNotFoundError` from root. | EXECUTION DEFECT | REPRODUCIBLE (Crashes on direct run) | TEAM TO PRIORITIZE | Shamit (TMS) | Resolve path using `Path(__file__).parent / 'candidate_compounds.csv'`. |
| **GAP-DD-002** | AUDIT-DD-002 | `drug-discovery--main` | Shamit (TB Drug Discovery) | Data Persistence / Missing Artifact | `candidate_provenance.py` requires `candidate_records_v1.json` and `calculate_scores_v2.py` requires `candidate_records_with_provenance.json` (both missing). | DATA PIPELINE DEFECT | REPRODUCIBLE (Fails with missing file error) | TEAM TO PRIORITIZE | Shamit (TMS) | Commit prerequisite JSON datasets or implement ingestion pipeline generator. |
| **GAP-DD-003** | AUDIT-DD-003 | `drug-discovery--main` | Shamit (TB Drug Discovery) | Test Automation / Verifiability | Documented "Phase 7 — Testing: 16/16 PASS" consists of manual console output; no automated `pytest` test suite exists. | TEST INTEGRITY GAP | REPRODUCIBLE (`pytest` collects 0 tests) | TEAM TO PRIORITIZE | Shamit (TMS) | Implement formal automated `pytest` test suite covering search and filtering logic. |
| **GAP-BIO-001** | AUDIT-BIO-001 | `symmetrical-octo-invention-main` | Shyamal (Biosimilarity QC) | Architecture / Build System | Standalone Python QC engine coexists with unreferenced, unbuilt Vite/TypeScript web scaffold. | ARCHITECTURAL DIVERGENCE | REPRODUCIBLE (Two disconnected frameworks) | TEAM TO PRIORITIZE | Shyamal | Connect frontend to Python backend or document separate components. |

---

## 3. Detailed Gap Register Entries

### GAP-PV-001: Missing Package Hierarchy (`src`) & Broken Test Discovery
- **Source Finding ID:** AUDIT-PV-001
- **Repository:** `Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main`
- **Capability:** Task 1 Core Pipeline
- **Category:** Packaging / Runtime
- **Observed Condition:** The repository contains all Python source and test files flattened directly into the root folder. However, all internal imports use package notation: `from src.openfda_connector import search_drug`, `from src.normalize_evidence import normalize_record`, `from src.evidence_schema import validate_evidence`. Furthermore, `pytest.ini` defines `testpaths = tests`, but no `tests/` directory exists.
- **Evidence:** Executing `pytest -q` directly terminates with 8 collection errors: `ModuleNotFoundError: No module named 'src'`. Executing `python app.py` crashes at line 16 with the same error. When an external diagnostic wrapper aliases `src` to the repository root, all 47 tests pass in 0.14s.
- **Impact:** Any developer cloning the repository cannot run the command-line application or execute automated tests out-of-the-box without modifying files or setting external `PYTHONPATH` environment variables.
- **Contract Status:** **DOCUMENTED CONTRACT VIOLATED.** `FINAL_DELIVERABLE_REPORT.md` (lines 40–46) explicitly states: *"The automated test suite was executed using: pytest -q. Result: 47 passed."* This claim is false for the supplied repository layout.
- **Reproducibility Status:** REPRODUCIBLE (Fails cleanly and deterministically on fresh checkout).
- **Priority:** TEAM TO PRIORITIZE
- **Owner:** Sarvesh Patil (Documented Developer)
- **Recommended Next Action:** Restructure the repository into a standard Python layout by moving core modules (`openfda_connector.py`, `normalize_evidence.py`, `entity_resolution.py`, `evidence_schema.py`, `evidence_store.py`, `monitoring.py`, `error_handler.py`) into a `src/` directory, moving test files into a `tests/` directory, and validating that `pytest -q` executes cleanly.
- **Verification Required:** Run `pytest -q` and `python app.py` in an unconfigured shell to verify successful execution without environment shims.

---

### GAP-PV-002: Multi-Drug Regimen Entity Resolution Misattribution
- **Source Finding ID:** AUDIT-PV-002
- **Repository:** `Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main`
- **Capability:** Task 1 Core Pipeline
- **Category:** Analytical / Normalization
- **Observed Condition:** In [normalize_evidence.py](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main/normalize_evidence.py#L51-L57), the normalizer extracts the first drug listed in the FAERS patient drug array:
  ```python
  if drugs:
      first_drug = drugs[0]
      if isinstance(first_drug, dict):
          drug_name = first_drug.get("medicinalproduct")
  ```
  In multi-drug therapy regimens, the first drug in the array is often not the drug requested in the query.
- **Evidence:** A controlled test with `drugs=[{'medicinalproduct': 'ASPIRIN'}, {'medicinalproduct': 'IBUPROFEN'}]` and query `"ibuprofen"` output: `DRUG_NAME_IN_RECORD: ASPIRIN`, `ENTITY_RESOLUTION_STATUS: UNRESOLVED`, and attributed the adverse event to `ASPIRIN`. In the supplied [evidence_output.json](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main/evidence_output.json#L28-L51), record `EV-000002` resulted from an "ibuprofen" search but contains `drug_name: "BENLYSTA"` and `entity_resolution_status: "UNRESOLVED"`.
- **Impact:** Reported adverse events are attributed to the wrong drug, corrupting pharmacovigilance safety data and causing false-negative entity resolution for queried drugs.
- **Contract Status:** **ANALYTICAL LOGIC FLAW.** Deliverable documentation implies evidence gathered reflects the safety profile of the queried drug, but the code captures concomitant drugs appearing at array index 0.
- **Reproducibility Status:** REPRODUCIBLE (100% reproducible with multi-drug inputs).
- **Priority:** TEAM TO PRIORITIZE
- **Owner:** Sarvesh Patil (Documented Developer)
- **Recommended Next Action:** Iterate over the `patient.drug` array to find a match against `requested_drug_name` before falling back to `drugs[0]`, or emit separate evidence records for all concomitant drugs.
- **Verification Required:** Ingest multi-drug test fixtures and verify that the queried drug is correctly resolved and attributed.

---

### GAP-PV-003: Permissive Clinical Evidence Schema Allows Blank Clinical Fields
- **Source Finding ID:** AUDIT-PV-003
- **Repository:** `Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main`
- **Capability:** Task 1 Core Pipeline
- **Category:** Validation / Schema
- **Observed Condition:** In [evidence_schema.py:validate_evidence()](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main/evidence_schema.py#L58-L126), validation verifies the presence of metadata fields (`source_record_id`, `provenance`, `source`, `timestamp`) and enum values (`entity_resolution_status`), but fails to enforce non-empty constraints on core clinical payload fields (`drug_name`, `adverse_event`).
- **Evidence:** Executing `validate_evidence()` against 8 variations confirmed:
  - `drug_name=None`: returns `is_valid: True, errors: []`
  - `drug_name=""`: returns `is_valid: True, errors: []`
  - `adverse_event=None`: returns `is_valid: True, errors: []`
  - `adverse_event=""`: returns `is_valid: True, errors: []`
  Executing `normalize_record({})` generates an empty record that passes validation.
- **Impact:** Incomplete or corrupted records with no drug entity or adverse event can enter the evidence store and be exported as valid evidence.
- **Contract Status:** **VALIDATION GAP.** Schema fails to enforce completeness for the two fundamental clinical entities of pharmacovigilance.
- **Reproducibility Status:** REPRODUCIBLE (100% reproducible across all 8 tested variations).
- **Priority:** TEAM TO PRIORITIZE
- **Owner:** Sarvesh Patil (Documented Developer)
- **Recommended Next Action:** Add explicit non-empty string checks for `drug_name` and `adverse_event` in `validate_evidence()`:
  ```python
  if not record.get("drug_name"):
      errors.append("drug_name must be a non-empty string")
  if not record.get("adverse_event"):
      errors.append("adverse_event must be a non-empty string")
  ```
- **Verification Required:** Run unit tests asserting that `validate_evidence({"drug_name": None})` returns `is_valid: False`.

---

### GAP-PV-004: OpenFDA REST Query Construction Syntax Sensitivity
- **Source Finding ID:** AUDIT-PV-004
- **Repository:** `Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main`
- **Capability:** Task 1 Core Pipeline
- **Category:** External Dependency / Input Handling
- **Observed Condition:** In [openfda_connector.py:search_drug()](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main/openfda_connector.py#L18-L23), query parameters are constructed via direct string interpolation:
  ```python
  params = {
      "search": f'patient.drug.medicinalproduct:"{drug_name}"',
      "limit": limit
  }
  ```
- **Evidence:** Drug queries containing internal quotes, punctuation, or complex salt formulations (e.g., multi-word chemical names) are not escaped or URL-encoded. Upstream OpenFDA servers reject improperly formatted queries with HTTP 500 or 400 errors.
- **Impact:** Valid complex drug searches fail completely when querying live OpenFDA endpoints.
- **Contract Status:** **INPUT HANDLING RISK.** The pipeline assumes clean alphanumeric drug names without handling special query syntax characters.
- **Reproducibility Status:** REPRODUCIBLE (Query construction behavior verified).
- **Priority:** TEAM TO PRIORITIZE
- **Owner:** Sarvesh Patil (Documented Developer)
- **Recommended Next Action:** Use `urllib.parse.quote` or sanitize input strings, and implement fallback unquoted phrase search if quoted query returns an error.
- **Verification Required:** Test queries with special characters (`"aspirin/caffeine"`, `"acetylsalicylic acid"`) against the OpenFDA connector.

---

### GAP-MH-001: Starlette TestClient / Modern HTTPX Version Incompatibility
- **Source Finding ID:** AUDIT-MH-001
- **Repository:** `bhiv-MHEALTH-Production-main`
- **Capability:** Task 4 Runtime / API
- **Category:** Test Execution / Dependency Compatibility
- **Observed Condition:** `requirements.txt` leaves `httpx` unpinned. Starlette 0.27's `TestClient` instantiates `httpx.Client(app=...)`, passing the `app` keyword argument. In `httpx >= 0.28.0`, the `app` parameter was completely removed in favor of `transport=ASGITransport(app=...)`.
- **Evidence:** Direct execution of `python -m pytest tests/test_task4.py -v` under Python 3.12 with `httpx 0.28.1` crashed all 39 tests during setup with:
  `TypeError: Client.__init__() got an unexpected keyword argument 'app'`.
  When a custom diagnostic test runner intercepted and removed the `app` parameter, all 39 tests passed in 3.29s.
- **Impact:** Automated CI/CD test execution fails out-of-the-box on any environment where modern `httpx` is installed.
- **Contract Status:** **DEPENDENCY COMPATIBILITY GAP.** Documented command `python -m pytest tests/test_task4.py -v` fails out-of-the-box.
- **Reproducibility Status:** REPRODUCIBLE (100% reproducible with `httpx >= 0.28.0`).
- **Priority:** TEAM TO PRIORITIZE
- **Owner:** MHEALTH Task 4 Team
- **Recommended Next Action:** Pin `httpx==0.27.0` in `requirements.txt`, or upgrade Starlette/FastAPI to a version compatible with modern HTTPX.
- **Verification Required:** Run `pytest tests/test_task4.py` in a fresh virtualenv with pinned dependencies.

---

### GAP-MH-002: SQLite Connection Handle Leaks Lock Database Files on Windows
- **Source Finding ID:** AUDIT-MH-002
- **Repository:** `bhiv-MHEALTH-Production-main`
- **Capability:** Task 4 Runtime / API
- **Category:** Platform / Resource Management
- **Observed Condition:** In `app/database/db.py:get_conn()`, SQLite database connections are opened via `sqlite3.connect()` but never explicitly closed (`conn.close()` is absent across database helper functions).
- **Evidence:** Running the test suite on Windows resulted in fixture teardown failures:
  `PermissionError: [WinError 32] The process cannot access the file because it is being used by another process: '...\\test.db'`.
  Windows enforces strict mandatory file locks on unclosed SQLite file handles, preventing file deletion during fixture cleanup.
- **Impact:** Tests crash on Windows workstations and CI runners during temporary database unlink operations.
- **Contract Status:** **RESOURCE MANAGEMENT GAP.** Platform-specific resource cleanup failure on Windows environments.
- **Reproducibility Status:** REPRODUCIBLE (100% reproducible on Windows when temporary databases are deleted).
- **Priority:** TEAM TO PRIORITIZE
- **Owner:** MHEALTH Task 4 Team
- **Recommended Next Action:** Refactor database connections to use context managers (`with get_conn() as conn:`) or explicitly call `conn.close()` in `finally:` blocks.
- **Verification Required:** Run test suite on Windows and verify that temporary database files delete without `PermissionError`.

---

### GAP-MH-003: Telemetry Ingestion Lacks Deduplication Mechanism
- **Source Finding ID:** AUDIT-MH-003
- **Repository:** `bhiv-MHEALTH-Production-main`
- **Capability:** Task 4 Runtime / API
- **Category:** Data Integrity / Deduplication
- **Observed Condition:** In `app/services/dataset_service.py` and `app/services/signal_service.py`, each ingested record is unconditionally assigned a newly generated random UUID4 (`id = str(uuid.uuid4())`).
- **Evidence:** Re-uploading `test-signals.csv` twice generated two distinct `dataset_id` entries (`1259cc5e`, `0de35d69`) and stored 4 signal rows in the database for the exact same input telemetry. Submitting identical JSON payloads to `/signal/classify` creates duplicate rows.
- **Impact:** Repeat ingestion causes silent telemetry inflation, duplicate patient sensor readings, and invalid downstream statistical analyses.
- **Contract Status:** **DATA INTEGRITY RISK.** (Refined in Phase 2: Not a documented contract violation, as repository documentation does not explicitly guarantee cross-upload deduplication).
- **Reproducibility Status:** REPRODUCIBLE (100% reproducible via duplicate uploads).
- **Priority:** TEAM TO PRIORITIZE
- **Owner:** MHEALTH Task 4 Team
- **Recommended Next Action:** Define a deterministic natural key (e.g., SHA-256 hash of `subject_id + recorded_at + signal_type + payload`) and enforce database uniqueness constraints.
- **Verification Required:** Attempt to upload duplicate telemetry records and verify HTTP 409 Conflict or deduplicated storage.

---

### GAP-MH-004: Synthetic Telemetry Exported Without Provenance Flag
- **Source Finding ID:** AUDIT-MH-004
- **Repository:** `bhiv-MHEALTH-Production-main`
- **Capability:** Task 4 Runtime / API
- **Category:** Provenance / Standards
- **Observed Condition:** Synthetic test datasets (e.g., `subj-01`, `subj-02` in `test-signals.csv`) are ingested, stored, and exported as HL7 FHIR R4 Observations without an explicit `is_synthetic` or `data_nature` metadata flag.
- **Evidence:** Querying `GET /fhir/observation/{signal_id}` for synthetic records produced valid HL7 FHIR R4 JSON with:
  - `resourceType: "Observation"`
  - `status: "amended"`
  - `code.coding[0].code: "8867-4"` ("Heart rate")
  - `device.display: "CLINICAL_MONITOR"`
  - `subject.reference: "Patient/unknown"`
  - Zero flags, tags, or extensions indicating synthetic origin.
- **Impact:** Synthetic test data can commingle undetected with genuine patient records in clinical data warehouses and FHIR servers.
- **Contract Status:** **DATA PROVENANCE / INTEGRATION RISK.** Software/data observation only; no clinical conclusions.
- **Reproducibility Status:** REPRODUCIBLE (Verified via FHIR export).
- **Priority:** TEAM TO PRIORITIZE
- **Owner:** MHEALTH Task 4 Team
- **Recommended Next Action:** Add `is_synthetic: bool = False` to `SignalCreate` and `DatasetProvenance`, and map it to a FHIR extension tag in `fhir_service.py`.
- **Verification Required:** Ingest synthetic data and verify that the exported FHIR JSON contains `is-synthetic: true`.

---

### GAP-MH-005: Enum Normalization Discrepancy Between Direct API & CSV Ingestion
- **Source Finding ID:** AUDIT-MH-005
- **Repository:** `bhiv-MHEALTH-Production-main`
- **Capability:** Task 4 Runtime / API
- **Category:** API Contract / Input Normalization
- **Observed Condition:** In `app/ingestion/ingest.py`, CSV ingestion normalizes lowercase input values (`"heart_rate"` $\to$ `SignalType.HEART_RATE`, `"wearable"` $\to$ `SignalSource.WEARABLE_DEVICE`). However, the direct JSON endpoint `POST /signal/classify` deserializes directly into Pydantic v2 Enums without case-folding.
- **Evidence:** POSTing `{"signal_type": "heart_rate", "source": "wearable", "heart_rate": {"bpm": 72.0}}` to `/signal/classify` returned HTTP 422 Unprocessable Entity (`Input should be 'HEART_RATE', 'SPO2' ...`). Only uppercase strings (`"HEART_RATE"`) are accepted by the direct API.
- **Impact:** Inconsistent API contract causes client integration confusion between file uploads and direct JSON evaluations.
- **Contract Status:** **INPUT NORMALIZATION GAP.** CSV and JSON entry points enforce asymmetric input formatting requirements.
- **Reproducibility Status:** REPRODUCIBLE (100% reproducible with lowercase JSON).
- **Priority:** TEAM TO PRIORITIZE
- **Owner:** MHEALTH Task 4 Team
- **Recommended Next Action:** Implement a Pydantic `@field_validator` or `@model_validator` with `mode="before"` to uppercase and strip string enum inputs.
- **Verification Required:** POST lowercase enum strings to `/signal/classify` and verify HTTP 200 response.

---

### GAP-MH-006: Database Initialization Lifecycle Dependency
- **Source Finding ID:** AUDIT-MH-006
- **Repository:** `bhiv-MHEALTH-Production-main`
- **Capability:** Task 4 Runtime / API
- **Category:** Execution / Lifecycle Dependency
- **Observed Condition:** In [app/main.py:lifespan()](file:///c:/Pratik_Bhuwad/Biotech/bhiv-MHEALTH-Production-main/app/main.py#L30-L39), `init_db()` is invoked exclusively within the FastAPI lifespan context manager. If API endpoints, tests, or scripts import and call database functions directly without entering `lifespan`, database tables are not created.
- **Evidence:** Invoking `c = TestClient(app); c.post('/signal/classify', ...)` without `c.__enter__()` crashed with `sqlite3.OperationalError: no such table: signals`.
- **Impact:** Ad-hoc scripts, worker processes, or test harnesses executing outside the FastAPI lifespan fail on database operations.
- **Contract Status:** **LIFECYCLE DEPENDENCY.** Standalone module functions rely on external orchestration for table creation.
- **Reproducibility Status:** REPRODUCIBLE (100% reproducible on uninitialized databases).
- **Priority:** TEAM TO PRIORITIZE
- **Owner:** MHEALTH Task 4 Team
- **Recommended Next Action:** Call `init_db()` lazily within `get_conn()` if tables do not exist, or provide an explicit CLI command (`python -m app.database.init`).
- **Verification Required:** Delete `bhiv_mhealth.db`, invoke `insert_signal()` directly, and verify that tables are initialized automatically without error.

---

### GAP-FS-001: Unicode Dash Encoding Sensitivity in Quality Classification
- **Source Finding ID:** AUDIT-FS-001
- **Repository:** `BHIV_Formulation_Stability_T3_Day1`
- **Capability:** Sara (Formulation & Stability Evidence Intelligence)
- **Category:** Data / Character Encoding
- **Observed Condition:** In `src/evidence_intelligence.py:L18`, the allowed evidence quality set defines `"Moderate–High"` using a Unicode en-dash (`U+2013`).
- **Evidence:** When `python run.py run` executes in standard Windows console environments without active UTF-8 codepage (CP65001), the en-dash is printed as a replacement character: `"evidence_quality": "ModerateHigh"`.
- **Impact:** Downstream systems consuming console stdout or parsing non-UTF-8 outputs encounter string matching failures when filtering by quality.
- **Contract Status:** **DATA QUALITY RISK.** String literal uses typography glyphs rather than ASCII standard hyphen-minus (`-`).
- **Reproducibility Status:** REPRODUCIBLE (Observed directly in standard terminal execution).
- **Priority:** TEAM TO PRIORITIZE
- **Owner:** Sara
- **Recommended Next Action:** Replace `"Moderate–High"` with standard ASCII hyphen `"Moderate-High"` and enforce UTF-8 output streams.
- **Verification Required:** Run `python run.py run` in standard Windows terminal and verify clean ASCII output.

---

### GAP-FS-002: CLI Input Handling Lacks Graceful Validation on Unknown Subcommands
- **Source Finding ID:** AUDIT-FS-002
- **Repository:** `BHIV_Formulation_Stability_T3_Day1`
- **Capability:** Sara (Formulation & Stability Evidence Intelligence)
- **Category:** CLI / Usability
- **Observed Condition:** In `run.py`, the CLI parser requires strict subcommand syntax. Passing options without subcommands or with malformed arguments prints default `argparse` usage messages without guidance.
- **Evidence:** Executing `python run.py` raises `error: the following arguments are required: cmd`.
- **Impact:** Minor operator friction when interacting with CLI without `--help`.
- **Contract Status:** **USABILITY / CLI HANDLING.**
- **Reproducibility Status:** REPRODUCIBLE.
- **Priority:** TEAM TO PRIORITIZE
- **Owner:** Sara
- **Recommended Next Action:** Add a default help action when no subcommand is provided.
- **Verification Required:** Execute `python run.py` and verify helpful menu output.

---

### GAP-DS-001: Missing Package Hierarchy & Broken Automated Test Discovery
- **Source Finding ID:** AUDIT-DS-001
- **Repository:** `Drug_Safety_QC_Evidence_Map-main`
- **Capability:** Shravani (Drug Safety Evidence Integration)
- **Category:** Packaging / Runtime Blocker
- **Observed Condition:** Source and test files are all flattened directly into the repository root. However, test modules and integration scripts import from subpackages:
  - `test_conflicts.py` imports `from conflicts.detect_conflicts import detect_conflicts`
  - `test_drug_matcher.py` imports `from matching.drug_matcher import match_drug`
  - `test_duplicates.py` imports `from duplicates.detect_duplicates import detect_duplicates`
  - `test_human_review.py` imports `from review.human_review import review_for_validation...`
  - `test_validation.py` imports `from validation.validate_evidence import validate_record`
  None of these subpackages exist as directories in the repository.
- **Evidence:** Executing `python -m pytest` directly terminates with 6 collection errors:
  `ModuleNotFoundError: No module named 'conflicts'` / `'matching'` / `'duplicates'` / `'review'` / `'validation'`.
  When a Python diagnostic wrapper provides module aliases, all 23 tests pass in 0.06s.
- **Impact:** Any developer cloning the repository cannot execute automated tests out-of-the-box.
- **Contract Status:** **DOCUMENTED CONTRACT VIOLATED.** `HANDOVER.md` (lines 131–139) states: *"Automated tests: 23/23 PASS"*. This is false for the supplied flattened layout.
- **Reproducibility Status:** REPRODUCIBLE (Fails cleanly on fresh checkout).
- **Priority:** TEAM TO PRIORITIZE
- **Owner:** Shravani
- **Recommended Next Action:** Move implementation files into their respective subpackage directories (`conflicts/`, `matching/`, `duplicates/`, `review/`, `validation/`), or adjust imports to root-relative module imports.
- **Verification Required:** Run `pytest` directly in root without shims and verify 23 passed.

---

### GAP-DS-002: Documented CLI Command References Missing Directory Path
- **Source Finding ID:** AUDIT-DS-002
- **Repository:** `Drug_Safety_QC_Evidence_Map-main`
- **Capability:** Shravani (Drug Safety Evidence Integration)
- **Category:** Documentation / Execution Defect
- **Observed Condition:** In `HANDOVER.md:L94`, the documented execution command is:
  ```powershell
  python drug_safety_tracker.py --file sample_data/evidence_records.csv --drug "Drug A"
  ```
  However, no `sample_data/` directory exists in the repository. `evidence_records.csv` is located directly in the root directory.
- **Evidence:** Executing the documented command terminates with:
  `ERROR: CSV file not found: sample_data/evidence_records.csv`.
  Pointing to `evidence_records.csv` in the root executes successfully.
- **Impact:** Operators following repository documentation encounter immediate execution failure.
- **Contract Status:** **DOCUMENTATION DEFECT.** Documented command does not match repository filesystem structure.
- **Reproducibility Status:** REPRODUCIBLE (Fails 100% when run as documented).
- **Priority:** TEAM TO PRIORITIZE
- **Owner:** Shravani
- **Recommended Next Action:** Update `HANDOVER.md` to point to `evidence_records.csv`, or move sample data into `sample_data/`.
- **Verification Required:** Execute documented command from `HANDOVER.md` in fresh terminal.

---

### GAP-DS-003: Simplistic Conflict Detection Relies on Negative Keyword Substrings
- **Source Finding ID:** AUDIT-DS-003
- **Repository:** `Drug_Safety_QC_Evidence_Map-main`
- **Capability:** Shravani (Drug Safety Evidence Integration)
- **Category:** Analytical / Logic Limitation
- **Observed Condition:** In `detect_conflicts.py`, observation conflict detection relies on checking whether observation strings contain negative keywords (e.g. `"no clear association"` vs `"reported after exposure"`).
- **Evidence:** Source code analysis shows no semantic ontology, numerical value comparison, or clinical contradiction matrix. Contradictory dosage or statistical findings that lack specific keywords are not flagged.
- **Impact:** Subtle or quantitative clinical safety conflicts may pass undetected.
- **Contract Status:** **ANALYTICAL LIMITATION.** System detects only hardcoded demonstration keyword conflicts.
- **Reproducibility Status:** REPRODUCIBLE.
- **Priority:** TEAM TO PRIORITIZE
- **Owner:** Shravani
- **Recommended Next Action:** Define structured observation categories and formal conflict criteria across sources.
- **Verification Required:** Test conflict detection against non-keyword contradictory clinical claims.

---

### GAP-DD-001: Hardcoded Relative Parent Path in Comparison Script
- **Source Finding ID:** AUDIT-DD-001
- **Repository:** `drug-discovery--main`
- **Capability:** Shamit (TB Drug-Discovery Candidate Intelligence)
- **Category:** Execution / Path Dependency
- **Observed Condition:** In `generate_comparison.py:L3`, the CSV input path is hardcoded as:
  ```python
  CSV_FILE = "../candidate_compounds.csv"
  ```
  The script was placed in the root directory rather than a `phase4/` subfolder.
- **Evidence:** Executing `python generate_comparison.py` from the repository root crashes with:
  `FileNotFoundError: [Errno 2] No such file or directory: '../candidate_compounds.csv'`.
- **Impact:** The candidate comparison generator cannot be executed directly from the repository root.
- **Contract Status:** **EXECUTION DEFECT.** Script fails out-of-the-box due to unverified path assumption.
- **Reproducibility Status:** REPRODUCIBLE (100% reproducible on fresh checkout).
- **Priority:** TEAM TO PRIORITIZE
- **Owner:** Shamit (TMS)
- **Recommended Next Action:** Update line 3 to resolve the file dynamically:
  ```python
  from pathlib import Path
  CSV_FILE = Path(__file__).resolve().parent / "candidate_compounds.csv"
  ```
- **Verification Required:** Run `python generate_comparison.py` from repository root and verify that `candidate_comparison_results.csv` is generated.

---

### GAP-DD-002: Pipeline Relies on Missing Prerequisite JSON Artifacts
- **Source Finding ID:** AUDIT-DD-002
- **Repository:** `drug-discovery--main`
- **Capability:** Shamit (TB Drug-Discovery Candidate Intelligence)
- **Category:** Data Persistence / Missing Pipeline Artifact
- **Observed Condition:** `candidate_provenance.py` requires `candidate_records_v1.json` to execute, and `calculate_scores_v2.py` requires `candidate_records_with_provenance.json`. Neither JSON file exists in the repository.
- **Evidence:** Executing `python candidate_provenance.py` outputs:
  `ERROR: candidate_records_v1.json not found. Loaded canonical records: 0`.
  Executing `python calculate_scores_v2.py` outputs:
  `ERROR: Input file not found: ...\candidate_records_with_provenance.json. Loaded records: 0`.
- **Impact:** Core Phase 5 and Phase 6 scoring pipelines cannot be executed or reproduced from the supplied repository state.
- **Contract Status:** **DATA PIPELINE DEFECT.** Repository documentation describes Phase 5 and Phase 6 workflows, but intermediate artifacts were not committed.
- **Reproducibility Status:** REPRODUCIBLE (Fails 100% out-of-box).
- **Priority:** TEAM TO PRIORITIZE
- **Owner:** Shamit (TMS)
- **Recommended Next Action:** Commit `candidate_records_v1.json` or add an export step from `candidate_compounds.csv` to generate it automatically.
- **Verification Required:** Run `candidate_provenance.py` followed by `calculate_scores_v2.py` and verify clean execution.

---

### GAP-DD-003: Absence of Automated Test Framework
- **Source Finding ID:** AUDIT-DD-003
- **Repository:** `drug-discovery--main`
- **Capability:** Shamit (TB Drug-Discovery Candidate Intelligence)
- **Category:** Test Automation / Verifiability
- **Observed Condition:** `README.md` and `PHASE7_TEST_RESULTS.md` document 16 passed tests. However, there are no `test_*.py` files or test directories in the repository.
- **Evidence:** Running `python -m pytest` outputs `collected 0 items / no tests ran`. Tests were conducted via manual console input in an interactive CLI session.
- **Impact:** Independent automated CI/CD verification is impossible without rewriting tests into a formal testing framework.
- **Contract Status:** **TEST INTEGRITY GAP.** Documented test results are not backed by executable automated test files.
- **Reproducibility Status:** REPRODUCIBLE (`pytest` finds 0 tests).
- **Priority:** TEAM TO PRIORITIZE
- **Owner:** Shamit (TMS)
- **Recommended Next Action:** Convert the 16 documented manual test scenarios into a formal `pytest` suite in a `tests/` directory.
- **Verification Required:** Run `pytest` and verify that all 16 test cases execute and pass automatically.

---

### GAP-BIO-001: Dual-Ecosystem Divergence (Python QC Engine vs Unused Vite/TypeScript Scaffold)
- **Source Finding ID:** AUDIT-BIO-001
- **Repository:** `symmetrical-octo-invention-main`
- **Capability:** Shyamal (Biosimilarity & Preclinical Evidence QC)
- **Category:** Architecture / Build System Divergence
- **Observed Condition:** The repository contains a complete, fully tested Python QC engine (`qc_engine/engine.py`, 24 passing `pytest` tests), but also contains a complete Node.js/Vite/TypeScript web application scaffold (`package.json`, `vite.config.ts`, `src/App.tsx`, `index.html`).
- **Evidence:** Source code analysis reveals that the TypeScript web application does not call the Python QC engine, import its schemas, or consume its JSON outputs. No build script, Dockerfile, or IPC bridge links the two codebases.
- **Impact:** Repository contains unmaintained boilerplate frontend code that diverges from the operational Python QC implementation.
- **Contract Status:** **ARCHITECTURAL DIVERGENCE.** Unclear whether the repository is intended as a CLI Python library or a full-stack web application.
- **Reproducibility Status:** REPRODUCIBLE (Two disconnected frameworks confirmed).
- **Priority:** TEAM TO PRIORITIZE
- **Owner:** Shyamal
- **Recommended Next Action:** Document the boundary between the Python QC engine and the web UI, or remove the unused frontend scaffold if out of scope.
- **Verification Required:** Align project documentation on supported entry points.

---

## 4. Phase 5 Decision & Contract Verification Status

**Audit Pass:** Phase 5 Decision & Contract Verification (READ-ONLY)  
**Authoritative Reference Documents:**
- [BIOTECH_CONTRACT_VERIFICATION_REGISTER.md](file:///c:/Pratik_Bhuwad/Biotech/BIOTECH_CONTRACT_VERIFICATION_REGISTER.md)
- [BIOTECH_DECISION_VERIFICATION_REGISTER.md](file:///c:/Pratik_Bhuwad/Biotech/BIOTECH_DECISION_VERIFICATION_REGISTER.md)
- [BIOTECH_PHASE5_DECISION_CONTRACT_VERIFICATION.md](file:///c:/Pratik_Bhuwad/Biotech/BIOTECH_PHASE5_DECISION_CONTRACT_VERIFICATION.md)

### Verification Status of Existing Gaps
* **GAP-PV-001 through GAP-PV-004:** **CONFIRMED IN PHASE 5**. Flattened root files, packaging shims, and missing schema integration confirmed.
* **GAP-MH-001 through GAP-MH-006:** **CONFIRMED IN PHASE 5**. Test runner incompatibility, DB locks, and unflagged synthetic patient exports confirmed.
* **GAP-FS-001 through GAP-FS-002:** **CONFIRMED IN PHASE 5**. Isolated CLI execution and private Polars/JSON schemas confirmed.
* **GAP-DS-001 through GAP-DS-003:** **CONFIRMED IN PHASE 5**. Missing package declarations, flat module imports, and isolated data structures confirmed.
* **GAP-DD-001 through GAP-DD-003:** **CONFIRMED IN PHASE 5**. Runtime execution re-verified: `generate_comparison.py` crashes on `../candidate_compounds.csv`; `candidate_provenance.py` crashes on missing `candidate_records_v1.json`; 0 automated tests collected.
* **GAP-BIO-001:** **CONFIRMED IN PHASE 5**. Python QC engine and detached Vite scaffold divergence confirmed.

### New Cross-Repository Contract Gaps (Phase 5 Additions)

| Gap ID | Source Contract | In Scope Repositories | Category | Observed Condition | Impact | Priority | Owner | Recommended Next Action |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **GAP-CTR-001** | `CONTRACT-PV-DS-001` | Sarvesh (Task 1) & Shravani (Task 3) | Contract / Schema Incompatibility | Passing Sarvesh's `EV-000001` to Shravani's `validate_record()` fails with 4 fatal errors and 10 missing fields. | Automated signal pipeline between Task 1 and Task 3 is completely broken out-of-the-box. | TEAM TO PRIORITIZE | Team / Shravani & Sarvesh | Implement bidirectional `BiotechSafetyAdapter` mapping `EvidenceRecord` $\to$ `BHIV_Drug_Safety_Evidence`. |
| **GAP-CTR-002** | `CONTRACT-MH-001` & `CONTRACT-PROV-001` | MHEALTH (Task 4) & Ecosystem | Provenance / Clinical Standards | MHEALTH exports synthetic subjects (`subj-01`) as genuine HL7 FHIR R4 Observations without an `is_synthetic` extension tag. | Risk of corrupting external EHR/FHIR stores with mock test data. | TEAM TO PRIORITIZE | MHEALTH Team | Add `is_synthetic: bool = False` to `DatasetProvenance` and emit official FHIR StructureDefinition extension. |
| **GAP-CTR-003** | `CONTRACT-ID-001` | All 6 Repositories | Architecture / Entity Resolution | Repositories generate mutually incompatible identifiers (`EV-000001`, `EV-001`, `E001`, `C-001`, `SYN-001`, `id: 1`). | Impossible to perform cross-repository analytical joins on identical compounds, batches, or evidence. | TEAM TO PRIORITIZE | Team Architecture Lead | Establish global namespaced identifier scheme: `BIO:<DOMAIN>:<ENTITY>:<ID>`. |

