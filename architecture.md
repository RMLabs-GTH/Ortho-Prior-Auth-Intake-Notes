# Agent 2: Document Fetch

## Purpose
Fetch required clinical artifacts deterministically using validated adapters and return an auditable bundle.

## Non-goals
- No interpretation of medical necessity.
- No payer submission.
- No credential management beyond scoped adapter config.

## Inputs
- `FetchRequest` (requested document types, date bounds, patient identifiers, adapter config)
- Case context (case_id, clinic_id)

## Outputs
- `EvidenceBundleV1` when artifacts exist
- `FetchLogEvent` stream (FETCH_* provenance events)
- `Agent2Result` with normalized filenames and dedupe semantics

## Determinism and side effects
- Deterministic planning, ordering, normalization, and bundle assembly for identical inputs.
- Side effects: external reads via adapters. No external writes.

## Interfaces and contracts
- Version constant: `AGENT2_VERSION = "0.1.0"`
- Contracts: `DocumentType`, `FetchRequest`, `EvidenceBundleV1`, `FetchLogEvent`, `Agent2Result`, `Agent1ChecklistV1` reference

## Core flow
1. Validate adapters and request.
2. Plan and normalize fetch operations.
3. Execute with retry rules for timeouts.
4. Normalize filenames, dedupe, and persist artifacts.
5. Emit logs and bundle only if artifacts exist.

## Failure modes and retries
- Adapter validation failure: explicit error result.
- Timeouts: retry with capped attempts; log all attempts.
- Empty results: no bundle emitted, explicit empty result.

## Test strategy
- Adapter contract tests
- Deterministic plan hashing tests
- Golden log/event sequences

## Integration points
- Upstream: Agent 4 planning or orchestrator requests
- Downstream: Agent 4 evidence assembly consumes `EvidenceBundleV1`
