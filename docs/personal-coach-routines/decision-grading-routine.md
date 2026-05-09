# Routine: Decision Grading (weekly scan)

**Trigger:** schedule, weekly (Monday morning is conventional).
**Connectors recommended:** none. The Routine reads only the local Decisions/ folder.
**Privacy:** moderate — decision-journal content transits Anthropic's cloud during the scan. If decisions involve highly sensitive corporate strategy, prefer the desktop Scheduled Task path (`/personal-coach:setup-decision-grading`) instead.

## Prompt

```
Run a weekly decision-grading scan.

1. List every file in <vault>/Decisions/ (or ~/.claude/decisions/) where:
   - frontmatter status is `pending` or `committed`, AND
   - frontmatter deadline is today or earlier.
2. For each match, extract:
   - Title (one sentence).
   - prior_lean and confidence%.
   - deadline date.
   - The "How we'll know in 90 days if this was right" criterion from the body.
3. Write a summary file at <vault>/Decisions/grading-due-<YYYY-MM-DD>.md
   listing the matches in deadline order (oldest first), with each entry
   pointing to the source file and quoting the falsification criterion verbatim.
4. If there are zero matches, write a one-line file: "No decisions due
   for grading this week."
5. Do NOT auto-grade any decision. The grading judgment is mine.
6. Stop.
```

## Why scan-only, not grade

Calibration depends on the user's honest retrospective judgment. If the Routine grades on the user's behalf, the calibration data is contaminated by the assistant's hindsight — which defeats the entire decision journal practice.

The Routine surfaces what's due. The user grades when they next open Cowork (Monday morning, ideally — fresh week, no urgency). Grading takes 5 minutes per decision; doing 1-3 per week is sustainable.

## Cadence

Weekly. Monday at the user's chosen time (often paired with morning-briefing — if so, use the bundled scheduled-task path instead of two separate Routines).

## What this Routine will NOT do

- Will not auto-grade.
- Will not edit the decision frontmatter `status` field — that flips to `graded` only when the user actually grades.
- Will not surface decisions that are still in their 90-day window.
- Will not propose a backfill marathon if many decisions are overdue.

## Verification

The first run produces `grading-due-<date>.md`. Open it and check:

- Are all overdue decisions listed?
- Is each deadline correct against the source file?
- Are any decisions you've already graded surfaced anyway? (If yes — likely a `status` field that wasn't updated; clean up the source file.)
