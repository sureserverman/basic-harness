---
name: brainstorming
description: Use before any non-trivial creative or strategic work — research designs, campaigns, briefs, lesson plans, proposals, project pitches, or any deliverable that benefits from a validated design before drafting. Turns a vague idea into a validated design by exploring purpose, constraints, alternatives, and risks one question at a time. Terminal handoff is to the planning-projects skill, which produces the staged plan. Triggers on "I want to do", "let's design", "help me think through", "how should I approach", "brainstorm", "what should this look like", "I'm starting a project on".
---

# Brainstorming Ideas Into Designs

Turn an idea into a design the user has explicitly validated, section by section, before any planner or implementer touches the work. The output of this skill is a **design document** that the next skill (`planning-projects`) uses as input.

**Announce at start:** "Using the brainstorming skill to turn this idea into a validated design."

<HARD-GATE>
Do NOT invoke `planning-projects`, start drafting the deliverable, or run any execution skill until you have presented a design and the user has said yes. This applies to every project regardless of perceived simplicity. The design may be three sentences for a trivial task — but it must be presented and approved before handoff.
</HARD-GATE>

## Core principle

Design work is about surfacing the assumptions that separate "what the user said" from "what the user meant" — and the constraints that separate "what works in the head" from "what works in practice." Skipping this produces plans that solve the wrong problem, and a wrong plan executed perfectly is worse than no plan (see also `planning-projects`, Phase -1).

## Anti-pattern: "this is too simple"

"Simple" is where unexamined assumptions hide. A two-page brief has an audience model. A workshop agenda has a rollback strategy (what if the room is half-empty?). A single-paragraph statement has a tone that will be read by people you didn't picture. Every project goes through brainstorming — the design can be compact, but it must be validated.

---

## Checklist

Create a task for each of these and work them in order. Do not skip ahead.

1. **Ground the idea in available context** — read the user's existing notes, prior drafts, related documents
2. **Clarify purpose and constraints** — one question at a time, using the question heuristics below
3. **Surface alternatives** — propose 2-3 approaches with tradeoffs; name your recommendation
4. **Pre-mortem the recommended approach** — what would cause this to fail or be regretted in 6 months?
5. **Present the design in sections** — purpose, audience, structure/components, key dependencies, risks, rollout; each section scaled to its complexity; confirm after each
6. **Write the design document** — save to `docs/plans/YYYY-MM-DD-<topic>-design.md` (or wherever the user keeps planning notes)
7. **Hand off to planning-projects** — that skill produces the staged plan from this design

---

## Phase 1 — Ground the idea

Before asking anything of the user, gather what the available context already tells you. The fewer questions you ask, the better — questions you can answer from evidence should not be asked.

- Read any prior drafts, briefs, or notes on the same topic the user has shared
- Scan `docs/`, `docs/plans/`, or the user's notes folder for prior decisions on this topic
- If the work is part of an ongoing engagement, read the earlier deliverables to match tone, structure, and assumptions
- Look for patterns in the user's existing work that a new piece should match (don't propose a third citation style, a second template, a parallel taxonomy)

If an Obsidian vault is linked via `vault-context`, `.claude/vault-context.md` already points to relevant prior work — consult it before asking the user.

## Phase 2 — Clarify, one question at a time

Ask only what you cannot infer. For each question:

- **Prefer multiple choice** when the answer space is finite
- **State an assumption and ask for confirmation** when you can infer an answer
- **One question per message.** A wall of questions gets a wall of shallow answers
- **Stop asking when you have enough to design** — not when you have everything you'd ever want

### What to clarify (5W1H framing)

- **Why** — what is the underlying problem or opportunity? What happens if we do nothing?
- **Who** — who is the audience or the user of the end result?
- **What** — what is explicitly in scope and what is explicitly out?
- **When** — deadlines, deliverable dates, ordering with other work
- **Where** — what venue, channel, format, or platform is the output for?
- **How (success)** — how will the user know it's done? What does "good" look like, concretely?

### Red flags in user answers

