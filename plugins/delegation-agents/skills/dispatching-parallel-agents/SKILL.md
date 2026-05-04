---
name: dispatching-parallel-agents
description: Use when the executing-plans skill (or the user directly) has a set of tasks from a planning-projects plan marked Parallel YES whose dependencies are all green. Dispatches one agent per independent task, runs them concurrently, and integrates results respecting the plan's dependency graph. Triggers on "dispatch these tasks in parallel", "run these Parallel YES tasks", "fan out these independent tasks", or when executing-plans hands off ready tasks.
---

# Dispatching Parallel Agents

Fan out independent tasks from a `planning-projects` plan to concurrent sub-agents, collect their results, propagate the dependency graph, and return control to `executing-plans` (or the caller).

This skill is the operational arm of `planning-projects` Phase 5 ("Parallel Execution") and is usually invoked by `executing-plans` rather than the user directly.

**Announce at start:** "Using the dispatching-parallel-agents skill to fan out <N> independent tasks."

---

## Preconditions

Refuse to dispatch unless all are true:

1. **A plan exists** at a known path, produced by `planning-projects`. The plan's task fields (`Depends on`, `Blocks`, `Parallel`, `Evidence-of-done`, `Confirm-Adjust max cycles`) are the source of truth.
2. **Every task in scope has `Parallel: YES`** in the plan.
3. **Every task's `Depends on` list is fully green** (every referenced prior task has met its evidence-of-done).
4. **No two tasks in scope modify the same file.** This is checked against the plan's declared file paths; if the plan doesn't list files, scan the task descriptions. When in doubt, force sequential.
5. **The stage gate has not yet run.** Dispatches happen before the gate, not after.

If any precondition fails, stop and report which one — do not relax them on your own.

---

## Checklist

Create a task for each:

1. **Select** — identify the set of tasks that satisfy all preconditions
2. **Guard** — verify file-path disjointness; force any conflicting pair sequential
3. **Brief** — construct an agent prompt per task from the plan (see prompt template)
4. **Dispatch** — launch all selected tasks in a single message (multiple Agent calls)
5. **Collect** — wait for every sub-agent to return
6. **Integrate** — accept green results, escalate failures, re-check the dependency graph
7. **Report** — hand control back to `executing-plans` with the updated task status

---

## Phase 1 — Select

From the current stage, identify tasks where:

- `Parallel: YES`
- Every task in `Depends on` is in the completed set (check the task list or plan status notes)
- The task has not already been dispatched or completed

Call this set **S**. If |S| < 2, there's no parallelism to exploit — return to the caller and execute sequentially.

## Phase 2 — Guard against file conflicts

For every pair `(tᵢ, tⱼ)` in S, compare the file paths each task will modify:

- If paths are disjoint → keep both in S
- If paths overlap → remove one from S (prefer keeping the higher-`Blocks`-count task, since it unblocks more downstream work). The removed task will be worked sequentially after dispatch returns.

Also guard against **shared resources** beyond files: same database table, same shared config block, same spreadsheet row range, same calendar slot — these are "logical" file conflicts even when the literal paths differ.

Log which task (if any) was deferred and why.

## Phase 3 — Brief each agent

For each task remaining in S, construct a self-contained prompt. A sub-agent will not see the conversation — it sees only its prompt.

### Prompt template

```
You are a sub-agent executing Task <N.M> from plan <plan-path>.

## Task
<Task description from plan, verbatim>

## Files
<Files from plan, verbatim — Create / Modify / Reference paths>

## Context (only what this task needs)
<Extract from Research Summary the 2-4 bullets that bear on this task.
Do NOT paste the entire research summary.>

## Execution model (Confirm-Adjust loop)
1. Attempt the task as described
2. Check the evidence-of-done:
   <Evidence-of-done, verbatim from plan>
   Expected: <expected pass criterion>
3. If NOT YET: diagnose what's missing, form a hypothesis, make ONE targeted adjustment, re-check
4. Max <Confirm-Adjust max cycles> NOT-YET cycles. If exceeded, STOP and report — do not keep looping

## Constraints
- Do NOT modify files outside the Files list above
- Do NOT refactor or restructure unrelated content
- Do NOT introduce new dependencies, new templates, or new conventions not already in use
- Save / commit (if applicable) with the note: "Stage <N> Task <N.M>: <description>"

## Return
A structured report:
- STATUS: GREEN | ESCALATE
- If GREEN: file paths touched, evidence-check result, any notes
- If ESCALATE: last gap, diagnosis, what was tried, what's needed from the caller
```

