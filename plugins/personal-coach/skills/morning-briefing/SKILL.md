---
name: morning-briefing
description: Use to start a working day with a deliberate frame — surfaces yesterday's commitments, today's intentions, blockers, and one reflection prompt. Reads the user's profile, recent journal entries, and decision-journal entries that hit their 90-day grading deadline. Triggers on "morning briefing", "let's start the day", "what's on for today", "daily standup with myself", "good morning", or first message of a session in the user's local morning. Short and structured — five fields, not a 20-minute ritual.
---

# Morning Briefing

A 5-minute self-standup. Pulls together the artifacts the other skills produced (profile, journal, decisions) into a single, predictable frame for the day. The point isn't planning — it's **deliberate entry** into the day, instead of opening Slack and reacting.

**Announce at start:** "Using the morning-briefing skill — five fields, then we're done."

## Phase 1 — Read

Pull from disk, in this order:

1. **Profile** — `personal-profile`'s **Goals → This week** section. If absent, skip.
2. **Yesterday's journal** — most recent file in `<vault>/Journal/` or `~/.claude/journal/`. Quote one line if there's a committed action that wasn't marked done.
3. **Decisions due** — any file in `<vault>/Decisions/` or `~/.claude/decisions/` with `status: pending` and `deadline` ≤ today. These need grading.
4. **Open recurring topics** — anything in profile's **Recurring decisions** field.

Do not read more than these. The briefing is short; bloat kills it.

## Phase 2 — The five fields

Walk the user through, one at a time:

1. **One thing from yesterday I want to acknowledge.** Win, lesson, or unresolved feeling. One sentence.
2. **The one outcome that would make today a good day.** One outcome, not three. Force the prioritisation.
3. **The blocker most likely to derail #2.** Internal (focus, energy) or external (waiting on someone). Name it before it bites.
4. **A commitment for the body.** Sleep, walk, food, training — one thing. The skill is opinionated about this because cognitive performance follows physical state, not the other way round (Walker, *Why We Sleep*, 2017; Ratey, *Spark*, 2008).
5. **One reflection prompt for the end of the day.** Pick one from the rotating list below or write a custom one.

Rotating reflection prompts:

- "When did I feel most like myself today?"
- "What did I avoid that I should look at tomorrow?"
- "Where did I add value vs. where was I just busy?"
- "Whose advice or example helped me today?"
- "What did I learn about <recurring decision topic>?"

## Phase 3 — Decisions due grading

If Phase 1.3 surfaced any decisions hitting their grading deadline, surface them here:

> Two decisions are due for grading today:
>
> - 2026-02-05 — "Hire <name> as Head of Compliance" — your prior lean was A, 70%. How did it land? right-call / wrong-call / right-call-wrong-reasons / wrong-call-right-reasons.
> - ...

Walk the user through grading each. Update the file's frontmatter `status: graded` and append:

```markdown
## Grading (90-day review, <YYYY-MM-DD>)
- **Outcome:** <right-call / wrong-call / right-call-wrong-reasons / wrong-call-right-reasons>
- **What we got right:** <one or two sentences>
- **What we got wrong:** <one or two sentences>
- **Lesson for the next decision of this shape:** <one sentence>
```

Calibration over time is the whole point of the decision journal — without grading, it's a diary.

## Phase 4 — Save (one line)

Append to `<vault>/Journal/YYYY-MM-DD-briefing.md` (or `~/.claude/journal/...`):

```markdown
---
date: <YYYY-MM-DD>
type: briefing
---

# Morning briefing — <YYYY-MM-DD>

- **Acknowledge:** <Phase 2.1>
- **One outcome:** <Phase 2.2>
- **Likely blocker:** <Phase 2.3>
- **Body commitment:** <Phase 2.4>
- **Evening reflection prompt:** <Phase 2.5>
```

That's it. Don't extend the briefing into a full planning session — that's `business-mentoring` or `planning-projects`, not this.

## Hard rules

- **Five fields. Not six.** If the user wants to add more, suggest they open a separate session for it.
- **Do not produce a generated to-do list.** The skill writes the entry, not the day.
- **Honor "skip the briefing today".** Some days the user shouldn't be briefed — they should be left alone. The skill is for when it helps, not as a duty.

## In Cowork (connector-aware enrichment)

Morning-briefing is the skill most worth running as a **Scheduled Task** in Cowork — its whole reason for existing is daily cadence. Use the `/personal-coach:setup-morning-briefing` command to wire it up.

When connectors are granted:

- **Google Calendar** — Phase 2 field 3 ("the blocker most likely to derail #2") gains a real input: the meetings on today's calendar. The skill surfaces the most disruptive one rather than asking the user to remember.
- **Gmail** — the skill may surface 1–3 unread threads marked urgent (starred, important-tagged, from a small allowlist of senders the user names). Strict cap: never more than three; never paste the email body into the briefing.
- **News digest pairing** — if the `news-digest` companion plugin is installed and its skill has fired earlier the same morning, surface the link to today's digest at the top of the briefing for context. Do not duplicate the digest content into the briefing. Skip this when the companion plugin is not present.

If running as a Scheduled Task without an interactive user:

- The skill writes the briefing entry with whatever it can ground (calendar, news-digest path, decisions due) and leaves Phase 2 fields 1–5 as **prompts** rather than answered fields. The user fills them in when they sit down at their desk. The scheduled task produces the scaffold, not the content.

## Sources and rationale

- **One-outcome forcing function** — Greg McKeown, *Essentialism* (2014), and the journalism "lede" discipline.
- **Body-commitment field** — Matthew Walker, *Why We Sleep* (2017); John Ratey, *Spark: The Revolutionary New Science of Exercise and the Brain* (2008).
- **Evening reflection prompt** — Stoic *praemeditatio* / examen tradition; *Meditations* II.1, V.1; Marcus Aurelius's morning-and-evening structure.
- **Decision-journal grading cadence** — Annie Duke, *Thinking in Bets* (2018), Ch. 5 on calibration.
