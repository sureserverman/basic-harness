---
description: Two-minute calm read-only walkthrough of the personal-coach plugin — what each skill is for, what each subagent does, what the routines and setup commands are, where files land. For users who've already done /personal-coach:onboard (or want to skip onboarding) and want the wider map. Read-only — no profile writes, no scheduling, no skill invocations.
---

# Tour (personal-coach)

A two-minute walkthrough of the personal-coach ecosystem. This is the depth tour for a user who's seen the marketplace map (`/welcome:tour`) or finished onboarding and wants to know what else is around. **Read-only.**

## Step 0 — Detect language

Same as `/personal-coach:onboard` and `/welcome:tour`. Detect from invocation; ask in three languages if ambiguous.

## Step 1 — The shape (one paragraph)

Equivalent of:

> `personal-coach` is shaped around three roles Claude can play for you, each grounded in primary-source methodology:
>
> - **Personal assistant** that remembers who you are across sessions.
> - **Reflective listener** for journaling and CBT-style reframing — not therapy, with explicit limits.
> - **Business mentor** for hard decisions, with frameworks like JTBD / Cynefin / pre-mortem and a decision journal that gets graded 90 days later.
>
> Plus a daily rhythm: a morning briefing. The news digest and fintech legal triage live in their own plugins (`news-digest` and `fintech-legal-advisor`) — install those alongside if you want either.

## Step 2 — Skills (one disclosure level — short labels)

> Five skills. You don't memorize these — they pick themselves up from what you say. Just so you know what's around:
>
> - **`onboarding`** — the value-first first-run flow you may already have done.
> - **`personal-profile`** — your persistent profile (name, role, values, goals, sensitivities). Edited only with your confirmation.
> - **`reflection-session`** — structured CBT-style reflection. Safety-gated.
> - **`business-mentoring`** — strategic-decision sparring with a decision journal.
> - **`morning-briefing`** — five-field daily standup with yourself.

## Step 3 — Subagents (one paragraph)

> Two model-pinned worker agents that get dispatched automatically when the corresponding skill needs them:
>
> - **`psychologist-listener`** (Sonnet) — reflective listening only, one question per turn, refuses diagnosis.
> - **`business-mentor`** (Sonnet) — strategic-decision worker; refuses to make the call for you.
>
> A third worker (`fintech-legal-analyst` on Opus) lives in the companion `fintech-legal-advisor` plugin — install it separately if you want it.

## Step 4 — Setup commands (one paragraph)

> Three slash commands. The first is the entry point you may already know; the other two wire the recurring rhythms into Cowork's Scheduled Tasks:
>
> - `/personal-coach:onboard` — start (or resume / re-run) the onboarding.
> - `/personal-coach:setup-morning-briefing` — daily briefing rhythm.
> - `/personal-coach:setup-decision-grading` — weekly grading scan for past-deadline decisions.
>
> Companion plugins add more: `/news-digest:setup-news-digest` (from the `news-digest` plugin) for the daily digest rhythm; `/fintech-legal-advisor:setup-legal-triage-routine` (from `fintech-legal-advisor`) for the Cowork Routine that triages new Drive contracts.

## Step 5 — Cowork Routines (one paragraph)

> Two optional Routine templates live at `docs/personal-coach-routines/` in the basic-harness repo (not inside the plugin) — `morning-briefing-routine.md` and `decision-grading-routine.md`. Routines run in Anthropic's cloud with the laptop closed. **Reflection and profile edits are deliberately not routinable** because reflection content is the most sensitive material in the plugin and shouldn't transit cloud. Read `docs/personal-coach-routines/README.md` before installing — the privacy posture matters. The news-digest and legal-triage Routine templates now live with their respective companion plugins.

## Step 6 — Where files live

> Everything stays on your machine:
>
> - With a personal vault: `<vault>/Profile`, `<vault>/Journal`, `<vault>/Decisions`, `<vault>/Goals`, `<vault>/Business`, `<vault>/People`, `<vault>/Reading`. The companion plugins, when installed, also write to `<vault>/Legal` and `<vault>/News` under the same vault.
> - Without a vault: `~/.claude/personal-profile.md`, `~/.claude/journal/`, `~/.claude/decisions/`. Companion plugins use `~/.claude/legal-triage/` and `~/.claude/news/` when no vault is configured.
>
> If you set up the vault later (`/vault-librarian:bootstrap-vault` with the personal schema), the files don't auto-migrate — you'd move them yourself. The plugin works the same either way.

## Step 7 — Hard limits (always — never skipped)

> A few things this plugin will not do, by design:
>
> - **No medical or psychological diagnosis.** The reflective listener mirrors and asks; never assesses.
> - **No silent profile edits.** Every write is shown and confirmed.
> - **No surrogate decisions.** The business-mentor sharpens your call; it doesn't make it.
>
> (If you install the `fintech-legal-advisor` companion plugin, it adds: **no legal advice** — issue lists only, with a take-to-counsel block on every output.)
>
> If you ever want to push past these, the plugin will tell you so explicitly and stop. That's a feature, not a flaw — it's how the plugin is safe to use as a long-running companion.

## Step 8 — Closing

> That's the tour. A few starting points, depending on what's most useful right now:
>
> - Just talk and the right skill will pick up — you don't need to memorize names.
> - To set up the daily rhythm: `/personal-coach:setup-morning-briefing`.
> - For a daily news digest tailored to topics you care about, install the `news-digest` companion plugin and run `/news-digest:setup-news-digest`.
> - For fintech legal triage (interactive or via a Cowork Routine watching a Drive folder), install the `fintech-legal-advisor` companion plugin.
> - For the wider basic-harness map (other plugins beyond personal-coach), see `/welcome:tour`.
>
> Done. Take care.

Stop. Do not chain.

## What this command will NOT do

- **Will not write to disk.** Pure read-only orientation.
- **Will not invoke any of the named skills or commands.** The user runs them when they're ready.
- **Will not retry onboarding.** If the user wants to onboard, they type `/personal-coach:onboard` — we don't push.
- **Will not run more than ~2 minutes.** Use Steps 5 and 6 as expansion points if the user asks; otherwise keep it tight.

## Failure modes

- **User asks "tell me more about X"** for a specific skill — answer in 2-3 sentences and offer the SKILL.md path on disk for the curious. Do not paste the whole skill body.
- **User wants to start using something mid-tour** — break out of the tour, hand off to that thing. The tour is not so important that we'd block real work for it.
