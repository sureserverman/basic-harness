---
description: One-shot setup for a recurring decision-grading scheduled task in Claude Cowork — every week, surfaces decision-journal entries whose 90-day grading deadline has landed and walks the user through grading them. In Claude Code (no scheduler), prints a shell-cron equivalent.
---

# Setup Decision Grading

Wire a weekly check that scans `Decisions/` for entries whose `deadline` has passed and surfaces them for grading. Without grading, the decision journal is a diary; with it, the user actually calibrates over time.

## Step 1 — Detect host

Same logic as `setup-morning-briefing`. Cowork supports `/schedule`; Code uses shell cron.

## Step 2 — Ask the user two questions

1. "Which day of the week should the grading check fire? (`monday` is conventional — fresh week, no urgency.)"
2. "What time? (24-hour, e.g., `09:30`. Pick a slot you'll actually have 5 minutes for.)"

Resolve to user's timezone. If timezone is in their profile, mention it.

## Step 3a — Cowork path

Present the scheduled-task prompt:

```text
Run a weekly decision-grading check.

1. List every file in <vault>/Decisions/ (or ~/.claude/decisions/) where:
   - frontmatter status is `pending` or `committed`, AND
   - frontmatter deadline is in the past (today or earlier).
2. For each match, surface:
   - The decision title (one-sentence).
   - The original prior_lean and confidence%.
   - The deadline date.
   - The "How we'll know in 90 days if this was right" criterion.
3. If I'm here interactively, walk me through grading each one. For each:
   - Ask: right-call / wrong-call / right-call-wrong-reasons / wrong-call-right-reasons.
   - Ask one sentence on what we got right.
   - Ask one sentence on what we got wrong.
   - Ask one sentence on the lesson for the next decision of this shape.
   - Append the Grading section to the file and update frontmatter status to `graded`.
4. If non-interactive (scheduled run with no user present), write a stub entry to
   <vault>/Decisions/grading-due-<YYYY-MM-DD>.md listing the matches so I can grade
   them when I sit down.
5. Stop. Do not auto-grade — the call is mine, not the assistant's.
```

Then tell the user:

> Open Cowork → **Scheduled** in sidebar → **+ New task** → paste the prompt → set time `<HH:MM>` on `<day>` → save.
>
> Or in any Cowork chat: `/schedule`, paste the prompt, same result.

## Step 3b — Code path

```cron
<minute> <hour> * * <day-num> /usr/local/bin/claude -p "Scan ~/.claude/decisions and any vault Decisions/ folder for entries with status pending/committed and deadline in the past; print the list" >> ~/.claude/journal/cron.log 2>&1
```

Day numbers: `1` = Monday, `7` = Sunday. Tell the user this fires the check non-interactively; they'll need to open Code interactively to actually grade.

For Code, recommend pairing the cron entry with a desktop reminder so the user opens Code and runs the grading walk-through manually after the cron fires.

## Step 4 — Save the choice

Append to `~/.claude/personal-coach.local.md`:

```markdown
- <YYYY-MM-DD> setup-decision-grading — host: <Cowork | Code>, day: <day>, time: <HH:MM>
```

## Step 5 — Hand off

> Setup recorded. The first weekly check fires `<day>` at `<HH:MM>` `<TZ>`. Anything past its deadline by then will be surfaced for grading. If no decisions are due that week, the task runs and produces a one-line "no decisions due" log entry — that's expected.

## Hard rules

- **The assistant never grades a decision on the user's behalf.** Surfacing is automated; grading is manual. Auto-grading would corrupt the calibration data the journal exists to build.
- **No backfill.** If the user has a hundred ungraded historical decisions, the skill does not propose a grading marathon — it surfaces them in order of deadline and lets the user grade at a sustainable pace.
- **Privacy:** decision-journal content stays on disk. The scheduled task reads from local folders only. If running as a cloud Routine instead (see `routines/decision-grading-routine.md`), the file contents transit Anthropic's cloud — flag the tradeoff.
