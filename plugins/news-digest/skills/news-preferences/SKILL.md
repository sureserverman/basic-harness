---
name: news-preferences
description: Use to capture, update, or read the user's news preferences — topics of interest, topics to exclude, preferred sources, sources to avoid, language, length, style, cadence, and an open-questions watchlist. The substrate the news-digest skill reads from. Triggers on "set up my news", "what news do I want", "update my news preferences", "what topics am I tracking", "I want a news digest about", "remember I'm interested in", or whenever the user mentions a recurring news interest the digest should know about. Never edits silently — every write is shown and confirmed.
---

# News Preferences

The substrate for `news-digest`. Captures **what kind of news the user wants** so the digest produces signal, not noise. Without this file, news-digest will refuse to run — generic headlines are exactly what the user already gets from every app on their phone.

**Announce at start:** "Using the news-preferences skill to <read | update | initialize> your news preferences."

<HARD-GATE>
Never write or modify preferences without showing the exact change to the user and getting an explicit "yes". Silent drift is the failure mode this skill exists to prevent.
</HARD-GATE>

## Where the file lives

Resolution order (use the first that exists):

1. **Vault path** — `<vault>/Profile/news-preferences.md` if a personal vault is bootstrapped.
2. **Home path** — `~/.claude/news-preferences.md`, the default.

Project-scoped news preferences don't make sense — news interests are per-person, not per-project — so unlike `personal-profile`, this skill skips the project path option.

## Schema

```markdown
---
title: News Preferences
type: news-preferences
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
timezone: <IANA TZ — e.g., Europe/Tallinn>
---

# News Preferences — <user's preferred name>

## Topics of interest
<one bullet per topic; weight is high / medium / low>
- <topic>: <weight> — <one-line "why this matters to me", optional but recommended>

## Topics explicitly excluded
<one bullet per excluded topic; the digest will filter against these>
- <topic> — <reason if relevant>

## Preferred sources
<outlets the user trusts; the digest weights these higher>
- <source name> — <URL or "via search" or "RSS: <url>">

## Sources to avoid
<outlets the user has rejected — paywalled-and-not-subscribed, low-trust, repetitive>
- <source name> — <reason>

## Language
- **Primary:** <language>
- **Acceptable:** [<list — e.g., English, Russian>]
- **Auto-translate from:** [<list of source languages to render in primary>]

## Format preferences
- **Length:** <short (5 items) | medium (10 items) | long (20 items)>
- **Style:** <facts only | analytical | analytical with named opinion>
- **Structure:** <flat bullets | sectioned by topic | sectioned by source>
- **Per-topic cap:** <max items per topic — default 3>
- **Include "why this matters" line per item:** <yes | no>

## Cadence
- **Default firing time:** <HH:MM in user's timezone>
- **Days:** <weekdays | daily | mon,wed,fri | custom>

## Open questions on watch
<things the user is actively tracking that may not be daily news but should surface when there is movement; the digest checks these on every run>
- <one-line question or topic — e.g., "Any movement on the EU AI Act enforcement timeline">

## Hard exclusions (sensitive)
<topics that cause real harm to surface; the digest filters these strictly, no "but it's relevant" carve-outs>
- <topic>

## Change log
- <YYYY-MM-DD> — <what changed, in one line>
```

## Phase 1 — Read before write

If the file exists, **read it first** and summarize back what's there in 4-6 lines. The user confirms the assistant has the right baseline before any edit.

If the file doesn't exist, ask the four bootstrap questions in order — one per message:

1. "What 3-5 topics do you want me to track for you in a daily digest? Just names; we'll add detail later." (e.g., "fintech regulation in EU/UK", "AI policy", "crypto enforcement", "the company I work for: <name>".)
2. "What topics should I explicitly **not** include?" (e.g., "no celebrity, no sports, no US domestic politics".)
3. "Any sources you specifically trust or want me to weight higher?" (e.g., "FT, Bloomberg, EUR-Lex for regs, /r/fintech".)
4. "What time of day, and which days?" (e.g., "8:00 weekdays, Europe/Tallinn".)

Write a minimal preferences file from those four answers. Everything else is added incrementally as the digest runs and the user reacts to it.

## Phase 2 — Capture during conversation

Listen for signal in normal conversation:

- "Tell me more about <topic>" → candidate for **Topics of interest**.
- "I'm tired of <topic> headlines" → candidate for **Topics excluded**.
- "<source> is the only one I trust on this" → candidate for **Preferred sources**.
- "I keep wondering about <X>" → candidate for **Open questions on watch**.

When you spot signal, **propose** the edit using this format:

```
I'd like to add to your news preferences:

  Section: <Topics of interest>
  Add bullet: "EU AI Act enforcement: high — affects fintech compliance roadmap"
  Reason: user mentioned this twice in last two sessions.

Save? (yes / no / edit)
```

## Phase 3 — Write surgically

- Use `Edit`, never `Write`. Bump `updated:`. Append to `## Change log`.
- Touch exactly the section named in the proposal.

## Phase 4 — Read on demand

When `news-digest` runs, it reads this file. If the `personal-coach` plugin is also installed, its skills may consult this file in narrow ways:

- `personal-profile` may suggest adding a topic to news preferences when a goal is added (e.g., "you set a goal around <X>; want to track news on that?").
- `business-mentoring` may suggest adding a topic when a decision touches an ongoing market situation.

Both integrations are soft — if `personal-coach` is not installed, `news-preferences` and `news-digest` work fine on their own.

## Hard rules

- **Topics ≠ identity.** This file describes informational interests, not personal politics. The skill does not record "the user is pro-X" or "anti-Y" — it records "the user wants to read about Z".
- **Hard exclusions are strict.** No "but it was relevant to today's story" carve-outs. If the user has put a topic in **Hard exclusions**, the digest never surfaces it, period.
- **Open questions don't become assumptions.** "Any movement on X lawsuit" is a search prompt, not a claim that X lawsuit is a concern of the user — for the latter, write to the personal profile.

## Examples

### First-time setup (compact)

> User: "Set up news for me. I care about fintech regulation in EU and UK, AI policy, and what's happening with my company. Skip celebrity and sports."

Right reaction: ask for the missing pieces (sources, time, days) in three quick messages, then show the draft preferences file and ask "save this?".

### Updating mid-conversation

> User: "I keep seeing crypto crime stories that aren't relevant to me. Skip those."

Right reaction:

> Proposing:
>
>   Section: Topics excluded
>   Add bullet: "Crypto crime / heist / hack reporting — user finds these noise."
>
> Save? (yes / no / edit)

### Refusing inferred edits

> User: "I think a lot of fintech coverage is too breathless."

Right reaction: do **not** silently add "anti-fintech-hype" to preferences. Ask: "Want me to add a preference to weight measured / analytical sources over breathless coverage?". If the user says no, drop it — it was an opinion, not an instruction.

## Sources and rationale

- **Confirm-before-write** — silent edits to a preferences file produce drift the user cannot audit; explicit confirmation prevents it.
- **Open-questions watchlist** — adapted from intelligence-analysis "Indicators and Warnings" practice (Heuer, *Psychology of Intelligence Analysis*, 1999, Ch. 10): cheap-to-track signals that beat retrospective surprise.
- **Hard exclusions are strict** — Tversky & Kahneman's anchoring effect (1974): "just one mention" of an excluded topic still reshapes the user's day. Strict filtering is the only filter that works.
- **Source-weighting over inclusion-only** — Tetlock & Gardner, *Superforecasting* (2015), Ch. 6: source quality dominates volume in forecasting accuracy; news consumption follows the same logic.
