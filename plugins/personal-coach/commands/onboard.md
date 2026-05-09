---
description: Start (or restart) the personal-coach onboarding — a value-first, language-aware opening that asks what's on the user's mind right now and engages it directly (decision frame / reflection / fintech triage / news digest), with one tangible artifact in turn 2-3. Detects whether the user has been onboarded before and offers resume / re-run / cancel.
---

# Onboard

Explicit entry point for the onboarding skill. Invoked when the user types `/personal-coach:onboard` or asks to be onboarded. The command itself is short — it's a router that decides whether to run onboarding fresh, resume a partial onboarding, or skip onboarding because the user has already done it.

## Step 0 — Detect language

The user's invocation message is the first signal. If the user typed `/personal-coach:onboard` with no other context, the language is unclear; ask in three languages briefly (see the onboarding skill's Step 0 for the exact pattern). If the user wrote a sentence around the command ("онбордни меня", "set me up", "comencemos"), the language is detected.

Hold the language preference in working memory; the onboarding skill picks it up from there.

## Step 1 — Check onboarding history

Read `~/.claude/personal-coach.local.md` if it exists. Look for an `## Onboarding history` section.

Branches:

### A — File missing or no onboarding entry → first-time user

Invoke the **onboarding** skill directly. The skill's Step 1 is the value-first opening message — no preamble, no "five steps" framing. The user's first response sets the track; engagement happens in turn 2.

(Skill Step 0 is language detection, already done at the command level — pass the detected language through.)

### B — Prior complete onboarding entry exists → returning user

Tell the user:

> Looks like we already did onboarding on `<date>`. What would you like?
>
> 1. Add to my profile (some new context)
> 2. Re-run onboarding from scratch (replaces the saved state)
> 3. Set up something specific (news digest / morning briefing / decision-grading)
> 4. Just continue the conversation — no setup
>
> Pick a number, or tell me what you want.

Branches per answer:

- **(1)** → invoke the `personal-profile` skill in update mode.
- **(2)** → confirm explicitly: "Re-running onboarding will create a new profile draft and may overwrite parts of the existing one. Sure? (yes / no)". On yes, invoke the **onboarding** skill from Step 1.
- **(3)** → ask which of the three setup commands they want; invoke that command.
- **(4)** → exit cleanly with a one-line "ok, just say what's on your mind whenever."

### C — Partial onboarding entry exists (started but never finished) → resume

Tell the user:

> Looks like we started onboarding on `<date>` but didn't finish. Want to pick up from where we stopped, start fresh, or skip onboarding altogether?
>
> 1. Pick up from where we stopped
> 2. Start fresh
> 3. Skip — just say what you want

Branches per answer:

- **(1)** → read the partial state from disk (whatever was saved up to that point — profile draft, news preferences draft, etc.) and invoke **onboarding** at the appropriate step.
- **(2)** → invoke **onboarding** at Phase 1.
- **(3)** → exit cleanly.

## Step 2 — Hand off

Once the branch is decided, invoke the relevant skill or command. Do not proceed to do the onboarding work in this command — the command is a router; the work belongs to the skill.

## What this command will NOT do

- Will not silently run onboarding without an explicit user gesture. The user types the command, and even after that, gets a confirmation in branch A.
- Will not overwrite a complete profile without explicit "yes" in branch B.
- Will not ask deep questions — those are the onboarding skill's job. This command is short, gentle, and routing-only.
- Will not localize the entire skill output here — Phase 0 language detection is enough at the command level; the rest is handled inside the onboarding skill.
