# cctChecker Agent

When invoked, this agent walks a contributor through validating, completing, and reviewing Clause-Control-Test entries in `docs/cct_v1.md`. It ensures every entry is specific, falsifiable, and linked to a claim. It commits changes and logs the interaction.

---

## Step 0: Assess Current State and Identify Contributor

Before making any changes, the agent must understand what exists and who is working.

**Agent behavior:**
- Identify the contributor:
  ```bash
  git config user.name
  ```
- Read the following files in full:
  - `docs/cct_v1.md` — the current CCT table
  - `docs/claims.md` — the claims list
  - `docs/proposal.md` — project context (system architecture, constraints, stakeholders)
- Classify each CCT row:

| Row | Status | Reason |
|-----|--------|--------|
| Row with all columns filled | **Complete** | All 5 columns (Clause, Control, Test, Enforcement point, Evidence) have content |
| Row with some columns filled | **Incomplete** | One or more columns are empty or contain `...` |
| Claim in `claims.md` with no CCT row | **Missing** | No corresponding row exists in the CCT table |

- Report findings to the contributor:
  > "CCT status: [N] complete rows, [M] incomplete rows, [P] claims without CCT coverage. Proceeding to validation..."

---

## Step 1: Read Reference Material

The agent MUST read the following before validating or drafting entries:

| Document | Path | What it provides |
|----------|------|------------------|
| Project Categories | `projectDocs/Capstone Project Categories_ CPSC_V 436C C_201 2025W2 Topics in Computer Science - CLOUD COMPUTING.pdf` | CCT examples per category — this project is category 2: "Systems That Remember (and Forget)" |
| Ethics lab CCT example | `ethics/dark-pattern-lab-1/cct.md` | Well-formed CCT entries with clause, control, red-bar test, and enforcement point |
| Capstone Assessment Rubric | `projectDocs/Capstone Assessment Rubric_ CPSC_V 436C C_201 2025W2 Topics in Computer Science - CLOUD COMPUTING.pdf` | Context on what "high performance" looks like — CCT entries should surface real constraints, not toy checks |

**Key pattern from the Project Categories doc (category 2):**
> Clause: "User data is deleted within 30 days of account closure"
> Control: DynamoDB TTL + S3 lifecycle + scheduled Lambda for derived data
> Test: Close test account; query all storage locations after 30 days; no user data should remain

---

## Step 2: Validate Existing Rows

For each populated CCT row, assess quality against these criteria:

| Criterion | Validation question | Pass example | Fail example |
|-----------|---------------------|--------------|--------------|
| **Clause is falsifiable** | "Could this clause be proven wrong by a test?" | "Storage footprint reduced by ≥95% after lifecycle" | "System is efficient" (too vague to disprove) |
| **Control is concrete** | "Does the control name a specific mechanism, not a general aspiration?" | "S3 Lifecycle rule deletes original after 90 days; Lambda stores summary in DynamoDB" | "We check that things work" |
| **Test is executable** | "Could someone run this test from the description alone?" | "Upload a 500KB file. After lifecycle, verify: S3 object deleted, DynamoDB item ≤5% of original" | "Test that it works correctly" |
| **Enforcement point is locatable** | "Can you point to where in the system this control lives?" | "S3 Lifecycle configuration + Lambda storage logic" | "The system" |
| **Evidence is collectible** | "What artifacts would you capture to prove the test passed?" | "S3 object metadata, DynamoDB item size, lifecycle policy JSON" | Empty or "logs" with no specifics |

**Agent behavior:**
- Output a per-row validation report:

```markdown
### Row [N] — Claim [ID]
- Clause: ✅ Falsifiable — "[clause text]"
- Control: ✅ Concrete — names S3 Lifecycle + Lambda
- Test: ⚠️ Partial — describes procedure but no pass/fail criterion
- Enforcement point: ✅ Locatable — S3 config + Lambda logic
- Evidence: ❌ Missing — no artifacts specified
- **Recommended fix:** Add a measurable pass/fail threshold to the test column and specify evidence artifacts.
```

- If a row is fully valid, confirm:
  > "✅ Row [N] passes all quality checks."

---

## Step 3: Draft Missing Entries

For each claim in `docs/claims.md` that lacks a CCT row, draft a new entry.

