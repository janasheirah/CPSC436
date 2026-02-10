# Capstone Rubric Checklist (Self-/Peer-/TA Use)

Use this to self-evaluate before submission. Tie evidence back to code/docs.

## 1) System Design & Correctness (30%)
- [ ] Architecture addresses requirements and constraints
- [ ] Clear data flow and trust model diagrams
- [ ] APIs and schema are coherent and documented
- [ ] Trade-offs justified (chosen pattern vs alternatives)

## 2) Reliability & Scaling (20%)
- [ ] Load tests run (targets: reads/writes, surge window)
- [ ] Back-pressure, idempotency, and failure modes covered
- [ ] Observability (metrics/logs/traces) demonstrates behavior
- [ ] DR/restore or redeployability plan exists (IaC)

## 3) Privacy & Ethics (20%)
- [ ] PIA completed (data inventory, minimization, retention, access)
- [ ] Telemetry decision matrix included; k-anon checks where applicable
- [ ] Concrete mitigations implemented in code (not just docs)
- [ ] Disclosure (“what we collect & why”) and opt-ins where feasible

## 4) Cost & Operability (10%)
- [ ] Basic cost modeling (idle vs surge; dominant drivers)
- [ ] Runbook (incident handling, abuse mitigation) present
- [ ] Alerts/SLOs or pragmatic equivalents defined

## 5) Documentation & Demo (10%)
- [ ] Clear README with how-to-run and assumptions
- [ ] Demo script that shows success cases and limits
- [ ] Postmortem template or known-issues section

## 6) Stretch / Innovation (10%)
- [ ] Optional advanced features (e.g., DP heatmaps, offline-first)
- [ ] Rationale for why it’s valuable given constraints

> TAs: calibrate partial credit; prioritize demonstrated engineering evidence over aspirational claims.

