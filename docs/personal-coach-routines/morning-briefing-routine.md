# Routine: Morning Briefing (scaffold-only)

**Trigger:** schedule, weekday mornings at user's preferred time.
**Connectors recommended:** Google Calendar (for "today's meetings"), Gmail (for top urgent threads, capped at 3).
**Privacy:** mixed — the Routine reads profile / journal / decisions metadata via filesystem, plus calendar/Gmail if granted. All of this transits Anthropic's cloud during execution. If profile/journal content is sensitive, prefer the desktop Scheduled Task path (`/personal-coach:setup-morning-briefing`) instead of this Routine.

## Prompt

```
Run the morning-briefing skill in non-interactive scaffold mode.

1. If the news-digest fired earlier today, locate the digest file at
   <vault>/News/<today>-digest.md. Note the path.
2. Read my profile (Goals → This week section) and my most recent
   journal entry. Note any committed action that wasn't marked done.
3. Scan my Decisions/ folder for entries with status pending/committed
   and deadline today or earlier. Note them.
4. If Calendar connector is granted, read today's meetings.
5. Write a briefing entry to <vault>/Journal/<today>-briefing.md with:
   - Frontmatter type: briefing.
   - Body sections for the five fields (acknowledge / one outcome / blocker /
     body / evening reflection prompt) — each as a PROMPT, not an answer.
     Example for the blocker field:
       "Likely blocker: <I'll fill this in when I sit down.
        Today's meetings: <list from calendar>. Yesterday's
        unfinished commit: <from journal>.>"
6. If decisions are due for grading, list them under a "Due for grading
   today" section so I can run /personal-coach:setup-decision-grading
   workflow at my own pace.
7. Stop.
```

## Why scaffold mode

The interactive briefing flow asks the user five questions one at a time. A Routine has no interactive user — answering for the user would be fabrication.

The scaffold mode produces the **structure** of the briefing with **inputs** the user might not remember (today's meetings, yesterday's unfinished items, decisions due) pre-populated, and the **answers** left blank for the user to fill in when they sit down.

This is the right division of labor: automation handles facts, the user handles judgment.

## Cadence

Default: weekdays at user-specified time. Avoid weekends — briefings on weekends usually aren't read and become noise.

## What this Routine will NOT do

- Will not answer the five briefing fields on the user's behalf.
- Will not edit the personal profile.
- Will not auto-grade decisions (it surfaces them; grading is interactive).
- Will not run on weekends unless the user explicitly sets `daily`.

## Verification

The first scheduled run produces a briefing scaffold. Open it and check:

- Are the five-field prompts present, with inputs pre-populated?
- Are decisions due for grading surfaced, if any?
- Is the news-digest path correctly linked, if news-digest also fired?

If the calendar connector is granted but no meetings appear, the connector may not be granting today's events — verify in Cowork's connector settings.
