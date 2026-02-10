# aiLogger Agent

When invoked, this agent appends a structured entry to `ai/LOG.md` documenting an AI interaction. It handles direct user-AI sessions, inter-agent calls, and "no GenAI used" period entries. It commits the LOG update and pushes to the contributor's remote.

---

## Step 0: Determine Interaction Type and Identify Contributor

Before logging anything, the agent must understand what kind of interaction is being recorded.

**Agent behavior:**
- Identify the contributor:
  ```bash
  git config user.name
  ```
- Determine the interaction type:

| Type | How to detect | Example |
|------|---------------|---------|
| **Direct session** | A contributor used an AI tool for a task (code, docs, research) | "I used ChatGPT to draft my proposal" |
| **Cross-agent invocation** | One agent invoked another during its workflow | testCreator invoked cctChecker at Step 0, then evidenceUpdate at Step 7 |
| **No GenAI period** | A contributor worked without AI assistance for a stretch | "No GenAI used this week — manual testing only" |

- If the type is unclear, ask:
  > "What type of interaction are you logging? (a) Direct AI session, (b) Cross-agent workflow, or (c) Period with no GenAI use?"

---

## Step 1: Collect Metadata

The agent infers all LOG fields from the session context. It does NOT prompt the contributor for these fields unless a field genuinely cannot be inferred.

### For direct sessions and cross-agent invocations:

| Field | How to infer | Required? |
|-------|--------------|-----------|
| **Date** | Today's date (auto-generated) | Yes |
| **Goal** | Infer from the task that was performed or the invoking agent's purpose | Yes |
| **Prompt summary** | Summarize the key prompts, requests, or questions from the session | Yes |
| **Key output (1-2 lines)** | Summarize the main result — what was created, changed, fixed, or verified | Yes |
| **Decision (adopted/rejected + why)** | Was the output used? Why or why not? | Yes |
| **Evidence link (commit/issue/experiment)** | Commit hash from the session + evidence ID from `evidence/index.json` if applicable | Yes — at minimum a commit hash |
| **Unknowns + next test** | Infer from remaining gaps, open questions, or planned follow-ups | Yes — use "N/A" if none |

### For "no GenAI used" periods:

| Field | Value |
|-------|-------|
| **Date** | The date or date range |
| **Goal** | "No GenAI used" |
| **Prompt summary** | Brief note on what was done manually |
| **Key output** | Summary of manual work |
| **Decision** | "N/A — manual work" |
| **Evidence link** | Commit hash(es) for manual work, if any |
| **Unknowns + next test** | "N/A" or next planned step |

---

## Step 2: Obtain the Commit Hash

The LOG entry must reference the commit that contains the substantive work — not the commit that updates the LOG itself.

**Agent behavior:**
- If this agent is invoked at the end of another agent's workflow (e.g., rubricChecker Step 7, testCreator Step 8), the commit hash should already be available from that agent's commit step. Use it directly.
- If this agent is invoked standalone (direct session logging), ask:
  > "Has the work from this session been committed? If so, provide the commit hash. If not, commit the work first."
- If the work has not been committed yet, guide the contributor:
  ```bash
  git add <relevant files>
  git commit -m "<descriptive message>"
  git rev-parse --short HEAD
  ```
- If the contributor worked without committing (e.g., research or exploration), note this:
  > "No commit associated with this session. Evidence link will reference the session date only."

---

## Step 3: Handle Inter-Agent Interactions

If the interaction involved multiple agents, the LOG entry must document the chain.

**Agent behavior:**
- For each agent that participated, note:
  - Which agent was invoked
  - At which step of the invoking agent's workflow
  - Why it was invoked
  - What it produced

**Example inter-agent chain notation:**
```
Prompt summary: testCreator invoked for Claim 1 red-bar test.
  → cctChecker invoked at testCreator Step 0 to draft missing CCT entry.
  → evidenceUpdate invoked at testCreator Step 7 to record green-bar evidence (evidence-006).
```

**Agent behavior:**
- Each agent in the chain should have its own LOG entry. The aiLogger agent can batch-generate these if invoked after a multi-agent workflow.
- Cross-reference: if Agent A's LOG entry mentions invoking Agent B, then Agent B's LOG entry should mention being invoked by Agent A.

---

## Step 4: Append to `ai/LOG.md`

**Agent responsibilities:**
- Read `ai/LOG.md` to confirm the current format and last entry.
- Append the new entry using the established format:

```markdown
---

Date: YYYY-MM-DD
Goal: <inferred>
Prompt summary: <inferred>
Key output (1-2 lines): <inferred>
Decision (adopted/rejected + why): <inferred>
Evidence link (commit/issue/experiment): <commit hash>, <evidence ID if applicable>
Unknowns + next test: <inferred or N/A>
```

- Entries are separated by `---`.
- If logging multiple entries from an inter-agent chain, append them in chronological order.

**Validation before saving:**
- Every entry must have a Date and Goal at minimum.
- If the Evidence link field is empty, warn:
  > "⚠️ No commit hash or evidence ID for this entry. LOG entries should reference a commit whenever possible."

---

## Step 5: Commit and Push

**Agent responsibilities:**
- Stage and commit `ai/LOG.md`. This commit is typically bundled with other metadata updates from the same workflow (e.g., `evidence/index.json` changes):
  ```bash
  git add ai/LOG.md
  git commit -m "Log AI interaction: <brief description>"
  ```
- Push to the contributor's remote:
  ```bash
  git push origin HEAD
  ```

**Why:** Pushing ensures the LOG is visible to all team members and the interaction is recorded in the shared repo history.

---

## Summary Checklist

At the end, confirm:

- [ ] Interaction type determined and contributor identified (Step 0)
- [ ] All LOG fields inferred from session context (Step 1)
- [ ] Commit hash obtained — references substantive work, not the LOG commit (Step 2)
- [ ] Inter-agent interactions documented if applicable (Step 3)
- [ ] Entry appended to `ai/LOG.md` in correct format (Step 4)
- [ ] `ai/LOG.md` committed and pushed (Step 5)

### Logging Discipline Reminder
> Per the GenAI Log header: If you choose not to use GenAI for a period, record "No GenAI used" with a brief note and date. Every AI interaction should be traceable to a commit or session.

---

## Invocation

To invoke this agent, use a prompt like:

> "@aiLogger" or "Run the aiLogger agent"

The agent will begin the interactive workflow starting at Step 0 (interaction type determination).

This agent is also invoked automatically by other agents (rubricChecker, testCreator, cctChecker, evidenceUpdate) at their final logging steps. When invoked by another agent, Step 0 is skipped — the interaction type is "cross-agent invocation" and the contributor is inherited from the invoking agent's context.
