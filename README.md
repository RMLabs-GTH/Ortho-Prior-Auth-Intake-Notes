# Ortho Prior Auth Intake Notes

## Disclaimer
This repository documents an internal design and build progression.
It is not a deployed product, not connected to any live clinic systems, and not a claim of outcomes.
All examples are synthetic and contain no PHI.

---

## The problem
Orthopaedic clinics lose time to prior authorization back-and-forth before anyone can answer a basic question:
Is the case ready to submit, or is required documentation missing?

Delays here slow imaging and procedures and create avoidable friction for surgeons, staff, and patients.

---

## The core idea
Before payer rules, portals, or EHR integration, the first job is **intake + readiness**:
- accept the documents clinics already use (upload, email, fax)
- normalize them into a single case view
- apply a simple completeness checklist
- clearly route the case to **READY** or **NEEDS INFO**

This “readiness layer” is intentionally narrow. It aims to remove ambiguity early.

---

## What exists today (design scope)
A foundational intake layer that:
- ingests documents from common clinic channels (upload, email, fax)
- normalizes into a canonical case view
- evaluates a configurable checklist
- outputs a clinic-readable summary of:
  - readiness state (READY / NEEDS INFO)
  - missing vs received items
  - recommended next action

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
- Not submitting to portals/fax/APIs
- Not monitoring payer responses
- Not a pilot or deployment
- Not a guarantee of time savings or approval rates

---

## Why share this
This repo is a short, public snapshot of how I’m approaching the intake/readiness bottleneck in orthopaedics.

---

Last updated: 2025-12-31
