---
name: executing-plans
description: Use when you have a plan file produced by the planning-projects skill (format with Stages, Tasks, Depends on / Blocks / Parallel fields, evidence-of-done, Confirm-Adjust max cycles, and Stage gates) and need to execute it. Drives Confirm-Adjust loops, respects the stage-gate model, and dispatches independent tasks in parallel when the plan allows. Triggers on "execute this plan", "run the plan", "implement docs/plans/...", "pick up this plan", "start working on this plan".
---

# Executing Plans

Execute a plan produced by `planning-projects`. Honor the stage-gate model: tasks run through Confirm-Adjust loops, a stage's gate must pass before the next stage starts, and independent tasks are dispatched in parallel when the plan's dependency graph allows it.

**Announce at start:** "Using the executing-plans skill to work through `<plan-path>`."

## What this skill expects

The plan file was produced by `planning-projects`. It contains:

- A **Research Summary** (background, not executed)
- A **Preflight** checklist (verified before Stage 1)
- One or more **Stages**, each with:
  - Goal, Depends on, Blocks, Risk, Rollback
  - Ordered **Tasks**, each with `Depends on`, `Blocks`, `Parallel: YES|NO`, `Evidence-of-done:` (a concrete way to know it's finished), `Confirm-Adjust max cycles: N`
  - A **Stage gate** checklist

If the plan doesn't have these fields, stop — it wasn't produced by `planning-projects` and must be either rewritten through that skill or executed manually.

---

## Checklist

Create a task for each, work them in order:

1. **Load and critique the plan** — raise concerns before starting
2. **Run Preflight** — verify every check; stop on failure
3. **For each stage, in order:**
   a. Dispatch `Parallel: YES` tasks via parallel sub-agents; work `Parallel: NO` tasks in the main session
   b. Drive each task through its Confirm-Adjust loop
   c. Run the stage gate; stop if it fails
4. **After all stages green:** hand off for review and finalize (see Phase Close-out)

---

## Phase 1 — Load and critique

1. Read the plan file in full
2. Verify the structure: Research Summary, Preflight, Stages with the expected fields
3. Critique: is any task's evidence-of-done vague ("should be ready")? Is any stage oversized (>7 tasks)? Is any dependency cycle present? Does any task modify a file that a parallel sibling also modifies?
4. **If concerns exist, surface them to the user before starting.** A plan with an unrunnable check or a dependency cycle will waste an entire Confirm-Adjust budget before the problem is found

Create a task list mirroring the plan: one task per stage, sub-items per task. Mark the current stage as `in_progress` only when Preflight passes.

## Phase 2 — Preflight

Run every check in the Preflight section and report pass/fail:

- Tools / resources installed and at compatible versions
- Sources, files, and references reachable
- Access / permissions verified
- Collaborators or stakeholders reachable on the plan's timeline
- Baseline prior work in a known-good state

**If Preflight fails, stop.** Report which check failed and how it failed. Do not proceed to Stage 1. A broken baseline makes every downstream Confirm-Adjust loop noise.

## Phase 3 — Stage execution

For each stage in order:

### Step 3.1 — Identify what can run now

Scan the stage's tasks. A task is **dispatchable** when every task in its `Depends on` list is green. At stage start, this is every task whose `Depends on` is either empty or lists only tasks from already-green prior stages.

### Step 3.2 — Split by parallelism

- Tasks with `Parallel: YES` and no file conflicts with another ready task → dispatch as parallel sub-agents
- Tasks with `Parallel: NO` or that modify files another parallel task modifies → work sequentially in the main session

**File-conflict check:** before dispatching, verify no two parallel tasks edit the same file. If they do, force one of them sequential even if the graph says independent.

### Step 3.3 — Confirm-Adjust loop (per task)

Every task follows this loop. No task is "done" until its evidence-of-done is in.

```
 Attempt → Confirm → Pass? ──yes──► Next task
            │
            no
            ↓
         Diagnose → Adjust → Re-confirm
            (max `Confirm-Adjust max cycles` per task)
```

**Loop rules:**

1. **One adjustment per cycle.** Don't shotgun. Isolate, fix that one thing, re-confirm.
2. **Diagnose before adjusting.** Read what's actually missing. Form a hypothesis. Confirm against the work. Then make the change.
3. **Respect the cycle budget.** The plan sets a max (default 3). When exceeded, stop and escalate — don't keep looping. Three failed targeted adjustments means the approach is wrong, not just the execution.
4. **Never skip the confirm.** The task's Evidence-of-done is the gate. "It looks right" is not green.
5. **Save / commit after each green task** with a note referencing the stage and task (`"Stage 2 Task 2.3: keynote speaker confirmed"`).

### Step 3.4 — Propagate unblock

When a task finishes green, scan its `Blocks` field. For each blocked task, check whether ALL of its `Depends on` items are now green. If yes, it becomes dispatchable — return to Step 3.1.

### Step 3.5 — Stage gate

When every task in the stage is green, run the stage gate:

- Each gate check has a specific pass criterion (a confirmation, a sign-off, a verification)
- Run them in order; stop at the first failure
- Re-check that prior stages' work is still in good shape (regressions check)

**If the gate fails:**

1. Identify which task interaction caused it (gate failures are usually integration problems, not single-task problems)
2. Add a new evidence-of-done item covering that interaction to the relevant task
3. Run that task through its Confirm-Adjust loop again
4. Re-run the gate

**If the gate passes:** mark the stage complete, save / commit with `"Stage N green"`, and start Step 3.1 for the next stage.

---

## Stop conditions

Stop immediately and escalate to the user when:

- Preflight fails
- A task exhausts its Confirm-Adjust cycle budget
- A stage gate fails and re-running the culprit task doesn't fix it after one additional cycle
- The plan contains an instruction you don't understand
- An evidence-of-done check cannot be performed (missing source, unreachable collaborator, unclear criterion)
- Producing the evidence requires modifying shared infrastructure (production system, public artifact, signed contract) — see Safety rails below

**Never guess through a stop condition.** Ask.

## When to revisit earlier steps

Return to Phase 1 (critique) when:

- The user updates the plan after feedback — treat the new version as a fresh plan and re-critique
- A stage gate failure reveals a fundamental gap in the plan (e.g., missing task, wrong dependency) — stop execution, return to `planning-projects` to revise

## Phase Close-out — After the last stage

When every stage is green:

1. Re-check the **full** project from a clean state (don't trust the per-stage runs)
2. Run any integration / e2e checks the plan flagged
3. Update the plan document with a closing note: "Completed YYYY-MM-DD. Outputs: <list>."
4. Report to the user with:
   - Stages completed
   - Total saves / commits / outputs produced
   - Plan location for future reference
   - Any deferred items the user explicitly deprioritized during execution
5. Offer finalize options (publish, send, archive, branch merge). Do not finalize without explicit confirmation.

---

## Safety rails

- **Never start on `main` / `master` (or the live, public version of a deliverable) without explicit user consent.** Use a feature branch, draft, or working copy.
- **Destructive or public-facing actions** (sending, publishing, signing, deploying, deleting) — confirm before running, even if the plan says to.
- **Secrets / credentials** — if a task would read or write credentials, stop and confirm the mechanism (env var, secrets manager, password manager) with the user before proceeding.
- **Shared resources** — staging/prod-adjacent or audience-facing changes get confirmation per stage, not per plan.

## Remember

- Critique the plan before starting
- Preflight is a hard gate
- Follow the plan's exact evidence-of-done checks
- Respect the cycle budget — three targeted adjustments, then stop
- Stage gates check integration, not just aggregate task success
- Never silently skip a Confirm-Adjust cycle — report and move on is fine; skip is not
- Save each green task; never bundle silently during execution

---

## Sources and rationale

- **Confirm-Adjust loop** — Kent Beck, *Test-Driven Development: By Example* (2002); the "test first, then make it pass" cycle, generalized from "test" to "evidence-of-done" so it applies outside coding
- **Stage gates** — Robert Cooper, *Winning at New Products* (1986); phase gates with specific pass/fail criteria
- **Max 3 failure cycles** — heuristic from debugging literature; after three targeted fixes without resolution, the hypothesis (not the execution) is wrong. See Feynman on "the first principle is that you must not fool yourself"
- **Preflight as hard gate** — aviation checklist tradition; Atul Gawande, *The Checklist Manifesto* (2009)
- **Save per green task** — frequent, small saves; *The Pragmatic Programmer* Ch. 7
- **Never skip the confirm** — Beck (TDD), Fowler ("Continuous Integration"); the evidence-of-done is the only signal that says "done"

## Integration

- **planning-projects** — produces the plan this skill consumes
- **dispatching-parallel-agents** (when available) — invoked for `Parallel: YES` tasks with no file conflicts
- **evidence-first-investigation** — invoke when a stage gate fails and the cause is not obvious; this skill builds the case for a root cause before more adjustments
