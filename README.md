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
A clinical readiness and intelligence infrastructure that:
- **Intake**: ingests documents from common clinic channels (upload, email, fax) and normalizes them into a canonical case view.
- **Rules**: applies deterministic payer coverage rules to evaluate clinical readiness.
- **Patterns**: captures actual payer outcomes to identify patterns where real-world behavior diverges from published policies.
- **Risk Signaling**: provides "soft risk" scores and recommendations based on historical approval trends and common denial reasons.
- **Visibility**: surfaces all readiness states, missing items, and learned patterns via a unified dashboard.

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
### 2026-01-16 — Dynamic Patterns Layer & Risk Signaling
- Created a learning layer that tracks actual payer outcomes (approvals and denials) to identify patterns that diverge from published rules.
- Integrated these insights into the decision engine to provide real-time risk signaling and recommendations.
- Launched a patterns dashboard to visualize approval trends, common denial reasons, and document effectiveness.

### 2026-01-04 — Package hygiene & intake imports
- Simplified the intake helpers and adapters so they rely on the package layout rather than manual path tricks.
- Double-checked that the modules load cleanly with the new imports and that the core tests still pass.

### 2026-01-03 — Agent 2 deterministic fetch & integration
- Captured the deterministic Agent 2 architecture, guarantees, and safety/lock rules so its behavior is transparent.
- Built the runner, persistence layer, and orchestrator handoff that deliver stable bundles tied to each intake checklist run.
- Locked the schema, stamped the version, and added validation checks so Agent 2 feels trusted and repeatable.

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
