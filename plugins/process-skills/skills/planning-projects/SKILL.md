---
name: planning-projects
description: Use when the user wants a staged plan for a non-trivial project — a research investigation, a campaign, a multi-deliverable engagement, a course, a complex piece of writing, anything that needs phase gates before execution. Triggers on "plan", "roadmap", "how should I build", "break this down", "what are the steps", "create a plan", "what order should I do this", "help me structure this project".
---

# Project Planner

Create detailed, staged project plans grounded in real research. Every task gets evidence-of-done. Every piece of evidence-of-done gets a Confirm-Adjust loop. Nothing moves forward until the current task's evidence is in.

This skill produces plans where every claim traces back to researched sources, every task has a concrete way to know it's finished, and the execution model prevents half-built states through stage gates and rollback notes.

---

## Phase -1 — Clarification

Before researching or planning anything, make sure you understand what the user actually wants. Ambiguous or underspecified prompts produce plans that solve the wrong problem — and a wrong plan executed perfectly is worse than no plan at all.

### When to ask

Ask clarifying questions if any of these are unclear:

- **Scope**: What's included and what's explicitly out of scope?
- **Target context**: What audience, venue, organization, or platform is the output for?
- **Constraints**: Budget, deadlines, length limits, compliance requirements, stakeholder approvals?
- **Existing state**: Is this from scratch or does it build on prior work? If prior, where is it?
- **Success criteria**: How will the user know the project is done? What does "good" look like?
- **Audience**: Who reads, attends, or uses the end result?

### How to ask

- One question at a time. Don't dump a wall of questions
- Prefer multiple-choice when the options are finite ("Are you targeting A, B, or C?")
- If you can infer an answer from context or prior documents, state your assumption and ask for confirmation rather than asking open-ended
- Stop asking once you have enough to produce a meaningful plan. You don't need perfect information — you need enough to avoid building the wrong thing

### When NOT to ask

If the prompt is specific enough to plan against (names a topic, audience, deliverable, scope), skip straight to Phase 0. Don't ask questions for the sake of being thorough — ask because the answer would change the plan.

---

## Phase 0 — Research

Before writing a single task, gather the facts. Plans built on assumptions fall apart mid-execution when the source you needed isn't accessible, the venue's submission rules changed last month, or the audience you imagined doesn't actually exist.

### Online sources

Use WebSearch / WebFetch to pull primary sources for every relevant element:

- Submission requirements, style guides, format constraints (for writing or grant work)
- Venue / publication / conference rules and deadlines
- Up-to-date statistics, prior coverage, comparable projects
- Known issues — has this approach been tried, what happened, what was learned?

If the project depends on specific tools, frameworks, or platforms, use context7 MCP to fetch current documentation rather than relying on training data that may be months out of date.

### Local notes

If an Obsidian vault is linked (check `vault-context:status`), search it for:

- Prior decisions on this topic (notes, ADRs, design docs)
- Background context that constrains the approach
- Related past work — what was tried, what worked, what didn't

Even if no vault is linked, check `docs/plans/`, `docs/`, and the user's notes folder for existing design documents and prior plans.

### Project context

Read what already exists before planning against it:

- Prior drafts, briefs, related deliverables — the plan should fit, not fight
- Existing templates and conventions the user already follows
- Stakeholders or collaborators who must be consulted
- Any process the user's organization or venue requires

### Research summary

Compile findings into a **Research Summary** at the top of the plan document. Every task below should trace back to something learned here. If a task can't be grounded in research, that's a signal you need to research more before planning it.

---

## Phase 1 — Preflight

Before Stage 1 begins, verify that everything needed to execute the plan is in place. Discovering a missing source or expired access mid-execution wastes time and breaks flow.

### Preflight checklist

Verify each of these and report the result:

- [ ] **Tools / resources**: All tools, software, files, or resources required by the plan are available
- [ ] **Access**: Required permissions exist (archive access, interviewee availability, venue confirmation, login credentials)
- [ ] **Sources**: Primary sources are reachable; references can be retrieved; data exists where the plan assumes it does
- [ ] **People**: Anyone who must approve, review, or contribute is reachable on the plan's timeline
- [ ] **Environment**: The user can do the work in the place they intend to (right software, right files, right permissions)
- [ ] **Baseline**: Any prior work the plan builds on is in a known-good state

If any preflight check fails, stop. Fix it or flag it to the user before proceeding. Starting Stage 1 with a broken preflight is how you end up troubleshooting access issues instead of doing the work.

---

## Phase 2 — Stage Breakdown

Divide the project into sequential stages. Each stage is a coherent unit of work that produces a verifiable milestone — something you can point to and say "this is done end-to-end."

### Stage structure

