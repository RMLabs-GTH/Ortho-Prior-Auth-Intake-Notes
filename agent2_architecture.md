# Agent 2 Architecture (One-Page Summary)

## Purpose
Agent 2 is the deterministic evidence-gathering engine that sits between the Agent 1 intake/checklist and downstream automation. It never interprets clinical content, never writes files on its own, and exposes a pure `run_agent2` API that returns a versioned `Agent2Result` with bundles, logs, gaps, and invariants.

## Core Layers
- **Contracts and models (`agent2_document_fetch/models.py` + `contracts/agent1_checklist_v1.py`)** define the immutable schemas (`DocumentType`, `FetchRequest`, `EvidenceBundleV1`, `FetchLogEvent`, `Agent2Result`, `Agent1ChecklistV1`) along with serializers (`to_json/to_dict`). These models also track `AGENT2_VERSION = "0.1.0"` and include raw-key vs. enum mapping to keep Agent 1 and Agent 2 loosely coupled.
- **Deterministic utilities** (`hashing.py`, `normalize.py`, `planner.py`) compute stable SHA256 fingerprints, normalized filenames, and ordered fetch requests. The planner composes dedupe keys and source priorities without touching any adapter I/O.
- **Executor + adapters (`agent2_document_fetch/agent.py`, `adapters.py`, `registry.py`)** run the fetch loop: validate credentials, retry on timeouts, dedupe artifacts, populate provenance metadata (query fingerprints, adapter names), assemble `EvidenceBundleV1`, emit `FETCH_*` log events, record invariants, and respect the global `AGENT2_ALLOW_NETWORK` kill switch before any HTTP (FHIR adapter uses `requests`).
- **Smoke + validation scripts (`scripts/agent2_smoke.py`, `validate_agent1_checklist_contract.py`, `run_agent2_on_demo_case.py`)** ensure deterministic behavior, log stability, and schema contract compliance under a fixed clock.

## Integration into ortho-greenlight
- **Runner (`projects/ortho-greenlight/integrations/agent2_fetch/runner.py`, `persistence.py`, `config.py`)** maps canonical cases to `CaseContext`, merges adapters via `registry`, runs `run_agent2`, persists outputs atomically to `<storage_root>/<case_id>/case_artifacts/` (bundle, log, index, artifact metadata), and updates the case statebased on `Agent2Result`.
- **Orchestrator (`projects/ortho-greenlight/orchestrator.py`)** watches for READY cases, reads the versioned `case_artifacts/agent1_checklist_v1.json`, maps raw keys (preferred) to Agent 2 enums, logs `AGENT2_DISPATCHED/COMPLETED`, and gates downstream agents if gaps or invariants exist.
- **Agent 1 contract emitter (`projects/ortho-greenlight/agent_intake_v1/intake.py`)** now writes the `Agent1ChecklistV1` artifact with raw underscore keys (`clinical_notes`, `imaging_report`, etc.), enabling future schema bumps via `SCHEMA_VERSION` updates.

## Operational notes
- **Lock state:** Both Agent 2 and the integration are locked at v0.1.0 with documented `LOCKED.md` files; any changes to public models, log types, or storage paths must bump `AGENT2_VERSION`, refresh READMEs, and rerun the locked validation set. The summary of those validation runs is recorded for traceability.
- **Security guardrails:** The FHIR adapter honors `enabled_real_http` plus the `AGENT2_ALLOW_NETWORK=true` kill switch. No network activity occurs without intentional configuration.
- **Determinism:** All artifacts are sorted/deduped, hashing is SHA256, normalized filenames include case/doc/adapters, logs use `sort_keys=True`, and retries emit `RETRY_SCHEDULED` without sleeping.

## How to evaluate correctness
- **Determinism:** identical inputs and a fixed clock reproduce the same planner output, fetch logs, and bundle hash because adapters/logs/fingerprints are sorted and SHA256-based.
- **Auditability:** every `FetchedArtifact` records `query_fingerprint`, `dedupe_key`, `adapter_name`, `doc_type`, and `content_sha256` so auditors can trace each artifact back to its source.
- **Safety:** Agent 2 stays read-only, purposefully avoids clinical inference, and keeps outbound HTTP disabled unless both `enabled_real_http` and environment `AGENT2_ALLOW_NETWORK=true` are set.

## Integration diagram (textual)
- Agent 1 writes `case_artifacts/agent1_checklist_v1.json` with raw document keys.
- The orchestrator dispatches Agent 2 when the case is READY, using the checklist to derive requested document types and date bounds.
- The runner executes `run_agent2`, then persists `evidence_bundle_v1.json`, `fetch_log_v1.jsonl`, `agent2_index.json`, and the optional raw files under `<storage_root>/<case_id>/case_artifacts/`.
- The case transitions to `EVIDENCE_COLLECTED` when evidence is complete or stays `NEEDS_INFO` with gap metadata otherwise.

## Shareable takeaway
- Agent 2 is purely deterministic and self-contained; its integration-level runner handles persistence and case-state updates, while the orchestrator reads only the Asset 1 checklist artifact for dispatch inputs.  
- The schemas (`EvidenceBundleV1`, `FetchLogEvent`, `Agent1ChecklistV1`) and their serializers are versioned (`v1`, `AGENT2_VERSION=0.1.0`) and locked—changes mean bumping versions, refreshing artifacts, and rerunning the validation scripts documented in `LOCKED.md`.  
- Smoke scripts, demo harnesses, and validators capture the current behavior, so this architecture is ready to share with stakeholders as an immutable, auditable foundation for future enhancements.