**Prompt discipline:** focused scope (one task), self-contained (all needed context inlined), explicit constraints (no scope creep), specific return format (so the caller can integrate).

## Phase 4 — Dispatch

Launch every selected task in a **single message** with multiple `Agent` tool calls. This is the only way they actually run concurrently — sequential tool calls wait on each other.

```
Agent(description: "Task 2.3: outline chapter 3", prompt: <per task 2.3>)
Agent(description: "Task 2.4: outline chapter 4", prompt: <per task 2.4>)
Agent(description: "Task 2.5: outline chapter 5", prompt: <per task 2.5>)
```

Match each Agent's `subagent_type` to the work:

- **`bulk-reader`** (Haiku, read-only) — when the task is dominated by reading many files, grepping, link-checking, sampling logs. Cheapest tier; use whenever the task doesn't need to write.
- **`document-rewriter`** (Sonnet, Read+Edit only) — when the task is editing existing documents to a spec.
- **`draft-generator`** (Sonnet, Read+Write+Edit) — when the task is generating new documents from a spec.
- **`general-purpose`** — when no specialist fits or the task spans multiple kinds of work.

Pick the cheapest tier that can do the job. A scan-and-report task on Opus wastes ten times the tokens of the same task on Haiku.

## Phase 5 — Collect

Wait for every dispatched agent to return. A single outstanding agent blocks propagation — don't start integrating until all are in.

Each agent returns `STATUS: GREEN` or `STATUS: ESCALATE`. Trust-but-verify:

- For `GREEN`: confirm the files exist, re-check the evidence-of-done in the main session, spot-check the diff or the new content.
- For `ESCALATE`: read the reported diagnosis; that's evidence, not a fix — don't act on it blindly.

## Phase 6 — Integrate

For each GREEN task:

1. Mark the task completed in the plan's status notes and in your task list.
2. Re-check the task's evidence-of-done **in the main session** (not just via the sub-agent's report) — agents occasionally claim green on a check that was skipped or misreported.
3. Check the task's `Blocks` list: for each blocked task, check whether its `Depends on` is now fully green. If yes, that task becomes dispatchable in the next round.

For each ESCALATE task:

1. Do not dispatch its dependents (the graph is blocked through this node).
2. Surface the escalation to the user with: task ID, last gap, agent's diagnosis, your read on it, and what option you're recommending (re-dispatch with tighter scope, revise the plan, execute manually).
3. Wait for user direction.

### Merge check

After integrating, re-check the stage as a whole — not just the individual evidence-of-done items. Parallel work sometimes interacts at seams the individual checks don't cover (a chapter outline that double-counts a beat that another chapter also claims; a budget line that two parallel asks both spent). Catch it here before the stage gate.

## Phase 7 — Report and hand back

Return to `executing-plans` (or the calling session) with:

- Count dispatched, count green, count escalated
- File paths or outputs for green tasks
- Outstanding escalations
- Newly-unblocked tasks ready for the next dispatch round

`executing-plans` decides whether to call this skill again for the next round or move on to the stage gate.

---

## Examples (non-coding)

### Example A — Multi-chapter book outline

Stage 2 of a book project plans parallel outlines for Chapters 2, 3, 4, 5 (each independent — Chapter 1 and Chapter 6 wait on the middle four). Files: separate `Pieces/chapter-N-outline.md`. Dispatch four `draft-generator` agents in one message, each briefed on its chapter's beat sheet. After all return green, re-check that no two chapters claim the same beat (merge check) before the Stage 2 gate.

### Example B — Source-link audit across a folder