```
Stage N: [Name]
  Goal:       What this stage achieves (one sentence)
  Depends on: Stage(s) that must be green first
  Blocks:     Stage(s) that cannot start until this stage's gate passes
  Risk:       LOW | MEDIUM | HIGH — why
  Rollback:   What to undo if this stage fails irreparably

  Tasks (dependency-ordered):
    Task N.1: [description]
      Depends on: [prior task(s) or "none"]
      Blocks:     [task(s) that wait on this one, or "none"]
      Parallel:   YES | NO  (can a sub-agent run this concurrently?)
      Evidence-of-done: [concrete way to know this task is finished]

    Task N.2: [description]
      Depends on: Task N.1
      Blocks:     Task N.3, Task N.4
      Parallel:   NO  (blocked by N.1)
      Evidence-of-done: [concrete way to know this task is finished]

  Stage gate:
    - [ ] Integration check 1
    - [ ] Integration check 2
    - [ ] No regressions in prior work
```

### Dependency marking

Every task and stage carries two dependency fields — this makes the graph navigable in both directions:

- **Depends on**: What must be green before this task/stage can start
- **Blocks**: What is waiting on this task/stage to finish

These fields are symmetric: if Task 2.1 depends on Task 1.3, then Task 1.3 must list Task 2.1 in its Blocks field. This redundancy is intentional — when a task finishes, you can immediately see what it unblocks without scanning the entire plan.

Mark each task's **Parallel** field:

- **YES** if the task has no unfinished dependencies (all its `Depends on` items are green or "none") — it can be dispatched to a sub-agent or assigned to a collaborator immediately
- **NO** if it's blocked — list which dependency is blocking it

### Ordering rules

1. **Stages are sequential.** Stage 2 does not start until Stage 1's gate passes
2. **Tasks within a stage follow their dependency graph.** If Task B needs output from Task A, Task A comes first — this isn't optional, it's structural
3. **Independent tasks can run in parallel.** If Tasks 2.3 and 2.4 have no dependency on each other, they can be worked simultaneously
4. A task cannot enter its Confirm-Adjust loop until every task it depends on is green

### Risk flags

Mark each stage with a risk level. This tells the user (and you) where to expect friction:

- **LOW**: Well-understood territory, clear path, prior art exists
- **MEDIUM**: Some unknowns — unfamiliar source, complex coordination, limited reference material
- **HIGH**: Novel territory, unreliable external dependencies, tight constraints, or no prior art

High-risk stages deserve extra care: consider a small pilot or prototype first, prepare the rollback plan in detail, and expect the Confirm-Adjust loop to cycle more than once per task.

### Rollback notes

Each stage documents what to undo if it fails beyond recovery. Half-built states with no way back are worse than not starting:

- Which deliverables or drafts to set aside
- Which decisions or commitments to revisit
- Which collaborators or stakeholders to update on the change of course
- Which side effects (messages sent, content published, contracts signed) cannot be undone — flag these explicitly

### Stage sizing

If a stage has more than 7 tasks, it's too large. Split it. Large stages hide integration problems behind a wall of individual task checks that all pass but don't add up. Aim for 3-5 tasks per stage.

---

## Plan Document Format

