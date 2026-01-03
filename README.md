# Ortho Prior Auth Intake Notes

## Disclaimer
This repository documents an internal design and build progression.
It is not a deployed product, not connected to any live clinic systems, and does not represent outcomes or adoption.
All examples are synthetic and contain no PHI.

---

## The problem
In orthopaedic clinics, delays often occur before prior authorization even begins.
Staff and surgeons spend time going back and forth to answer a basic question:

**Is this case ready to submit, or is required documentation missing?**

When that question is unclear, imaging and procedures are delayed unnecessarily.

---

## The core idea
Before payer rules, portals, or EHR integration, the first job is **intake and readiness**.

The goal is to:
- accept the documents clinics already use (upload, email, fax)
- normalize them into a single case view
- apply a simple completeness checklist
- clearly route the case to **READY** or **NEEDS INFO**

This layer is intentionally narrow. It focuses on early clarity, not automation.

---

## What exists today (design scope)
A foundational intake layer that:
- ingests documents from common clinic channels (upload, email, fax)
- normalizes inputs into a canonical case view
- evaluates a configurable checklist for completeness
- outputs a clinic-readable summary showing:
  - readiness state (READY / NEEDS INFO)
  - missing vs received items
  - recommended next action

All behavior is demo-only and uses synthetic documents.

---

## Example output (synthetic)
Prior Authorization Intake Result  
- Procedure: CPT 27447  
- Status: READY  
- Missing Items: None  
- Documents Received: Imaging, Operative report  
- Next Action: Ready to submit prior authorization (demo mode)

---

## What this is not
- Not EHR-integrated
- Not payer-rule complete
- Not submitting to portals, fax, or APIs
- Not monitoring payer responses
- Not a pilot or deployment
- Not a claim of time savings or approval improvements

---

## Progress Log
### 2026-01-03 — Agent 2 deterministic fetch & integration
- Documented the full Agent 2 architecture, deterministic guarantees, and safety/lock rules in `agent2_architecture.md`.
- Implemented the Agent 2 runner, persistence, and orchestrator hook that reads `agent1_checklist_v1.json`, logs dispatch/completion, and persists bundles/logs/index under `<storage_root>/<case_id>/case_artifacts/`.
- Added schema locking (`LOCKED.md`), version stamping (`AGENT2_VERSION=0.1.0`), and validation scripts to prove deterministic bundles, invariant adherence, and Agent 1 contract compliance before treating Agent 2 as immutable.

### 2026-01-01 — Coverage Rules Decision Layer (Agent 3)
- Defined a standalone, deterministic decision layer for payer + CPT coverage rules.
- Implemented versioned, schema-validated rules catalogs with explicit support semantics.
- Added ordered and deduplicated required-document checklists with audit-ready provenance.
- Normalized payer aliases and CPT inputs with explicit INVALID/MISSING outcomes.
- Included response timelines, appeal precedence, conditional rule scaffolding, and full CI tests.
- Explicitly out of scope: EHR integration, document fetching, OCR/NLP, submission logic, UI.

### 2025-12-31 — Foundational Intake & Readiness Layer (Agent 1)
- Defined a clear intake boundary focused on readiness, not automation.
- Implemented deterministic routing to READY vs NEEDS INFO using a configurable checklist.
- Added clinic-readable summary output for demo use.
- Inputs supported: upload, email, fax (synthetic documents only).
- Explicitly out of scope: EHR integration, payer rules, submission, monitoring.

---

## Why share this
This repository is a public snapshot of how I’m approaching the intake/readiness bottleneck in orthopaedics, before touching EHRs, payer rules, or submission workflows.
