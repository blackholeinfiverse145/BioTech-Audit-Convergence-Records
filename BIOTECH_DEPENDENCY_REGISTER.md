# BIOTECH — TEAM DEPENDENCY REGISTER
**Auditor:** Pratik Bhuvad  
**Role:** AI/ML + Data Audit Owner — Independent verification of reproducibility and analytical/data integrity  
**Date:** 2026-10-05  
**Status:** Phase 4 Six-Repository System-Wide Audit (Authoritative & Verified)

---

## 1. Executive Summary & Overview

This Dependency Register provides an authoritative catalog of all runtime, testing, environmental, external service, dataset, and architectural dependencies across the complete six-repository Biotech implementation set:
1. `Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main` (Pharmacovigilance Pipeline v1 — Sarvesh Patil)
2. `bhiv-MHEALTH-Production-main` (BHIV MHEALTH Production Runtime Task 4 — MHEALTH Task 4 Team)
3. `BHIV_Formulation_Stability_T3_Day1` (Formulation & Stability Evidence Intelligence — Sara)
4. `Drug_Safety_QC_Evidence_Map-main` (Drug Safety Evidence Integration — Shravani)
5. `drug-discovery--main` (TB Drug-Discovery Candidate Intelligence — Shamit / TMS)
6. `symmetrical-octo-invention-main` (Biosimilarity & Preclinical Evidence QC — Shyamal)

Dependencies are documented only when supported by verifiable repository evidence (e.g., `requirements.txt`, `package.json`, import statements, configuration files, runtime behaviors, or documentation claims). Each entry articulates its technical role, failure modes, impact on reproducibility, and required maintenance actions.

---

## 2. Dependency Summary Table

