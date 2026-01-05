# Agent 2: Document Fetch Architecture

## Overview
Agent 2 is the **deterministic evidence-gathering engine** for the Ortho Greenlight workflow. It serves as a reliable bridge between initial patient intake and downstream automation by fetching required clinical artifacts (notes, reports, etc.) from various sources.

## Core Principles
The system is built on three foundational pillars:
1. **Determinism**: Identical inputs always produce the same retrieval plan and bundle of evidence. 
2. **Auditability**: Every fetched document is fingerprinted and logged with full provenance data, allowing auditors to trace any artifact back to its specific source.
3. **Safety**: Agent 2 is strictly **read-only**. It never interprets clinical content (no medical necessity interpretation) and never modifies source data.

## Process Flow
The "Document Fetch" lifecycle follows a simple, repeatable path:
1. **Intake Processing**: Agent 1 creates a checklist of required evidence.
2. **Orchestration**: When a case is ready, the system dispatches Agent 2 with specific document types and date ranges to fetch.
3. **Fetch Execution**: Validated adapters (e.g., FHIR, local file systems) retrieve the artifacts. 
4. **Normalization**: Filenames are standardized, duplicates are removed, and files are hashed (SHA256).
5. **Bundle Assembly**: A versioned Evidence Bundle and a detailed audit log are generated.

## Technical Guardrails
- **Immutable Contracts**: All interfaces are versioned (current: v0.1.0). Any changes to data structures require a version bump and regression testing.
- **Network Kill Switch**: Outbound network activity is disabled by default and requires explicit configuration and environment-level permission.
- **Pure Functional Core**: The internal planning logic is decoupled from actual I/O, making it highly testable and predictable.

## Business Value
By automating the collection of clinical evidence into a standardized, auditable format, Agent 2 significantly reduces the manual effort required for prior-authorizations while ensuring 100% compliance with data integrity policies.