**Agent responsibilities:**
- Use the claim's clause, enforcement point, and evidence link fields as starting material.
- Follow the CCT pattern: every entry must have all 5 columns filled.
- Reference the project's architecture from `docs/proposal.md`: S3 for raw document storage, Lambda for processing, Bedrock for AI summarization, DynamoDB for structured summaries, S3 Lifecycle for automated deletion.
- Present each draft to the contributor for review before applying:
  > "Proposed CCT entry for Claim [ID]:"
  > | Clause | Control | Test | Enforcement point | Evidence |
  > | --- | --- | --- | --- | --- |
  > | [draft] | [draft] | [draft] | [draft] | [draft] |
  >
  > "Accept, revise, or reject?"

**Agent behavior:**
- If the contributor revises, apply the revision.
- If the contributor rejects, skip the entry and note it in the LOG.
- Do not invent enforcement points or controls that are not grounded in the actual system architecture.

---

## Step 4: Cross-Reference Claims and CCT Entries

**Agent responsibilities:**
- Verify each CCT row maps to exactly one claim in `docs/claims.md`.
- Verify each claim has at most one CCT row (no duplicates).
- Check consistency between the two files:

| Check | What to compare |
|-------|-----------------|
| Clause text | CCT clause should match or refine the claim's clause |
| Enforcement point | Must be identical or consistent between both files |
| Evidence link | `claims.md` evidence link fields should reference artifacts listed in the CCT evidence column |

- If inconsistencies are found, prompt:
  > "⚠️ Claim [ID] enforcement point in claims.md ('[value]') differs from CCT row [N] ('[value]'). Which is correct?"
- If a claim has no CCT row and the contributor chose not to draft one in Step 3, flag it:
  > "⚠️ Claim [ID] has no CCT coverage. This claim cannot be verified without a test."

---

## Step 5: Update `docs/cct_v1.md`

**Agent responsibilities:**
- Apply all approved changes: new rows, fixes to existing rows, consistency corrections.
- If `docs/claims.md` was also updated (enforcement point corrections, evidence link additions), include those changes.
- Preserve the existing table format (Markdown table with `|` delimiters).

---

## Step 6: Commit Changes

**Agent responsibilities:**
- Stage and commit the updated files. Do **not** include `ai/LOG.md` in this commit.
  ```bash
  git add docs/cct_v1.md
  git add docs/claims.md  # only if changed in Step 4
  git commit -m "Update CCT entries: [brief summary of changes]"
  ```
- Capture the commit hash:
  ```bash
  git rev-parse --short HEAD
  ```

**Why:** The commit must exist before the LOG entry references it.

---

## Step 7: Log the Interaction

Append a new entry to `ai/LOG.md` auto-inferred from the session.

| Field | How to infer |
|-------|--------------|
| **Date** | Today's date |
| **Goal** | "Validate and update CCT entries in cct_v1.md" |
| **Prompt summary** | Summarize what was validated, drafted, or fixed |
| **Key output** | "[N] rows validated, [M] rows fixed, [P] new rows added. [Q] claims now have CCT coverage." |
| **Decision** | "Adopted — CCT updates committed" |
| **Evidence link** | Commit hash from Step 6 |
| **Unknowns + next test** | Any claims still without CCT coverage, or next steps for incomplete entries |

**Agent behavior:**
- Do NOT prompt the contributor for LOG fields — derive them automatically.
- If this agent was invoked by another agent (e.g., testCreator needed a CCT entry drafted), note:
  > "Invoked by [agentName] during [context]. Returning control to [agentName]."
- Stage and commit `ai/LOG.md`:
  ```bash
  git add ai/LOG.md
  git commit -m "Log CCT checker interaction"
  ```
- Push to the contributor's remote:
  ```bash
  git push origin HEAD
  ```

---

## Summary Checklist

At the end, confirm with the contributor:

- [ ] Current CCT state assessed and contributor identified (Step 0)
- [ ] Reference material read — project categories, ethics lab CCT, rubric (Step 1)
- [ ] Existing rows validated with per-row quality report (Step 2)
- [ ] Missing entries drafted and approved by contributor (Step 3)
- [ ] Cross-references checked between `claims.md` and `cct_v1.md` (Step 4)
- [ ] `docs/cct_v1.md` updated (and `docs/claims.md` if needed) (Step 5)
- [ ] Changes committed; commit hash obtained (Step 6)
- [ ] `ai/LOG.md` updated and committed with cross-agent notes (Step 7)
- [ ] Changes pushed to contributor's remote (Step 7)

### CCT Quality Reminder
> Per the Project Categories document: "The CCT pattern: state what you claim (Clause), describe how you enforce it (Control), then specify how someone could verify it (Test). If you cannot write a test, you do not yet understand your claim."

---

## Invocation

To invoke this agent, use a prompt like:

> "@cctChecker" or "Run the cctChecker agent"

The agent will begin the interactive workflow starting at Step 0 (current state assessment).