Stage gate of a research project requires that every URL in the `Sources/` folder still resolves. Dispatch one `bulk-reader` agent with the full list of files and a 200-URL probe budget. (Single agent, not a fan-out — but the same skill applies the read-only / no-mutation discipline.) On return, the caller files broken links into a `Findings/Stale Links.md` page.

### Example C — Conference panel asks

Stage 3 of a conference plan needs four panel-ask emails drafted in parallel — each to a different panelist, pulled from a different `People/<name>.md` page. Dispatch four `draft-generator` agents in one message, each briefed on the panelist's profile + the panel topic + the ask template. The drafts go to a `Pieces/panel-asks/` folder for the user to review and send manually — the agent never sends.

### Example D — Apply editor's marks across chapters

Editor returned three chapters with marked-up changes. Each chapter is a separate file; the changes don't overlap between chapters. Dispatch three `document-rewriter` agents in one message, each briefed on its chapter file and its specific edit list. On return, re-check that the running word count still matches the manuscript target.

---

## Common mistakes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Dispatching tasks that share a file | Conflicts or silent overwrites in parallel | Run the Phase 2 file-disjointness check; force sequential on conflict |
| Inlining the entire research summary into every agent prompt | Blown context, slow dispatch, agents get lost | Give each agent only the 2-4 bullets relevant to its task |
| Trusting the agent's "GREEN" without verifying | Integration-time surprises; green checks that were actually skipped | Re-run the evidence-of-done in the main session for every green return |
| Dispatching before all Depends-on are green | Agents block waiting for prerequisites, or produce broken output | Hard-check: every task in scope has every Depends-on in the completed set |
| Dispatching sequentially (one Agent call per message) | No actual parallelism; same runtime as running them one after another | Single message, multiple Agent tool calls |
| Picking the wrong tier | Bulk-read task on Sonnet/Opus; trivial edit on Opus; cost balloons | Pick the cheapest tier that can do the job: bulk-reader (Haiku) for read-only, document-rewriter (Sonnet) for edits, draft-generator (Sonnet) for new files |
| Relaxing preconditions "just this once" | The shape of the whole execution model breaks; future tasks depend on assumptions that no longer hold | Stop and ask — or revise the plan through `planning-projects` |
| Ignoring escalations to keep the pipeline moving | Downstream dispatch on a broken graph; wasted agent time | Escalation freezes the subtree; surface it and wait |

## When NOT to use this skill

- Tasks are related (one fix might fix others) — investigate as a group first.
- You don't have a plan — brainstorm and plan before dispatching.
- |S| = 1 — no parallelism; just execute.
- Tasks touch shared state (same file, same shared config, same spreadsheet) — force sequential.
- The stage gate is ready to run — gates are a synchronization point; don't dispatch past them.

---

## Sources and rationale

- **Dependency graph execution** — classic topological sort from graph theory; *Introduction to Algorithms* (CLRS) §22.4
- **Fan-out / fan-in pattern** — Communicating Sequential Processes (Hoare, 1978)
- **Self-contained agent prompts** — Anthropic multi-agent orchestration guidance; sub-agents have no conversation context and must be briefed completely
- **Trust-but-verify** — Reagan / Gorbachev; applied to sub-agent reports in *Google SRE Book* Ch. 9 on incident postmortems
- **Cost-tier model selection** — Anthropic *Choosing a Model* guide; Haiku for high-volume / latency-sensitive, Sonnet for balanced transformation work, Opus for complex reasoning. Pick the cheapest tier that can do the job.
- **Merge check after parallel work** — continuous integration practice; Fowler, "Continuous Integration" (2006)
- **Failure propagation through dependents** — Erlang "let it crash" + supervisor trees; don't build on broken foundations

## Integration

- **planning-projects** — the upstream skill producing the plan with Parallel and dependency fields this skill consumes.
- **executing-plans** — the usual caller; decides when to invoke this skill during stage execution.
- **bulk-reader** — Haiku-tier dispatch target for read-only tasks.
- **document-rewriter** — Sonnet-tier dispatch target for edits to existing documents.
- **draft-generator** — Sonnet-tier dispatch target for new-document generation.
