---
name: marketplace-tour
description: Use to give a calm two-minute read-only map of the basic-harness marketplace — what it is, the four personas it's built for, the install flow, pointers to the two specialized tours. Triggers on first-time-user-style natural language in any language — "show me what you do", "what can you do", "what is this", "introduce yourself", "give me a tour", "where do I start", "что ты умеешь", "с чего начать", "qué haces", "preséntate", "was kannst du", "qu'est-ce que tu fais", and equivalents in any language not listed. Treat these as trigger semantics, not literal matching — the full trigger list is in the body. Read-only; never writes files; never auto-chains into a specialized tour without explicit user "yes". Same skill fires when the user types /welcome:tour.
---

# Marketplace Tour

A two-minute orientation. Same flow whether the user typed `/welcome:tour` or said something like "show me what you do" in any language. **Read-only.** Do not write any files. Do not invoke any other skills. Do not auto-run another tour at the end.

**Announce at start, in the user's language:** something equivalent to "Sure — let me give you a quick map. About two minutes, no setup required."

<HARD-GATE>
This is the marketplace-level orientation, not a plugin-specific one. If the user is clearly asking about a specific plugin ("show me what personal-coach does", "что умеет personal-coach"), prefer that plugin's tour or onboarding skill. Use this skill when the question is at the basic-harness level.
</HARD-GATE>

<HARD-GATE>
This skill never auto-chains into a specialized tour. After Step 2 the user picks a persona, and you offer the relevant specialized tour — but invocation happens only on explicit "yes". A skill that auto-runs another skill at the end of itself is the antipattern this hard-gate exists to prevent.
</HARD-GATE>

## Triggers (full list, for language detection)

The skill fires on first-time-user-style questions in any language. Below is a non-exhaustive list — Claude generalizes to phrasings not on it. Use these as **trigger semantics**, not literal matching.

| Language | Phrases |
|---|---|
| English | show me what you do · what can you do · what do you do · what is this · what's around · what's available · introduce yourself · show me around · give me a tour · where do I start · how do I use this · what's basic-harness |
| Russian | что ты умеешь · что ты можешь делать · покажи что у тебя есть · проведи экскурсию · что это вообще · с чего начать · расскажи о себе |
| Spanish | qué haces · qué puedes hacer · muéstrame qué tienes · preséntate · por dónde empiezo |
| German | was kannst du · zeig mir was du machst · stell dich vor |
| French | qu'est-ce que tu fais · montre-moi · présente-toi · par où commencer |
| Mandarin | 你能做什么 · 介绍一下 |
| Hindi | तुम क्या कर सकते हो |
| Arabic | ماذا تستطيع أن تفعل |

If the user writes in a language not on this list, the skill still fires — the description's trigger semantics are language-agnostic for "first-time-user orientation question". Run the flow in whatever language the user wrote in.

## Step 0 — Detect language

Read the user's invocation message. If the language is clear, run in that language. If ambiguous (English-only message; mixed-script; single short phrase), ask in three languages:

> English: What language would you like to work in?
> Русский: На каком языке вам удобнее общаться?
> Español: ¿En qué idioma prefieres que hablemos?

Add other languages if the system context suggests one. Persist the preference in working memory for the rest of the tour.

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

- **1, 2, or 3** → "The right tour for you is `/process-skills:tour` — covers process-skills, delegation-agents, and the optional vault stack. About two minutes. Want me to start it now? (yes / not now)"
- **4** → "The right tour is `/personal-coach:tour`. Want me to start it now? (yes / not now)" — and also mention `/personal-coach:onboard` if the user wants to actually start using personal-coach today.
- **"More than one"** → run a 60-second summary of `process-skills` purpose followed by a 60-second summary of `personal-coach` purpose, then ask which to dive into first.
- **"I'm not sure"** → ask one disambiguating question: "Are you mostly trying to **produce something** (a report, an article, a project deliverable), or mostly trying to **think with someone** (a decision, a reflection, a daily rhythm)?". The first answer points to process-skills; the second to personal-coach.

**Do not start the specialized tour without an explicit "yes".** The user might want to read this map and stop here.

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

## What this skill will NOT do

- **Will not write to disk.** No profile bootstrap, no journal entry, no preferences file. Other tours and `/personal-coach:onboard` do that.
- **Will not auto-invoke a specialized tour.** The user must say yes.
- **Will not enumerate every skill in every plugin.** That's the specialized tours' job. Two disclosure levels max — same NNG principle as the onboarding skill.
- **Will not promote `personal-coach` over `process-skills` or vice versa.** Persona-driven branching, neutral framing.
- **Will not run more than ~2 minutes.** If the user asks lots of questions, answer them, but don't expand the tour into a deep tutorial. Point them at the specialized tour for depth.
- **Will not fire repeatedly in the same session.** If the user has already been through this skill (the language is set, they've picked a persona, they've heard Step 4), and they ask again, give a one-line "we already did the tour earlier — want me to run it again from the top?". Don't loop.

## Failure modes

- **User picks a persona that isn't installed yet.** The persona is fine; the tour they're being pointed to may not exist as an installed command. Tell them: "That tour lives in the `<plugin-name>` plugin, which isn't installed yet. Want the install line?". Do not auto-install.
- **User picks `welcome` itself ("tell me about welcome plugin")** — answer in one sentence: "This plugin is just the map you're reading. There isn't more to it." Loop back to Step 2.
- **Multilingual mid-flow switch** — match the new language; don't force consistency the user didn't ask for.
- **Skill fires on a question that was actually about a specific plugin** — e.g., user asks "what does personal-coach do" and this skill fires anyway because the question matched the description. Recognize the specificity, hand off: "That's specifically about personal-coach — want the personal-coach tour? `/personal-coach:tour`."

## Sources

- **Two disclosure levels max** — Nielsen Norman Group, *Progressive Disclosure* (2006, updated 2024). Same principle as the personal-coach onboarding skill.
- **Persona-driven branching** — Cooper, *About Face* (2014). Personas are decision-architecture; not every user is the same and the tour acknowledges that.
- **Neutral framing** — Yifrah, *Microcopy: The Complete Guide* (2017). No "you'll love this", no "this is amazing" — just what's there.
- **No auto-chain** — Nielsen, *10 Usability Heuristics* (1994), heuristic 3 (User control and freedom). Auto-running another tour at the end traps the user; explicit "yes" preserves agency.