- "Just like X but …" — compatibility requirements hiding as scope
- "Eventually we'll also want …" — future work trying to smuggle into current scope
- "It should be flexible" — unknown requirements masquerading as an abstraction
- "Obviously" or "of course" — the user has an unstated assumption the design should make explicit

When you hear these, ask the follow-up that pulls the hidden requirement to the surface.

## Phase 3 — Surface alternatives

Never present a single "the design." Always propose 2-3 approaches with named tradeoffs. This protects against anchoring on the first idea and gives the user material to push back against.

Format:

```
Option A — <one-phrase name>
  How it works: <2-3 sentences>
  Tradeoff: <what you give up>
  Fits when: <condition>

Option B — ...
Option C — ...

Recommendation: <A/B/C> because <specific reason>.
```

Ruthless **YAGNI** — cut features or sections that aren't in the "why" from Phase 2. Ruthless **KISS** — the simpler option wins unless a concrete need justifies the complexity.

Consider the classic forces (Christopher Alexander's pattern vocabulary, applied broadly):

- Clarity vs depth
- Flexibility vs simplicity
- Coupling vs reuse (a self-contained brief is portable; one that depends on three other documents is harder to circulate)
- Explicit vs convention

Name which forces are in tension in this design — it makes the tradeoff visible.

## Phase 4 — Pre-mortem

Before presenting the design, imagine it's six months later and the design is regretted. What went wrong? Three sources to check (Gary Klein's pre-mortem technique and Kahneman's planning-fallacy work):

- **Integration** — did this conflict with related work, prior commitments, or audience expectations?
- **Operability** — is this hard to update, hard to revisit, hard to retract if the situation changes?
- **Scope creep** — did we carry forward something that should have been cut?

Write down the top 2-3 failure modes and what the design does to prevent them. If a failure mode has no mitigation, that's a real risk — either add a mitigation, accept it explicitly, or revisit the choice of approach.

## Phase 5 — Present the design in sections

Walk the user through these sections. Scale each to the project's complexity — a few sentences for a trivial task, up to 200-300 words for nuanced areas. After each section, ask "does this match what you had in mind?" and be ready to revise.

1. **Problem statement** — the "why" from Phase 2, one paragraph
2. **Audience and shape** — who reads / receives this, in what venue, in what format
3. **Components** — the named pieces of the deliverable (sections of a brief, phases of a campaign, modules of a course), each with a one-line responsibility
4. **Flow** — how the audience moves through the deliverable (sequence, escalation path, decision points)
5. **Dependencies** — what must already be true (data, prior approval, source access) for this to work
6. **Risks and unknowns** — what's uncertain; what would change the approach if we learned it
7. **Rollout** — how does this go out (one big release? staged? internal review first?); how do we revise or retract?

Do not present sections 2-7 until section 1 is approved. Do not skip sections because "they don't apply" — if they don't apply, say so explicitly ("no rollout concerns: this is an internal working draft only"). Explicit-negative is better than silent-missing because it shows the reader you considered it.

## Phase 6 — Write the design document

Save to `docs/plans/YYYY-MM-DD-<topic>-design.md` (or the user's preferred notes location). Structure:

```markdown
# Design: <Topic>
Date: <YYYY-MM-DD>

## Problem
<why, constraints, success criteria>

## Audience and shape
<who reads it, where, in what form>

## Alternatives considered
- Option A: <name> — <why not>
- Option B: <name> — <why not>
- Option C (chosen): <name> — <why>

## Components
<per-component responsibilities and boundaries>

## Flow
<how the audience moves through it>

## Dependencies
<what must be true for this to work>

## Risks (from pre-mortem)
- <failure mode> — mitigation: <what we do>
- <failure mode> — mitigation: <what we do>

## Rollout / revision
<how to release, how to update, how to retract>
```

Save the design document before handoff so future sessions can find it.

## Phase 7 — Hand off to planning-projects

The **only** skill you invoke after brainstorming is `planning-projects`. Do not invoke `executing-plans` or any execution skill directly — those are downstream of the plan.

Say to the user:

> Design approved and saved to `docs/plans/<filename>-design.md`. Handing off to the `planning-projects` skill to produce the staged plan with research, preflight, tasks, and stage gates.

Then invoke `planning-projects` with the design document as input.

---

## Examples (non-coding)

These are illustrative — the structure works the same in coding work, this skill just stays neutral about the domain.

### Example A — Research-question design

User: "I want to write a paper on how local newspapers covered the 2008 housing crisis."

Brainstorming pulls out:

- **Why**: gap in the literature? class assignment? trade publication?
- **Who**: peer reviewers? a newsroom? a general reader?
- **What's in/out**: which papers, which years, which angles (foreclosure data? editorial framing? source diversity?)
- **Alternatives**: (A) close reading of 3 papers in depth; (B) corpus analysis across 50 papers; (C) interviews with the reporters who covered it. Recommend A if depth matters more than breadth, B if you want quantifiable claims, C if narrative is the deliverable.
- **Pre-mortem**: archive paywalls; some papers no longer exist; scope balloons to "the whole crisis" if "framing" isn't pinned down.

### Example B — Campaign brief

User: "Help me design an awareness campaign about water-bill late fees."

Brainstorming pulls out:

- **Why**: late fees disproportionately hit the same households repeatedly; campaign goal is policy change vs. consumer education vs. both
- **Who**: city council? affected residents? a coalition of nonprofits?
- **Components**: messaging frame, channels, calls to action, success metric
- **Alternatives**: (A) data-led op-ed campaign; (B) door-to-door + town hall; (C) social-media-led storytelling. Tradeoffs around budget, timeline, durability of attention.
- **Pre-mortem**: data is harder to obtain than assumed; coalition partners want different things; campaign is conflated with an unrelated political race.

### Example C — Lesson plan

User: "I'm teaching a 90-minute high-school workshop on critical reading."

Brainstorming pulls out:

- **Why**: students arrive unable to distinguish opinion from reporting; prior workshop relied on materials students rejected as "boring"
- **Audience and shape**: 25 students, 90 minutes, single session, must work without internet
- **Components**: hook (5 min), framing (15), worked example (20), small-group exercise (30), debrief (15), takeaway (5)
- **Alternatives**: (A) one long worked example; (B) compare-and-contrast across three sources; (C) game-based with team scoring. Tradeoffs around engagement, depth, replicability for other teachers.
- **Pre-mortem**: materials too long to read in session; the "right answer" framing kills group discussion; the takeaway is forgettable.

In all three examples, the deliverable is a **design document**, not the paper / campaign / lesson plan itself. `planning-projects` produces the staged plan to actually build it.

---

## Key principles

- **One question at a time** — break complex topics into multiple messages
- **Multiple choice preferred** — easier to answer than open-ended
- **YAGNI and KISS** — cut unnecessary scope, pick the simpler approach, let the need justify complexity
- **Explicit alternatives** — never a single design; always 2-3 with tradeoffs
- **Pre-mortem before approval** — name the failure modes before they happen
- **Validated by section, not in bulk** — present a section, get approval, move on
- **Explicit-negative** — "no rollout concerns" beats silent omission
- **Terminal state is `planning-projects`** — never jump straight to execution

---

## Sources and rationale

Cited so the methodology is defensible:

- **One question at a time / multiple choice** — Socratic dialogue tradition; easier to reason about a constrained choice than generate an open-ended answer
- **YAGNI / KISS / DRY** — *The Pragmatic Programmer* (Hunt & Thomas); Kent Beck's *Extreme Programming Explained*. The principles generalize beyond code.
- **Explicit alternatives with tradeoffs** — ADR (Architecture Decision Record) practice; Michael Nygard, "Documenting Architecture Decisions". Equivalent to the "alternatives considered" section in academic writing and policy memos.
- **Pre-mortem technique** — Gary Klein, *Performing a Project Premortem* (Harvard Business Review, 2007)
- **Planning fallacy** — Daniel Kahneman, *Thinking, Fast and Slow*, Ch. 23
- **Christopher Alexander pattern forces** — *A Pattern Language* (1977)
- **5W1H clarification** — journalism/business-analysis standard; see Kipling's "six honest serving men"
- **Section-by-section validation** — stage-gate process (Robert Cooper, *Winning at New Products*)

These are why the skill looks the way it does — the shape is not arbitrary.
