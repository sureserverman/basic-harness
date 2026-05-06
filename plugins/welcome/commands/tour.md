---
description: Two-minute calm read-only map of the basic-harness marketplace. Explains what basic-harness is, the four personas it's built for, the install flow, and points to the two specialized tours (/personal-coach:tour and /process-skills:tour). No artifacts written, no skills invoked. Runs in the user's preferred language.
---

# Tour

A two-minute orientation. The user types `/welcome:tour` and gets a clear, calm picture of the marketplace — enough to decide where to go next. **Read-only.** Do not write any files. Do not invoke any other skills. Do not auto-run another tour at the end.

## Step 0 — Detect language

Same pattern as `/personal-coach:onboard`. Read the user's invocation message; if the language is clear, run in that language. If ambiguous, ask in three languages briefly. Persist the preference in working memory.

## Step 1 — One paragraph: what basic-harness is

Equivalent of:

> `basic-harness` is a small marketplace of plugins for Claude Code and Claude Cowork. It's process discipline, knowledge management, and delegation patterns for structured work — not just coding. The plugins are designed to compose: you can use one in isolation, or stack them. Everything runs locally; nothing leaves your machine unless you explicitly enable a Cowork Routine, which you'd be told about up front.

Three sentences max. No feature list yet.

## Step 2 — Who it's for (the four personas)

> The marketplace is shaped around four personas. Pick the one that's closest:
>
> 1. **Researcher / analyst** — read sources, take notes, run multi-week investigations, write briefs.
> 2. **Writer / journalist** — long-form work with sources, drafting, editing, on-record discipline.
> 3. **Project lead / consultant** — engagements with deliverables, planning, weekly status updates.
> 4. **Personal user / fintech operator** — Claude as personal assistant, business mentor, reflective listener, and fintech legal-issue spotter.
>
> Which is closest? (1 / 2 / 3 / 4 / "I'm not sure" / "more than one")

Wait for the answer. Branches:

- **1, 2, or 3** → the user is in research / writing / project territory. Hand off to `/process-skills:tour`. Tell them: "The right tour for you is `/process-skills:tour` — covers process-skills, delegation-agents, and the optional vault stack. About two minutes. Want me to start it now? (yes / not now)"
- **4** → personal track. Hand off to `/personal-coach:tour`. Same pattern: "The right tour is `/personal-coach:tour`. Want me to start it now?"
- **"More than one"** → run a short version of both branches in sequence: 60-second summary of what `process-skills` is for, 60-second summary of what `personal-coach` is for, then ask which they want to dive into first.
- **"I'm not sure"** → ask one disambiguating question: "Are you mostly trying to **produce something** (a report, an article, a project deliverable), or mostly trying to **think with someone** (a decision, a reflection, a daily rhythm)?". The first answer points to process-skills; the second to personal-coach.

Do **not** start the specialized tour without an explicit "yes". The user might want to read this map and stop here.

## Step 3 — The install flow (only if asked)

If the user has not yet installed plugins beyond `welcome`, **and** asks about installation or seems unsure, give the canonical flow:

> ```
> /plugin marketplace add sureserverman/basic-harness
> /plugin install <plugin-name>@basic-harness
> ```
>
> The available plugin names are:
>
> - `process-skills` — six process skills.
> - `delegation-agents` — three subagents + a dispatching skill.
> - `vault-librarian` — optional, for a Markdown notes vault.
> - `personal-coach` — personal companion track (assistant / reflection / business mentor / fintech legal triage / news digest).
>
> Restart Claude Code (`/exit` and reopen) after installing, so the new skills register.

If the user hasn't asked, **don't volunteer this**. Step 3 is on-request.

## Step 4 — Closing (always)

Equivalent of:

> That's the map. Whenever you want to go deeper:
>
> - `/personal-coach:tour` — the personal companion track.
> - `/process-skills:tour` — the research / writing / project track.
> - `/personal-coach:onboard` — if you're personal-track and want to actually start using it (5 minutes, produces a real artifact).
>
> No need to remember commands — say what you want and the right one will come up.

End. Do not chain. The user comes back when they're ready.

## What this command will NOT do

- **Will not write to disk.** No profile bootstrap, no journal entry, no preferences file. Other tours and `/personal-coach:onboard` do that.
- **Will not auto-invoke a specialized tour.** The user must say yes.
- **Will not enumerate every skill in every plugin.** That's the specialized tours' job. Two disclosure levels max — same NNG principle as the onboarding skill.
- **Will not promote `personal-coach` over `process-skills` or vice versa.** Persona-driven branching, neutral framing.
- **Will not run more than ~2 minutes.** If the user is asking lots of questions, answer them, but don't expand the tour into a deep tutorial. Point them at the specialized tour for depth.

## Failure modes

- **User picks a persona that isn't installed yet.** The persona is fine; the tour they're being pointed to may not exist as an installed command. Tell them: "That tour lives in the `<plugin-name>` plugin, which isn't installed yet. Want the install line?". Do not auto-install.
- **User picks `welcome` itself ("tell me about welcome plugin")** — answer in one sentence: "This plugin is just the map you're reading. There isn't more to it." Loop back to Step 2.
- **Multilingual mid-flow switch** — match the new language, just like in `/personal-coach:onboard`.
