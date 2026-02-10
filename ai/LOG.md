# GenAI Log

If you choose not to use GenAI for a period, record "No GenAI used" with a brief note and date.

---

Date: 2026-01-20
Goal: Configure and verify SSH commit signing for GitHub
Prompt summary: Confirm SSH signing is working, commit to GitHub, update evidence index
Key output (1-2 lines): SSH signing configured (gpg.format=ssh, commit.gpgsign=true). Commit 90b36ca pushed successfully with verified SSH signature after adding signing key to GitHub.
Decision (adopted/rejected + why): Adopted - SSH signing is now working and enforced by GitHub repository rules
Evidence link (commit/issue/experiment): evidence-001, commit 90b36ca
Unknowns + next test: Run and pass the self-check script (tools/validate_repo.py)

---

Date: 2026-01-20
Goal: Deploy a trivial "Hello Cloud" function to Google Cloud Platform
Prompt summary: Create and deploy a simple HTTP Cloud Function to GCP, verify it works, add verification screenshot
Key output (1-2 lines): Deployed Python Cloud Function (gen2) to us-central1. Function responds with "Hello Cloud" at https://us-central1-project-e05df272-63bf-43a4-95d.cloudfunctions.net/hello-cloud
Decision (adopted/rejected + why): Adopted - Function successfully deployed and publicly accessible, verified via curl
Evidence link (commit/issue/experiment): evidence-002, commit c8846f3
Unknowns + next test: N/A

---

Date: 2026-01-29
Goal: Dark-pattern lab 1 — design dark and ethical ToS acceptance screens, document techniques, reflection
Prompt summary: Design dark ToS screen (manipulative) → ethical redesign → reflection; commit and update evidence per evidenceUpdate
Key output (1-2 lines): docs/ethics/dark-pattern-lab-1/ with dark.md, ethical.md, reflection.md, tos_acceptance_dark.html, tos_acceptance_ethical.html; evidence index and LOG updated
Decision (adopted/rejected + why): Adopted — lab committed and pushed; evidence entry and LOG added per evidenceUpdate workflow
Evidence link (commit/issue/experiment): evidence-003, commit b449745
Unknowns + next test: N/A

---

Date: 2026-01-29
Goal: Add CCT (Clause → Control → Test) enforcement doc for ethical ToS; commit and update evidence per evidenceUpdate
Prompt summary: Commit cct.md and use evidenceUpdate — identify 2 promises, clause/control/test/enforcement point
Key output (1-2 lines): docs/ethics/dark-pattern-lab-1/cct.md with 2 CCT entries (material terms visible, both choices equally accessible); evidence-004 and LOG updated
Decision (adopted/rejected + why): Adopted — cct.md committed; evidence index and LOG updated per evidenceUpdate
Evidence link (commit/issue/experiment): evidence-004, commit 02c9e73
Unknowns + next test: Implement red-bar tests described in cct.md (CI enforcement)

---

Date: 2026-01-29
Goal: Record capstone team repo URL; commit and update evidence per evidenceUpdate
Prompt summary: Added team repo URL to team_repo.txt; commit, push, evidence update
Key output (1-2 lines): capstone/team_repo.txt updated with https://github.com/janasheirah/CPSC436; evidence-005 and LOG updated
Decision (adopted/rejected + why): Adopted — team repo URL committed; evidence index and LOG updated
Evidence link (commit/issue/experiment): evidence-005, commit ca91127
Unknowns + next test: N/A
