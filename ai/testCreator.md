# testCreator Agent

When invoked, this agent guides a contributor through creating, running, and documenting tests that verify project claims. It follows the Red Bar Showcase pattern and ties every test to a CCT entry and claim. It commits test files, updates the ethics ledger when applicable, and logs the interaction.

---

## Step 0: Validate Test Target and Identify Contributor

Before writing any test, the agent must confirm what is being tested and who is doing the work.

**Agent behavior:**
- Identify the contributor:
  ```bash
  git config user.name
  ```
- Ask the contributor:
  > "Which claim or CCT entry does this test target? (Provide a claim ID from `docs/claims.md` or a CCT row number from `docs/cct_v1.md`.)"
- Read `docs/claims.md` and `docs/cct_v1.md`.
- Confirm the claim/CCT entry exists and has an enforcement point defined.

### Validation checks

| Check | Validation question | If it fails |
|-------|---------------------|-------------|
| Claim exists | "Does the claim ID appear in `docs/claims.md`?" | Warn and ask the contributor to add the claim first |
| CCT entry exists | "Is there a CCT row for this claim in `docs/cct_v1.md`?" | Offer to invoke the **cctChecker** agent to draft one before proceeding |
| Enforcement point defined | "Does the CCT row have a non-empty enforcement point?" | Cannot write a meaningful test without knowing where the control bites — block until resolved |

**Agent behavior on CCT gap:**
- If the CCT row is missing or incomplete, prompt:
  > "⚠️ Claim [ID] has no CCT entry (or the CCT entry is incomplete). A test must target an enforcement point. Would you like to invoke the cctChecker agent to draft a CCT entry first?"
- If the contributor agrees, note the cross-agent invocation for logging in Step 7.

---

## Step 1: Read Reference Material

The agent MUST read the following before writing any test:

| Document | Path | What it provides |
|----------|------|------------------|
| Red Bar Showcase | `projectDocs/Red Bar Showcase_ CPSC_V 436C C_201 2025W2 Topics in Computer Science - CLOUD COMPUTING.pdf` | Canonical examples: clause, enforcement point, test function, red-bar evidence, next action |
| Red-bar test conventions | `tests/redbar/README.md` | Where to place red-bar tests and documentation expectations |
| Ethics Debt Ledger Template | `projectDocs/Ethics Debt Ledger Template_ CPSC_V 436C C_201 2025W2 Topics in Computer Science - CLOUD COMPUTING.pdf` | How test results tie to ledger entries (status, acceptance test, enforcement point) |
| Current ethics ledger | `docs/ethics_ledger_v1.md` | Existing entries to update if the test relates to an ethics promise |

---

## Step 2: Write the Test

Write a test function following the Red Bar Showcase pattern.

**Required elements of every test:**

1. **Clause as docstring/comment** — the specific, falsifiable promise being tested
2. **Setup** — any preconditions (data seeding, service state, mocking)
3. **Action** — the operation that exercises the enforcement point
4. **Assertion** — a pass/fail check with a descriptive failure message
5. **Red-bar note** (if the test is expected to fail) — a comment explaining why it fails now and what control would make it pass