| Dependency ID | Repository | Dependency Name | Type | Current Status | Required For | Failure Mode | Owner |
|---|---|---|---|---|---|---|---|
| **DEP-PV-001** | Repo 1 (Pharmacovigilance) | OpenFDA / FAERS REST API | External Service | UNCONTROLLED | Live Data Ingestion | HTTP 500 / 404 / Network Timeout | Sarvesh Patil (Documented Developer) |
| **DEP-PV-002** | Repo 1 (Pharmacovigilance) | `src` Directory Package Hierarchy | Architectural Layout | DEFECTIVE (Missing) | Direct CLI & Test Execution | `ModuleNotFoundError: No module named 'src'` | Sarvesh Patil (Documented Developer) |
| **DEP-PV-003** | Repo 1 (Pharmacovigilance) | `requests` (HTTP Client) | Runtime Library | UNPINNED (Missing manifest) | HTTP Communication | Incompatible version or missing dependency | Sarvesh Patil (Documented Developer) |
| **DEP-PV-004** | Repo 1 (Pharmacovigilance) | `evidence_output.json` | Local File System | UNCONTROLLED (Hardcoded path) | Evidence Persistence | Concurrency collisions; path not found | Sarvesh Patil (Documented Developer) |
| **DEP-PV-005** | Repo 1 (Pharmacovigilance) | Python 3.10+ Standard Library | Runtime Platform | SATISFIED | Dataclasses & Hashing | Syntax / Feature incompatibility | Sarvesh Patil (Documented Developer) |
| **DEP-MH-001** | Repo 2 (MHEALTH) | Starlette TestClient / HTTPX | Testing Dependency | INCOMPATIBLE (`httpx >= 0.28`) | Automated Test Execution | `TypeError: unexpected keyword 'app'` | MHEALTH Task 4 Team |
| **DEP-MH-002** | Repo 2 (MHEALTH) | SQLite File Handle Lifecycle | Resource / OS Platform | DEFECTIVE on Windows | Windows Fixture Cleanup | `PermissionError: [WinError 32]` | MHEALTH Task 4 Team |
| **DEP-MH-003** | Repo 2 (MHEALTH) | FastAPI 0.111.0 / Pydantic 2.7.1 | Core Framework | PINNED in `requirements.txt` | REST API & Data Validation | Framework breaking changes if upgraded | MHEALTH Task 4 Team |
| **DEP-MH-004** | Repo 2 (MHEALTH) | `configs/thresholds.json` | Local Configuration | SATISFIED | Telemetry Classification | `FileNotFoundError` if config moved | MHEALTH Task 4 Team |
| **DEP-MH-005** | Repo 2 (MHEALTH) | PostgreSQL / `psycopg2-binary` | Optional Database | OPTIONAL / DOCUMENTED | Production Scalability | Database connection failure if URL set | MHEALTH Task 4 Team |
| **DEP-MH-006** | Repo 2 (MHEALTH) | `API_KEYS` Environment Variable | Security / Environment | SATISFIED (Dev fallback) | Route Authentication | HTTP 401 Unauthorized | MHEALTH Task 4 Team |
| **DEP-MH-007** | Repo 2 (MHEALTH) | LOINC & UCUM Terminology | Standards / Data | SATISFIED (Static mapping) | HL7 FHIR R4 Observation Export | Invalid coding in external registries | MHEALTH Task 4 Team |
| **DEP-FS-001** | `BHIV_Formulation_Stability_T3_Day1` | Python 3.10+ Standard Library | Runtime Platform | SATISFIED | Validation & Hashing | Missing standard library features | Sara |
| **DEP-FS-002** | `BHIV_Formulation_Stability_T3_Day1` | `pytest>=8,<9` | Testing Library | PINNED in `requirements.txt` | Automated Unit Tests | Test framework incompatibilities | Sara |
| **DEP-FS-003** | `BHIV_Formulation_Stability_T3_Day1` | `data/canonical_evidence_records.csv` | Local File System | SATISFIED | Evidence Data Ingestion | `FileNotFoundError` if path altered | Sara |
| **DEP-DS-001** | `Drug_Safety_QC_Evidence_Map-main` | Subpackage Layout Hierarchy | Architectural Layout | DEFECTIVE (Missing) | Automated Test Discovery | `ModuleNotFoundError: No module named 'conflicts'` | Shravani |
| **DEP-DS-002** | `Drug_Safety_QC_Evidence_Map-main` | `evidence_records.csv` | Local File System | DEFECTIVE (Path mismatch) | Safety Tracker Execution | `FileNotFoundError: sample_data/evidence_records.csv` | Shravani |
| **DEP-DS-003** | `Drug_Safety_QC_Evidence_Map-main` | `canonical_schema.json` | Schema Specification | SATISFIED | JSON Schema Validation | `FileNotFoundError` if schema moved | Shravani |
| **DEP-DD-001** | `drug-discovery--main` | `candidate_compounds.csv` | Local File System | DEFECTIVE (Relative path) | Candidate Comparison Generation | `FileNotFoundError: ../candidate_compounds.csv` | Shamit (TMS) |
| **DEP-DD-002** | `drug-discovery--main` | `candidate_records_v1.json` | Prerequisite Data Artifact | DEFECTIVE (Missing file) | Provenance Pipeline Execution | `ERROR: candidate_records_v1.json not found` | Shamit (TMS) |
| **DEP-DD-003** | `drug-discovery--main` | Python Standard Library (`csv`, `json`) | Runtime Platform | SATISFIED | Candidate Ingestion & Scoring | Missing standard library features | Shamit (TMS) |
| **DEP-BIO-001** | `symmetrical-octo-invention-main` | `schemas/evidence_qc_schema_v1.0.0.json` | Schema Specification | SATISFIED | Canonical QC Validation | `FileNotFoundError` if schema moved | Shyamal |
| **DEP-BIO-002** | `symmetrical-octo-invention-main` | `data/synthetic_evidence.json` | Synthetic Test Fixture | SATISFIED | QC Engine Batch Processing | `FileNotFoundError` if fixture moved | Shyamal |
| **DEP-BIO-003** | `symmetrical-octo-invention-main` | Node.js / Vite / TypeScript Toolchain | Secondary Frontend Platform | DISCONNECTED / UNUSED | Web UI Presentation | Build errors if frontend executed | Shyamal |

---

## 3. Detailed Dependency Register Entries