Output the plan as a markdown document following this structure. Save it to `docs/plans/YYYY-MM-DD-<topic>-plan.md` (or the user's preferred notes location).

```markdown
# Project Plan: [Name]
Date: [YYYY-MM-DD]

## Research Summary

### Online sources
- [What was found, with links]

### Vault / local docs
- [Prior decisions, background notes]

### Project context
- [Existing work, conventions, collaborators]

## Preflight

- [ ] [Check 1]: [how to verify]
- [ ] [Check 2]: [how to verify]

---

## Stage 1: [Name]

**Goal:** [one sentence]
**Depends on:** none
**Blocks:** Stage 2
**Risk:** LOW | MEDIUM | HIGH — [reason]
**Rollback:** [what to undo and how]

### Task 1.1: [description]
- **Depends on:** none
- **Blocks:** Task 1.2
- **Parallel:** YES
- **Evidence-of-done:** [exact criterion — a file exists, a draft has N sections, an interview is scheduled, etc.]
- **Confirm-Adjust max cycles:** 3

### Task 1.2: [description]
- **Depends on:** Task 1.1
- **Blocks:** Task 1.3, Task 2.1
- **Parallel:** NO (blocked by 1.1)
- **Evidence-of-done:** [exact criterion]
- **Confirm-Adjust max cycles:** 3

### Stage 1 Gate
- [ ] [Integration check]
- [ ] [No regressions in prior work]
- [ ] [Stage goal verified end-to-end]

---

## Stage 2: [Name]

**Goal:** ...
**Depends on:** Stage 1 gate passing
**Blocks:** Stage 3
**Risk:** ...
**Rollback:** ...

[Tasks with Depends on / Blocks / Parallel fields...]

### Stage 2 Gate
[Checks...]
```

---

## Phase 3 — The Confirm-Adjust Loop

This is the execution model for every task. No task is "done" until its evidence-of-done is in. "It looks right" is not done — produce the evidence.

```
    +----------+
    | Attempt  |  Do the work
    +----+-----+
         |
         v
    +----------+
    | Confirm  |  Check the evidence-of-done
    +----+-----+
         |
     +---+---+
     | Pass? |
     +---+---+
      |     |
    GREEN   NOT YET
      |     |
      |     v
      |  +----------+
      |  | Diagnose |  Read what's missing. Understand WHY
      |  +----+-----+
      |       |
      |       v
      |  +----------+
      |  |  Adjust  |  One targeted change based on diagnosis
      |  +----+-----+
      |       |
      |       v
      |    Re-confirm ----> back to Confirm
      |       |
      |  (max 3 NOT-YET cycles, then escalate)
      |
      v
  Next task
```

### Loop rules

1. **One adjustment per cycle.** Don't shotgun multiple changes hoping one sticks. Isolate the gap, fix that one thing, re-confirm
2. **Diagnose before adjusting.** Read what the evidence-of-done is missing. Form a hypothesis. Confirm it against the work. Only then make the change
3. **Max 3 NOT-YET cycles per task.** If a task fails its evidence check 3 times in a row, stop the loop and escalate to the user. Three failures usually means the approach is wrong, not just the execution. Continuing to loop is wasting time
4. **Never skip the confirm.** Every attempt ends with checking the evidence. No exceptions. The evidence is the only thing that says you're done

---

## Phase 4 — Stage Gates

After all tasks in a stage pass their individual evidence checks, run a stage-level integration check before proceeding to the next stage. Individual checks prove each piece works. Stage gates prove the pieces work together.

### What a stage gate checks

- **Integration**: The tasks in this stage hang together (e.g., the chapter outline matches the research summary, the campaign messages all support the core frame)
- **Regressions**: Prior work the stage built on still holds — the new work didn't undermine earlier choices
- **Goal verification**: The stage's stated goal is actually met end-to-end, not just task-by-task

### When a stage gate fails

If the gate fails, the problem is usually in how tasks interact, not in any single task. Identify which task interaction caused the failure, add a new evidence-of-done item for that interaction to the relevant task, and run that task through its Confirm-Adjust loop again.

---

## Phase 5 — Parallel Execution

When executing the plan, use sub-agents to run independent tasks concurrently. This is where the dependency graph pays off — you don't have to guess what can run in parallel, the `Blocks` and `Depends on` fields tell you exactly.

### Dispatch rules

1. **At stage start**, identify all tasks with `Parallel: YES` (no unfinished dependencies). Dispatch them all to sub-agents simultaneously
2. **When a task completes green**, check its `Blocks` list. For each blocked task, check if ALL of that task's dependencies are now green. If yes, dispatch it to a new sub-agent
3. **Never dispatch a task whose dependencies aren't all green.** The Parallel field in the plan is the initial state — during execution, a task becomes dispatchable only when its actual dependencies have passed
4. **Each sub-agent runs one task's Confirm-Adjust loop independently.** The sub-agent attempts the task, checks the evidence, and if NOT YET, diagnoses and adjusts within the 3-cycle limit. It reports back GREEN or ESCALATE
5. **Stage gate runs only after all tasks in the stage are green.** Don't start the gate while any task is still in its Confirm-Adjust loop

### Dispatch flow

```
Stage starts
    |
    v
Scan tasks: which have all dependencies green?
    |
    +---> Dispatch each ready task to a sub-agent (in parallel)
    |
    v
Wait for any sub-agent to finish
    |
    v
Task GREEN?
  |       |
  YES     NO (escalated after 3 NOT-YET cycles)
  |       |
  |       +--> Pause. Surface to user. Do NOT dispatch dependents
  |
  v
Check Blocks list of completed task
  |
  v
For each blocked task: are ALL its dependencies now green?
  |       |
  YES     NO
  |       |
  v       (wait for other dependencies)
Dispatch to sub-agent
    |
    v
(repeat until all tasks green or escalated)
    |
    v
Run stage gate
```

### What a sub-agent receives

Each sub-agent needs enough context to work independently:

- The task description and its evidence-of-done
- Relevant research findings from Phase 0 (not the entire research summary — just what this task needs)
- File paths, source pointers, or templates from the project context
- The Confirm-Adjust loop rules (attempt, confirm, diagnose, adjust, re-confirm, max 3 cycles)
- What to do on failure: report back with what's missing and why, don't keep looping silently

### Guardrails

- **No cross-task file conflicts.** Before dispatching parallel tasks, verify they don't modify the same files. If two tasks edit the same file, they must run sequentially even if the dependency graph says they're independent
- **Merge check after parallel tasks complete.** If multiple sub-agents wrote to the same stage's outputs, run a quick consistency check before the stage gate to catch conflicts introduced by parallel work
- **Failed task blocks its dependents.** If Task 2.1 fails and escalates, do not dispatch Tasks 2.3 and 2.4 that depend on it. Mark them as BLOCKED and surface the entire chain to the user

---

## Examples (non-coding)

### Example A — Book chapter rollout

A six-chapter trade book where Chapter 1 (Introduction) and Chapter 6 (Conclusion) depend on Chapters 2-5 being drafted. Plan: Stage 1 = research and outline all chapters (preflight: archive access, interview list confirmed). Stage 2 = parallel drafts of Chapters 2, 3, 4, 5 (file conflict check: separate chapter files). Stage 3 = sequential drafts of Chapter 1 and Chapter 6 referencing the middle chapters. Stage 4 = full-manuscript pass for tone consistency. Stage gate at each phase = beta-reader sign-off on the chapter set.

### Example B — Grant submission

Risk: HIGH (deadline-driven, multiple external dependencies). Stage 1 = letters of support (parallel asks; rollback = use a previous-cycle letter if a new one falls through). Stage 2 = budget + budget justification (sequential; depends on Stage 1 letters because the partner contributions affect the budget). Stage 3 = narrative sections in dependency order (project description, then methods which references it, then evaluation which references methods). Stage 4 = compliance review against funder's submission checklist. Preflight: institutional approval to submit, funder portal credentials work, character/page limits confirmed.

### Example C — Multi-day conference

Stage 1 = venue + dates locked (rollback: backup venues from Phase 0 research). Stage 2 = parallel asks to keynote speakers (file conflict = none; one task per speaker). Stage 3 = panel composition (depends on Stage 2 confirmations). Stage 4 = registration + comms (depends on Stage 1+2+3). Stage 5 = day-of run-of-show. Stage gates: Stage 2 gate = at least N keynote confirms (with explicit rollback to Plan B speakers); Stage 4 gate = registration system live and tested with a comp ticket.

---

## Checklist — Before Presenting the Plan

Before showing the plan to the user, verify:

- [ ] Every task has a concrete evidence-of-done — no "it should be ready" criteria
- [ ] Tasks within each stage follow their dependency order
- [ ] No task depends on something from a later stage
- [ ] Every stage has a risk flag with a reason
- [ ] Every stage has a rollback note
- [ ] Every stage has a gate with specific checks
- [ ] No stage has more than 7 tasks
- [ ] The research summary has actual findings, not placeholders
- [ ] Preflight checks cover all tools, access, and people needed by the plan
- [ ] Every task has both `Depends on` and `Blocks` fields — and they're symmetric
- [ ] Every task has a `Parallel` field (YES/NO) consistent with its dependencies
- [ ] No two parallel tasks modify the same file
- [ ] The plan is saved to a discoverable location

---

## Common Pitfalls

| Pitfall | What goes wrong | Fix |
|---------|----------------|-----|
| Tasks without evidence-of-done | You don't know if the task actually finished until 3 stages later when something doesn't add up | Write the evidence first. If you can't state it, the task is too vague — split or clarify it |
| Wrong task order | Task B fails because Task A's output isn't ready yet, wasting Confirm-Adjust cycles | Draw the dependency graph before ordering. If B reads from A, A comes first |
| Skipping research | You plan around a venue that just changed its submission rules, or a source that no longer exists | 20 minutes of research prevents days of rework. Check current sources |
| Monolith stages | A 12-task stage where one failing gate is impossible to diagnose | Split stages at natural boundaries. 3-5 tasks per stage |
| Vague stage gates | "Everything works" as a gate tells you nothing when it fails | Name the specific check. "Beta reader signs off in writing on the chapter set" |
| No rollback notes | Stage 3 fails, you've already published the announcement and signed the venue contract, and you don't know how to course-correct | Document rollback at planning time, not panic time |
| Infinite Confirm-Adjust loops | Cycling through adjustments without understanding the gap | 3 cycles max. If 3 targeted adjustments don't work, the approach — not just the execution — needs rethinking |
| Research-free planning | "I'll figure out the requirements as I go" | You won't. Research first, plan second, do third |
| Asymmetric dependencies | Task A says it blocks B, but B doesn't list A in Depends on — the graph is broken | Always write both directions. If you add a Depends on, update the other task's Blocks |
| Parallel file conflicts | Two collaborators edit the same draft simultaneously, producing merge conflicts or silent overwrites | Check file paths before dispatching. If tasks touch the same file, force sequential execution |
| Planning without clarifying | The prompt says "build a plan for the launch" so you plan a press push, but the user wanted an internal rollout | If the prompt is ambiguous about scope, audience, or constraints, ask before you plan. A 30-second question saves a 30-minute rewrite |
