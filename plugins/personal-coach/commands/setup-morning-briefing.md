---
description: One-shot setup for a recurring morning-briefing scheduled task in Claude Cowork — asks for time + days, generates the right /schedule prompt, and walks the user through enabling it.
---

# Setup Morning Briefing

Wire the `morning-briefing` skill into Cowork's Scheduled Tasks so it fires automatically every day. The whole point of morning-briefing is daily cadence — running it manually defeats the design.

## Step 1 — Ask the user three questions (one per message)

1. "What time should the briefing fire? (24-hour, e.g., `08:00`)"
2. "Which days? (`weekdays` / `daily` / `mon,wed,fri` / custom)"
3. "Should I include `news-digest` ahead of the briefing in the same scheduled task, so news lands first and the briefing can reference it? (yes / no)"

Resolve relative terms ("morning") to a concrete time before proceeding. If the user's timezone is in their personal profile or news-preferences file, mention it explicitly: "I'll schedule this in `Europe/Tallinn` based on your news preferences — confirm or override."

## Step 2 — Build the scheduled-task prompt

Build the prompt and present it to the user:

```text
Run my morning routine for today.

1. If the news-digest skill is available and my news preferences exist, run it first.
   Save the digest to its standard path. Note the path for the briefing.
2. Run the morning-briefing skill. Pull from my personal profile, recent journal,
   any decisions whose deadline has landed, and the news-digest if produced above.
3. If I'm here interactively, walk me through the five briefing fields.
   If this is a non-interactive scheduled run, write a scaffold entry with the
   five fields as prompts (not answers) so I can fill them in when I sit down.
4. Surface any decisions due for grading.
5. Save the briefing entry to the standard journal path. Stop.
```

Adjust the news-digest line based on the answer to question 3.

## Step 3 — Walk the user through Cowork's UI

Tell the user, verbatim:

> Open Cowork → click **Scheduled** in the sidebar → click **+ New task** → paste the prompt above → set time `<HH:MM>` and days `<answer>` → save.
>
> Alternatively, in any Cowork chat, type `/schedule` and paste the prompt — same result.
>
> Verify by clicking **Scheduled** afterwards: the new task should appear with the cadence you set. The first scheduled run is the real test — if anything looks off in tomorrow morning's briefing, run `/personal-coach:setup-morning-briefing` again to regenerate the prompt.

Do **not** attempt to invoke `/schedule` from this command. Cowork's scheduler is a UI surface; the user creates the task themselves so they can see what they're authorising.

## Step 4 — Save the choice

Append a one-line entry to `~/.claude/personal-coach.local.md` (creating the file if it doesn't exist) so future sessions know the briefing is wired up:

```markdown
---
plugin: personal-coach
---

## Setup history
- <YYYY-MM-DD> setup-morning-briefing — time: <HH:MM>, days: <pattern>, news-digest-bundled: <yes | no>
```

This file is local-only and is never committed.

## Step 5 — Hand off

Tell the user:

> Setup recorded. The first scheduled run is tomorrow at `<HH:MM>` `<TZ>`. If you want to set up the news-digest as a standalone scheduled task instead of bundling it, run `/personal-coach:setup-news-digest`. If you want decision-grading on a weekly cadence, run `/personal-coach:setup-decision-grading`.

## What this command will NOT do

- Will not call `/schedule` itself. The user creates the scheduled task in the Cowork UI so the authorisation flow is visible.
- Will not invoke `morning-briefing` to "test" it — the test is tomorrow morning, when it actually fires.
- Will not work in non-interactive sessions. If there's no user available to answer the questions, exit with a one-line note and recommend re-running interactively.