**Example structure (Python — adapt to your project's language):**
```python
def test_storage_reduction_after_lifecycle():
    """
    Clause: For documents older than 90 days, the system will reduce
    storage footprint by >=95% by replacing full documents with AI summaries.
    Enforcement point: S3 Lifecycle policy + DynamoDB summary storage.
    """
    # Setup
    original_size = upload_test_document(bucket="ingest-raw", size_kb=500)
    
    # Action
    trigger_lifecycle_and_summarization()
    summary_size = get_dynamo_item_size(table="summaries", key="test-doc")
    s3_object_exists = check_s3_object(bucket="ingest-raw", key="test-doc")
    
    # Assert
    assert not s3_object_exists, "Original S3 object still present after lifecycle"
    reduction = 1 - (summary_size / original_size)
    assert reduction >= 0.95, f"Storage reduction {reduction:.1%} is below 95% threshold"
```

**Agent responsibilities:**
- Place the test in the appropriate directory:
  - Red-bar tests → `tests/redbar/`
  - Green-bar / passing tests → `tests/` (or a relevant subdirectory)
- Name the test file descriptively: `test_<claim_keyword>.py` (or equivalent)
- Include the clause, enforcement point, and claim ID in the test's docstring or comments

---

## Step 3: Run the Test and Capture Output

**Agent responsibilities:**
- Run the test and capture the full output (stdout + stderr).
- Determine the result:

| Result | Meaning | Next steps |
|--------|---------|------------|
| **Red bar** (test fails) | The enforcement control is not yet in place — the test fails for the right reason | Document the failure; proceed to Step 4 |
| **Green bar** (test passes) | The enforcement control is working — the claim has evidence | Proceed to Step 4; this will also trigger evidence collection in Step 6 |
| **Error** (test crashes) | The test itself has a bug, not a meaningful failure | Fix the test before proceeding |

**Agent behavior:**
- If the test errors (not a clean assertion failure), warn:
  > "⚠️ The test errored rather than failing cleanly. A red-bar test should fail at the assertion, not crash. Please review the test for bugs."

---

## Step 4: Document the Result

**If red bar:**
- Create or update a short note at `tests/redbar/test_<claim_keyword>_notes.md` explaining:
  - Why the test fails now
  - What control or implementation would make it pass (the "next action")
  - Which enforcement point it targets
- This is the "red bar evidence" from the Red Bar Showcase pattern.

**If green bar:**
- Note the passing output — this will be used as evidence in Step 6.
- No separate notes file is needed for passing tests.

---

## Step 5: Update Ethics Ledger (If Applicable)

**Agent behavior:**
- Ask the contributor:
  > "Does this test relate to an ethics promise or ethical guardrail? (yes/no)"
- **If yes:** Update `docs/ethics_ledger_v1.md` following the Ethics Debt Ledger Template format:

| Field | Value |
|-------|-------|
| **Status** | 🔴 (red bar — test failing), 🟡 (partial mitigation), or 🟢 (enforced — test passing) |
| **Risk / Promise** | The clause from the test |
| **Who it protects** | Name the community, not just "users" — tie back to proposal stakeholders |
| **Control Owner** | The contributor or team member responsible |
| **Acceptance Test** | Reference to the test file (e.g., `tests/redbar/test_storage_reduction.py`) |
| **Enforcement Point** | From the CCT entry |
| **Next Action** | What needs to happen to move from red → yellow → green |
| **Last Reviewed** | Today's date |

- **If no:** Skip to Step 6.

---

## Step 6: Commit the Test Files

**Agent responsibilities:**
- Stage and commit the test file(s), any notes files, and updated docs (e.g., `docs/ethics_ledger_v1.md` if changed). Do **not** include `ai/LOG.md` or `evidence/index.json` in this commit.
  ```bash
  git add tests/redbar/test_<name>.py tests/redbar/test_<name>_notes.md
  git add docs/ethics_ledger_v1.md  # only if changed in Step 5
  git commit -m "Add red-bar test for claim [ID]: [short description]"
  ```
- Capture the commit hash:
  ```bash
  git rev-parse --short HEAD
  ```

**Why:** The commit must exist before the LOG or evidence entry references it.

---

## Step 7: Invoke evidenceUpdate Agent (If Green Bar)

If the test passed (green bar) and constitutes evidence for a claim:

**Agent behavior:**
- Invoke the `evidenceUpdate` agent workflow (see `ai/evidenceUpdate.md`) to:
  - Validate the evidence against EVIDENCE.md criteria
  - Add an entry to `evidence/index.json` with the commit hash from Step 6
  - Collect any receipts (test output logs, screenshots)
- Note this cross-agent invocation for the LOG entry in Step 8.

**If red bar:** Skip this step. Red-bar tests are documented but do not produce evidence entries until they turn green.

---

## Step 8: Log the Interaction

Append a new entry to `ai/LOG.md` auto-inferred from the session.

| Field | How to infer |
|-------|--------------|
| **Date** | Today's date |
| **Goal** | "Create [red-bar/green-bar] test for Claim [ID]" |
| **Prompt summary** | Summarize what was tested and against which enforcement point |
| **Key output** | "Test file: [path]. Result: [red bar / green bar]. [Brief outcome]." |
| **Decision** | "Adopted — test committed" or "Adopted — red bar documented, next action: [action]" |
| **Evidence link** | Commit hash from Step 6. If green bar and evidenceUpdate was invoked, also include the evidence ID |
| **Unknowns + next test** | What needs to happen next (e.g., implement the control to turn red → green) |

**Agent behavior:**
- Do NOT prompt the contributor for LOG fields — derive them automatically.
- If the cctChecker agent was invoked in Step 0, note:
  > "Invoked cctChecker at Step 0 to draft missing CCT entry for Claim [ID]."
- If the evidenceUpdate agent was invoked in Step 7, note:
  > "Invoked evidenceUpdate at Step 7 to record green-bar evidence."
- Stage and commit `ai/LOG.md`:
  ```bash
  git add ai/LOG.md
  git commit -m "Log test creation interaction"
  ```
- Push to the contributor's remote:
  ```bash
  git push origin HEAD
  ```

---

## Summary Checklist

At the end, confirm with the contributor:

- [ ] Test target validated — claim and CCT entry confirmed (Step 0)
- [ ] Reference material read (Step 1)
- [ ] Test written following Red Bar Showcase pattern (Step 2)
- [ ] Test executed and result captured (Step 3)
- [ ] Result documented — red-bar notes or green-bar output (Step 4)
- [ ] Ethics ledger updated if applicable (Step 5)
- [ ] Test files committed; commit hash obtained (Step 6)
- [ ] evidenceUpdate agent invoked if green bar (Step 7)
- [ ] `ai/LOG.md` updated and committed with cross-agent notes (Step 8)
- [ ] Changes pushed to contributor's remote (Step 8)

### Red Bar Reminder
> Per the Red Bar Showcase: A failing test captures a promise you have not met yet. It keeps the issue visible. When the control is in place, rerun the test and log the transition from red → green in your ethics debt ledger.

---

## Invocation

To invoke this agent, use a prompt like:

> "@testCreator" or "Run the testCreator agent"

The agent will begin the interactive workflow starting at Step 0 (test target validation).