### DEP-PV-001: OpenFDA External REST API Service
- **Dependency ID:** DEP-PV-001
- **Repository:** `Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main`
- **Dependency:** OpenFDA / FAERS Public REST API (`https://api.fda.gov/drug/event.json`)
- **Type:** External Web Service
- **Purpose:** Upstream data source for post-marketing drug adverse event reports.
- **Required For:** Live evidence acquisition in `openfda_connector.py:search_drug()` and runtime execution in `app.py`.
- **Evidence:** [openfda_connector.py:L5](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main/openfda_connector.py#L5): `OPENFDA_URL = "https://api.fda.gov/drug/event.json"`.
- **Current Status:** **UNCONTROLLED.** Pipeline relies directly on live public internet access to US FDA servers.
- **Failure Mode:** Network latency, rate limits (exceeding 240 requests/minute without API key), upstream HTTP 500 query parsing errors, or complete endpoint unavailability.
- **Reproduction Impact:** Offline reproduction is impossible. If the OpenFDA service is unreachable, live pipeline execution fails completely (`success: False`).
- **Owner:** Sarvesh Patil (Documented Developer)
- **Action Required:** Implement local response caching or mock fixture fallback to allow offline testing and decoupled deterministic reproduction.

---

### DEP-PV-002: Repository Package Structure (`src/` Namespace)
- **Dependency ID:** DEP-PV-002
- **Repository:** `Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main`
- **Dependency:** Root-level `src` module packaging hierarchy
- **Type:** Architectural Layout / Import Namespace
- **Purpose:** Enables Python interpreter to resolve imports formatted as `from src.<module> import ...`.
- **Required For:** Direct execution of `app.py` and automated test discovery via `pytest`.
- **Evidence:** [app.py:L16](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main/app.py#L16): `from src.openfda_connector import search_drug`, `from src.normalize_evidence import normalize_record`.
- **Current Status:** **DEFECTIVE (Missing).** The repository files are flattened into the root directory with no `src/` folder present.
- **Failure Mode:** `ModuleNotFoundError: No module named 'src'`.
- **Reproduction Impact:** Complete out-of-the-box execution blockage. Requires an external diagnostic wrapper aliasing `sys.modules['src']` to execute.
- **Owner:** Sarvesh Patil (Documented Developer)
- **Action Required:** Move implementation files into a `src/` directory or update import statements to root-relative paths.

---

### DEP-PV-003: `requests` HTTP Library
- **Dependency ID:** DEP-PV-003
- **Repository:** `Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main`
- **Dependency:** `requests` third-party HTTP client
- **Type:** Runtime Python Library
- **Purpose:** Synchronous HTTP communication with OpenFDA API.
- **Required For:** Executing `openfda_connector.py`.
- **Evidence:** [openfda_connector.py:L2](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main/openfda_connector.py#L2): `import requests`.
- **Current Status:** **UNPINNED / UNMANIFESTED.** No `requirements.txt` or `pyproject.toml` exists in the repository.
- **Failure Mode:** `ModuleNotFoundError: No module named 'requests'` on clean Python installations.
- **Reproduction Impact:** Prevents execution in standard bare virtual environments lacking pre-installed packages.
- **Owner:** Sarvesh Patil (Documented Developer)
- **Action Required:** Create a `requirements.txt` file pinning `requests>=2.31.0`.

---

### DEP-PV-004: Local Evidence File Path (`evidence_output.json`)
- **Dependency ID:** DEP-PV-004
- **Repository:** `Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main`
- **Dependency:** Local relative filesystem path `evidence_output.json`
- **Type:** Local File Storage
- **Purpose:** Persistent target for serialized `EvidenceRecord` JSON arrays.
- **Required For:** Persistence in `evidence_store.py` and `app.py`.
- **Evidence:** [app.py:L14](file:///c:/Pratik_Bhuwad/Biotech/Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main/app.py#L14): `DEFAULT_OUTPUT_FILE = "evidence_output.json"`.
- **Current Status:** **UNCONTROLLED.** Hardcoded relative path sensitive to the current working directory.
- **Failure Mode:** Write permission errors, concurrent overwrite collisions, or missing output when executed from alternative directories.
- **Reproduction Impact:** Running `app.py` writes directly to disk, altering the repository state.
- **Owner:** Sarvesh Patil (Documented Developer)
- **Action Required:** Make output path configurable and parameterize storage targets.

---

### DEP-PV-005: Python 3.10+ Standard Library Platform
- **Dependency ID:** DEP-PV-005
- **Repository:** `Live-Pharmacovigilance-Evidence-And-Drug-Safety-Intelligence-Pipeline-v1-AI-Biotech-Core-Task-1-main`
- **Dependency:** Python 3.10+ Standard Library (`dataclasses`, `hashlib`, `json`, `datetime`)
- **Type:** Runtime Platform
- **Purpose:** Core object modeling, MD5 hashing, and data serialization.
- **Required For:** Entire pipeline execution.
- **Evidence:** `from dataclasses import dataclass`, `import hashlib`.
- **Current Status:** **SATISFIED.** Functions cleanly on Python 3.10, 3.11, and 3.12.
- **Failure Mode:** Syntax errors on Python versions `< 3.10` due to type union syntax (`str | None`).
- **Reproduction Impact:** Requires Python 3.10 or higher.
- **Owner:** Sarvesh Patil (Documented Developer)
- **Action Required:** Document minimum Python version in `README.md`.

---

### DEP-MH-001: Starlette TestClient / Modern HTTPX Compatibility
- **Dependency ID:** DEP-MH-001
- **Repository:** `bhiv-MHEALTH-Production-main`
- **Dependency:** Compatibility between Starlette 0.27 and HTTPX Client
- **Type:** Testing Dependency Relationship
- **Purpose:** Orchestrating API requests in automated unit/integration tests without a live web server.
- **Required For:** Executing `pytest tests/test_task4.py`.
- **Evidence:** `starlette/testclient.py:L399` passes `app=app` to `httpx.Client.__init__()`.
- **Current Status:** **INCOMPATIBLE** with `httpx >= 0.28.0`.
- **Failure Mode:** `TypeError: Client.__init__() got an unexpected keyword argument 'app'`.
- **Reproduction Impact:** Fails all 39 tests out-of-the-box on fresh environments installing the latest HTTPX.
- **Owner:** MHEALTH Task 4 Team
- **Action Required:** Pin `httpx==0.27.0` in `requirements.txt` or upgrade Starlette.

---

### DEP-MH-002: SQLite File Handle Lifecycle on Windows
- **Dependency ID:** DEP-MH-002
- **Repository:** `bhiv-MHEALTH-Production-main`
- **Dependency:** Operating system file handle release behavior
- **Type:** Platform / OS Resource Management
- **Purpose:** Managing SQLite database connection lifecycles.
- **Required For:** Executing tests that create and unlink temporary databases.
- **Evidence:** `app/database/db.py:get_conn()` leaves SQLite connections unclosed.
- **Current Status:** **DEFECTIVE on Windows.**
- **Failure Mode:** `PermissionError: [WinError 32] The process cannot access the file because it is being used by another process`.
- **Reproduction Impact:** Fails test suite during fixture teardown on Windows.
- **Owner:** MHEALTH Task 4 Team
- **Action Required:** Ensure all SQLite connections are closed before file unlinking.

---

### DEP-MH-003: FastAPI 0.111.0 & Pydantic 2.7.1 Framework
- **Dependency ID:** DEP-MH-003
- **Repository:** `bhiv-MHEALTH-Production-main`
- **Dependency:** FastAPI web framework and Pydantic v2 data validation
- **Type:** Core Runtime Framework
- **Purpose:** REST API routing, request validation, serialization, and OpenAPI documentation.
- **Required For:** Entire API service operation (`app/main.py`).
- **Evidence:** `requirements.txt`: `fastapi==0.111.0`, `pydantic==2.7.1`.
- **Current Status:** **PINNED & SATISFIED.**
- **Failure Mode:** Breaking changes in Pydantic v2 if upgraded without validation.
- **Reproduction Impact:** Cleanly installs and runs under specified versions.
- **Owner:** MHEALTH Task 4 Team
- **Action Required:** Maintain pinned versions in `requirements.txt`.

---

### DEP-MH-004: Telemetry Classification Rules Configuration (`configs/thresholds.json`)
- **Dependency ID:** DEP-MH-004
- **Repository:** `bhiv-MHEALTH-Production-main`
- **Dependency:** `configs/thresholds.json` local file
- **Type:** Static Local Configuration
- **Purpose:** Defines numerical thresholds for classifying vital signs (`NORMAL`, `WARNING`, `CRITICAL`).
- **Required For:** `app/classifier/engine.py:load_rules()` and `classify_signal()`.
- **Evidence:** [app/classifier/engine.py:L14](file:///c:/Pratik_Bhuwad/Biotech/bhiv-MHEALTH-Production-main/app/classifier/engine.py#L14): `DEFAULT_CONFIG = Path(__file__).resolve().parents[2] / "configs" / "thresholds.json"`.
- **Current Status:** **SATISFIED.** File exists and conforms to schema.
- **Failure Mode:** `FileNotFoundError` if config is deleted or moved, crashing classification endpoints with HTTP 500.
- **Reproduction Impact:** Deterministic threshold evaluation relies on this file remaining unchanged.
- **Owner:** MHEALTH Task 4 Team
- **Action Required:** Implement fallback hardcoded rules in case configuration file is missing.

---

### DEP-MH-005: Optional PostgreSQL Database (`psycopg2-binary`)
- **Dependency ID:** DEP-MH-005
- **Repository:** `bhiv-MHEALTH-Production-main`
- **Dependency:** PostgreSQL relational database and `psycopg2-binary` driver
- **Type:** Optional Relational Database
- **Purpose:** Production database storage when `DATABASE_URL` environment variable is set.
- **Required For:** Production deployment (optional; SQLite used as default fallback).
- **Evidence:** `requirements.txt`: `psycopg2-binary==2.9.9`; `app/database/db.py:get_conn()`.
- **Current Status:** **OPTIONAL / VERIFIED.** SQLite used during audit tests.
- **Failure Mode:** OperationalError if PostgreSQL connection string is invalid or host unreachable.
- **Reproduction Impact:** None for local test execution; relevant only for production scaling.
- **Owner:** MHEALTH Task 4 Team
- **Action Required:** Maintain SQLite default fallback for local developer reproducibility.

---

### DEP-MH-006: API Security Header (`API_KEYS` Environment Variable)
- **Dependency ID:** DEP-MH-006
- **Repository:** `bhiv-MHEALTH-Production-main`
- **Dependency:** `API_KEYS` environment variable and `X-API-Key` HTTP header
- **Type:** Security / Environment Configuration
- **Purpose:** Authenticating REST API requests.
- **Required For:** All protected endpoints (`/signal/classify`, `/dataset/upload`, `/signal/export/fhir`).
- **Evidence:** `app/core/auth.py`: `DEFAULT_KEYS = ["dev-key-12345", "test-key-67890"]`.
- **Current Status:** **SATISFIED.** Built-in development fallback keys allow local testing without configuration.
- **Failure Mode:** HTTP 401 Unauthorized if missing or invalid key passed.
- **Reproduction Impact:** Tests pass using `dev-key-12345`.
- **Owner:** MHEALTH Task 4 Team
- **Action Required:** Ensure production deployment overrides default development keys.

---

### DEP-MH-007: Clinical Terminology Standards (LOINC & UCUM)
- **Dependency ID:** DEP-MH-007
- **Repository:** `bhiv-MHEALTH-Production-main`
- **Dependency:** LOINC observation codes and UCUM units of measure
- **Type:** External Standards Specification
- **Purpose:** Mapping internal signal types to internationally recognized clinical standards in HL7 FHIR exports.
- **Required For:** `app/services/fhir_service.py:to_fhir_observation()`.
- **Evidence:** [app/services/fhir_service.py](file:///c:/Pratik_Bhuwad/Biotech/bhiv-MHEALTH-Production-main/app/services/fhir_service.py): Mappings for `8867-4` (`/min`), `8480-6` (`mm[Hg]`), `8462-4` (`mm[Hg]`), `2708-6` (`%`), `8310-5` (`Cel`).
- **Current Status:** **SATISFIED.** Static hardcoded mapping dictionary.
- **Failure Mode:** Emits invalid or unrecognized LOINC codes if new unsupported signal types are introduced.
- **Reproduction Impact:** Deterministic mapping confirmed.
- **Owner:** MHEALTH Task 4 Team
- **Action Required:** Add unit tests validating LOINC and UCUM conformance against standard FHIR R4 registries.

---

### DEP-FS-001: Python 3.10+ Standard Library Platform (Sara)
- **Dependency ID:** DEP-FS-001
- **Repository:** `BHIV_Formulation_Stability_T3_Day1`
- **Dependency:** Python 3.10+ Standard Library (`csv`, `json`, `hashlib`, `pathlib`, `argparse`)
- **Type:** Runtime Platform
- **Purpose:** Core parsing, validation, duplicate analysis, and structured reporting.
- **Required For:** Entire repository operation (`run.py`, `src/evidence_intelligence.py`).
- **Evidence:** `import csv, json, hashlib`, `from pathlib import Path`.
- **Current Status:** **SATISFIED.** Zero external third-party dependencies required for runtime execution.
- **Failure Mode:** None under standard Python 3.10+.
- **Reproduction Impact:** Exceptional reproducibility; executes immediately out-of-the-box.
- **Owner:** Sara
- **Action Required:** Maintain standard library design for zero-dependency execution.

---

### DEP-FS-002: Pytest Test Framework (Sara)
- **Dependency ID:** DEP-FS-002
- **Repository:** `BHIV_Formulation_Stability_T3_Day1`
- **Dependency:** `pytest>=8,<9`
- **Type:** Testing Library
- **Purpose:** Automated test discovery and execution.
- **Required For:** Executing `python -m pytest -q` (11 unit tests).
- **Evidence:** `requirements.txt`: `pytest>=8,<9`; `pytest.ini`.
- **Current Status:** **SATISFIED.** 11/11 tests pass in 0.08s out-of-the-box.
- **Failure Mode:** Incompatible pytest major versions if upgraded to 9+.
- **Reproduction Impact:** Clean, verifiable out-of-the-box test pass.
- **Owner:** Sara
- **Action Required:** Maintain pinned version constraint.

---

### DEP-FS-003: Local Static Dataset (`data/canonical_evidence_records.csv`)
- **Dependency ID:** DEP-FS-003
- **Repository:** `BHIV_Formulation_Stability_T3_Day1`
- **Dependency:** `data/canonical_evidence_records.csv`
- **Type:** Local File Storage
- **Purpose:** Source dataset containing 8 formulation stability records.
- **Required For:** `run.py run`, `search`, `validate`, `duplicates`, `json`.
- **Evidence:** `DATA=ROOT/'data'/'canonical_evidence_records.csv'`.
- **Current Status:** **SATISFIED.** File exists and is tracked in git.
- **Failure Mode:** `FileNotFoundError` if data folder is moved or renamed.
- **Reproduction Impact:** Deterministic execution relies on this file.
- **Owner:** Sara
- **Action Required:** Preserve relative path resolution via `Path(__file__).parent`.

---

### DEP-DS-001: Subpackage Directory Hierarchy (Shravani)
- **Dependency ID:** DEP-DS-001
- **Repository:** `Drug_Safety_QC_Evidence_Map-main`
- **Dependency:** Directory hierarchy for `conflicts`, `matching`, `duplicates`, `review`, `validation`
- **Type:** Architectural Layout / Import Namespace
- **Purpose:** Allows Python interpreter to resolve imports in test modules and integration scripts.
- **Required For:** Direct execution of `pytest` and `evidence_integration.py`.
- **Evidence:** `test_conflicts.py`: `from conflicts.detect_conflicts import detect_conflicts`.
- **Current Status:** **DEFECTIVE (Missing).** Files are flattened into repository root without subpackage folders.
- **Failure Mode:** `ModuleNotFoundError: No module named 'conflicts'`.
- **Reproduction Impact:** Fails automated test discovery out-of-the-box (6 collection errors). Requires diagnostic packaging shim.
- **Owner:** Shravani
- **Action Required:** Restore directory hierarchy or update import syntax.

---

### DEP-DS-002: Local Evidence CSV Dataset (`evidence_records.csv`)
- **Dependency ID:** DEP-DS-002
- **Repository:** `Drug_Safety_QC_Evidence_Map-main`
- **Dependency:** `evidence_records.csv` local dataset
- **Type:** Local File Storage
- **Purpose:** Ingestion dataset for `drug_safety_tracker.py`.
- **Required For:** Drug safety tracker execution.
- **Evidence:** `HANDOVER.md` documents `sample_data/evidence_records.csv`, but file resides in root `evidence_records.csv`.
- **Current Status:** **DEFECTIVE (Path Mismatch).**
- **Failure Mode:** `ERROR: CSV file not found: sample_data/evidence_records.csv`.
- **Reproduction Impact:** Documented command fails out-of-the-box unless path is corrected.
- **Owner:** Shravani
- **Action Required:** Create `sample_data/` folder or update documentation.

---

### DEP-DS-003: Canonical Evidence Schema (`canonical_schema.json`)
- **Dependency ID:** DEP-DS-003
- **Repository:** `Drug_Safety_QC_Evidence_Map-main`
- **Dependency:** `canonical_schema.json`
- **Type:** Schema Specification
- **Purpose:** Validating multi-source drug safety records.
- **Required For:** `validate_evidence.py` and integration tests.
- **Evidence:** `canonical_schema.json` in repository root.
- **Current Status:** **SATISFIED.** File exists and conforms to JSON Schema specifications.
- **Failure Mode:** Schema validation errors if records omit mandatory fields.
- **Reproduction Impact:** Deterministic validation confirmed.
- **Owner:** Shravani
- **Action Required:** Maintain schema versioning.

---

### DEP-DD-001: Local Candidate CSV Dataset (`candidate_compounds.csv`)
- **Dependency ID:** DEP-DD-001
- **Repository:** `drug-discovery--main`
- **Dependency:** `candidate_compounds.csv`
- **Type:** Local File Storage
- **Purpose:** Source dataset containing 10 TB candidate compounds.
- **Required For:** `drug_discovery_pipeline.py`, `calculate_completeness.py`, `generate_comparison.py`.
- **Evidence:** `candidate_compounds.csv` in repository root.
- **Current Status:** **DEFECTIVE (Relative Path).** `generate_comparison.py` hardcodes `../candidate_compounds.csv`.
- **Failure Mode:** `FileNotFoundError: [Errno 2] No such file or directory: '../candidate_compounds.csv'`.
- **Reproduction Impact:** Comparison script crashes when run from repository root.
- **Owner:** Shamit (TMS)
- **Action Required:** Update path in `generate_comparison.py` to `Path(__file__).parent / 'candidate_compounds.csv'`.

---

### DEP-DD-002: Prerequisite Intermediate JSON Datasets (Shamit)
- **Dependency ID:** DEP-DD-002
- **Repository:** `drug-discovery--main`
- **Dependency:** `candidate_records_v1.json` and `candidate_records_with_provenance.json`
- **Type:** Data Pipeline Intermediate Artifact
- **Purpose:** Pipeline input for `candidate_provenance.py` and `calculate_scores_v2.py`.
- **Required For:** Phase 5 provenance enrichment and Phase 6 multi-criteria scoring.
- **Evidence:** [candidate_provenance.py:L7](file:///c:/Pratik_Bhuwad/Biotech/drug-discovery--main/candidate_provenance.py#L7): `INPUT_FILE = BASE_DIR / "candidate_records_v1.json"`.
- **Current Status:** **DEFECTIVE (Missing Files).** Neither file is present in the repository.
- **Failure Mode:** `ERROR: candidate_records_v1.json not found` and `ERROR: Input file not found: candidate_records_with_provenance.json`.
- **Reproduction Impact:** Phase 5 and Phase 6 advanced scoring pipelines fail out-of-the-box.
- **Owner:** Shamit (TMS)
- **Action Required:** Commit intermediate JSON datasets or write an automated transformation script from `candidate_compounds.csv`.

---

### DEP-DD-003: Python Standard Library Platform (Shamit)
- **Dependency ID:** DEP-DD-003
- **Repository:** `drug-discovery--main`
- **Dependency:** Python Standard Library (`csv`, `json`, `pathlib`)
- **Type:** Runtime Platform
- **Purpose:** Candidate search, filtering, and completeness calculation.
- **Required For:** `drug_discovery_pipeline.py` and `calculate_completeness.py`.
- **Evidence:** `import csv, json`, `from pathlib import Path`.
- **Current Status:** **SATISFIED.** Zero third-party dependencies required for core operations.
- **Failure Mode:** None under standard Python.
- **Reproduction Impact:** Core search and completeness scripts execute cleanly.
- **Owner:** Shamit (TMS)
- **Action Required:** Maintain standard library design for core CLI tools.

---

### DEP-BIO-001: Versioned QC Schema (`schemas/evidence_qc_schema_v1.0.0.json`)
- **Dependency ID:** DEP-BIO-001
- **Repository:** `symmetrical-octo-invention-main`
- **Dependency:** `schemas/evidence_qc_schema_v1.0.0.json`
- **Type:** Schema Specification
- **Purpose:** Defines structural validation rules and quality states for biosimilarity evidence.
- **Required For:** `qc_engine/engine.py` and test suite.
- **Evidence:** `schemas/evidence_qc_schema_v1.0.0.json` exists and is versioned.
- **Current Status:** **SATISFIED.**
- **Failure Mode:** Schema validation failure if input records do not conform.
- **Reproduction Impact:** Fully reproducible deterministic QC validation.
- **Owner:** Shyamal
- **Action Required:** Maintain schema version compatibility.

---

### DEP-BIO-002: Synthetic Test Fixtures (Shyamal)
- **Dependency ID:** DEP-BIO-002
- **Repository:** `symmetrical-octo-invention-main`
- **Dependency:** `data/synthetic_evidence.json` and `sample_data/candidate_evidence.csv`
- **Type:** Local File Storage / Test Fixtures
- **Purpose:** Test input data for QC engine and evidence tracker.
- **Required For:** `qc_engine/engine.py`, `evidence_tracker.py`, and test suite.
- **Evidence:** Files exist and contain 11 JSON records and 10 CSV records.
- **Current Status:** **SATISFIED.**
- **Failure Mode:** `FileNotFoundError` if sample directories are altered.
- **Reproduction Impact:** All 24 tests pass cleanly out-of-the-box.
- **Owner:** Shyamal
- **Action Required:** Preserve test fixture paths.

---

### DEP-BIO-003: Disconnected Node.js / Vite / TypeScript Toolchain (Shyamal)
- **Dependency ID:** DEP-BIO-003
- **Repository:** `symmetrical-octo-invention-main`
- **Dependency:** Node.js, npm, Vite, React 19, TypeScript
- **Type:** Secondary Frontend Ecosystem
- **Purpose:** Scaffolding for a proposed web-based evidence browser UI.
- **Required For:** Web application execution (`npm run dev`).
- **Evidence:** `package.json`, `vite.config.ts`, `src/App.tsx`, `index.html`.
- **Current Status:** **DISCONNECTED / UNUSED.** Does not interact with Python QC engine.
- **Failure Mode:** npm dependency vulnerabilities or build drift if Node environment is unconfigured.
- **Reproduction Impact:** Zero impact on the operational Python QC engine (which passes 24/24 tests independently).
- **Owner:** Shyamal
- **Action Required:** Clarify role of web UI or build an explicit API bridge to the Python QC engine.

---

## 4. Phase 5 Decision & Contract Verification Status

**Audit Pass:** Phase 5 Decision & Contract Verification (READ-ONLY)  
**Authoritative Reference Documents:**
- [BIOTECH_CONTRACT_VERIFICATION_REGISTER.md](file:///c:/Pratik_Bhuwad/Biotech/BIOTECH_CONTRACT_VERIFICATION_REGISTER.md)
- [BIOTECH_DECISION_VERIFICATION_REGISTER.md](file:///c:/Pratik_Bhuwad/Biotech/BIOTECH_DECISION_VERIFICATION_REGISTER.md)
- [BIOTECH_PHASE5_DECISION_CONTRACT_VERIFICATION.md](file:///c:/Pratik_Bhuwad/Biotech/BIOTECH_PHASE5_DECISION_CONTRACT_VERIFICATION.md)

### Verification of Baseline Dependencies
* **All 24 Technical & Environmental Dependencies (DEP-PV-001..005, DEP-MH-001..007, DEP-FS-001..003, DEP-DS-001..004, DEP-DD-001..003, DEP-BIO-001..004):** **CONFIRMED IN PHASE 5**.
* **Missing Dependency in Shamit's Repo (DEP-DD-002):** **CONFIRMED DEFECTIVE**. Execution of `candidate_provenance.py` re-verified: crashes due to missing `candidate_records_v1.json`.

### New Cross-Repository Contract & Adapter Dependencies (Phase 5 Additions)

| Dependency ID | Target Systems | Dependency Name | Dependency Type | Status | Failure Mode if Missing | Owner |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **DEP-CTR-001** | Sarvesh (Task 1) $\to$ Shravani (Task 3) | `BiotechSafetyAdapter` | Code Adapter / Mapper | **MISSING (IMPLEMENTATION REQUIRED)** | `validate_record()` rejects Task 1 signals with 4 fatal validation errors. | Shravani & Sarvesh |
| **DEP-CTR-002** | All 6 Repositories | Standardized `pyproject.toml` | Build System Manifest | **MISSING (IMPLEMENTATION REQUIRED)** | Test discovery failure out-of-the-box (`ModuleNotFoundError`). | Team DevOps / Leads |
| **DEP-CTR-003** | MHEALTH (Task 4) $\to$ FHIR | `StructureDefinition/is-synthetic` | HL7 FHIR Extension | **MISSING (IMPLEMENTATION REQUIRED)** | Synthetic patient fixtures emitted as genuine clinical observations. | MHEALTH Team |
| **DEP-CTR-004** | Shamit (TB Discovery) | `candidate_compounds.csv` Path Fix | Filesystem Anchor (`Path(__file__).parent`) | **DEFECTIVE (IMPLEMENTATION REQUIRED)** | `FileNotFoundError` on `generate_comparison.py` execution. | Shamit (TMS) |

