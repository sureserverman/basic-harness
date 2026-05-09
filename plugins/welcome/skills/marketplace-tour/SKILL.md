---
name: marketplace-tour
description: Use to give a calm, value-first opening for the basic-harness marketplace — one short message that names three concrete user moments (decide / reflect / fintech triage) and invites the user into theirs, in any language. Triggers on first-time-user-style natural language — "show me what you do", "what can you do", "what is this", "introduce yourself", "give me a tour", "where do I start", "что ты умеешь", "с чего начать", "qué haces", "preséntate", "was kannst du", "qu'est-ce que tu fais", and equivalents in any language. Read-only; never writes; never auto-chains.
---

# Marketplace Tour (value-first opening)

A short opening for someone arriving cold. Same flow whether the user typed `/welcome:tour` or said something equivalent in any language. **Read-only.** Do not write any files. Do not invoke any other skills. Do not auto-run another tour at the end.

The job of this skill is to invite the user into a real use case — not to enumerate the marketplace. If the user genuinely wants the wider catalog, the depth tours (`/personal-coach:tour`, `/process-skills:tour`) exist for that.

<HARD-GATE>
This is the marketplace-level opening, not a plugin-specific one. If the user is clearly asking about a specific plugin ("show me what personal-coach does", "что умеет personal-coach"), prefer that plugin's tour or onboarding skill.
</HARD-GATE>

<HARD-GATE>
This skill never auto-chains into another skill. After the opening, the user picks a thing to try; invocation of the next skill happens only on explicit "yes" or on a real situation the user describes.
</HARD-GATE>

## Step 0 — Detect language (silent if clear)

Read the user's invocation message.

- **Clear language** (Russian / Spanish / German / French / Mandarin / Hindi / Arabic / Portuguese / Italian / Japanese / Korean / etc.) — proceed in that language. Do not ask.
- **Ambiguous** (English-only, mixed-script, single short phrase) — ask in three languages briefly:

  > English: What language would you like to work in?
  > Русский: На каком языке вам удобнее общаться?
  > Español: ¿En qué idioma prefieres que hablemos?

Add other languages if the system context suggests one.

## Step 1 — One opening message

In the user's language, send something equivalent to:

> Hi. I'm here as a thinking partner. People usually bring me one of three things: a decision they're stuck on, something on their mind they want to think through, or a fintech regulatory question they need triaged before talking to a lawyer. There's also a daily news digest tailored to topics you actually care about, and a small set of process skills (brainstorm, plan, execute, investigate) for research and project work.
>
> **What's on your mind right now?**

Three sentences plus one open question. No bullet list, no persona menu, no slash commands, no skill names. The point is to get the user talking about *their* situation in turn 2, not to brief them on the catalog.

## Step 2 — Branch on what the user says

Do not present a menu. React to what the user actually says.

| User says something like... | Do this |
|---|---|
| A real decision ("I'm trying to decide whether to...") | "That sounds like a `personal-coach:business-mentoring` thing — I can frame it with you. Want to start? (yes / not now)". Hand off only on yes. |
| Something emotional or "on my mind" | "We can reflect on that together — `personal-coach:reflection-session` runs a CBT-style flow with a hard limit at therapy. Want to start? (yes / not now)" |
| A fintech regulatory question | "That's `personal-coach:fintech-legal-triage`. Issue-spotter, not legal advice — every output ends with a take-to-counsel block. Need to start with jurisdiction. Ready? (yes / not now)" |
| A research / writing / project-management ask | "That's the process-skills track — `/process-skills:tour` walks through it in two minutes. Want me to point you at the right skill, or run the tour?" |
| "I'm just looking" / "show me what's around" | Step 3 below — the calm one-line catalog, on request only. |
| Ambiguous / not clear which | One disambiguating question: "Are you mostly trying to **produce something** (report, article, deliverable), or mostly trying to **think through something** (decision, reflection, daily rhythm)?" The first answer points to process-skills; the second to personal-coach. |

**Do not start the next skill without an explicit "yes".** The user might just want to know the option exists.

## Step 3 — On-request: a one-line catalog

Only if the user asks "what else", "show me everything", "list what's around" — and only then. Equivalent of:

> The marketplace has five plugins:
>
> - **personal-coach** — decisions, reflection, fintech triage, daily news, profile that compounds across sessions.
> - **process-skills** — brainstorm, plan, execute, investigate, review (research / writing / project work).
> - **delegation-agents** — parallel workers (Haiku/Sonnet) for bulk reading, editing, drafting.
> - **vault-librarian** — set up a Markdown notes vault (one command).
> - **welcome** — this orientation, which is the map you're reading.
>
> The depth tours are `/personal-coach:tour` and `/process-skills:tour` — about two minutes each.

Stop. Don't push.

## Step 4 — Closing (always, calm)

Equivalent of:

> Whenever something comes up, just say it — the right plugin will pick up. No commands to memorize.

End. Do not chain.

## What this skill will NOT do

- **Will not write to disk.** No profile bootstrap, no journal entry, no preferences file.
- **Will not auto-invoke another skill.** The user must say yes — or describe a real situation that warrants the handoff.
- **Will not enumerate the catalog upfront.** The catalog appears only on Step 3, only on explicit request.
- **Will not promote one track over another.** Persona-relevant routing, neutral framing.
- **Will not run more than ~90 seconds.** If the user wants depth, point them at the depth tour.
- **Will not fire repeatedly in the same session.** If the user has already seen this opening, give a one-line "we covered this earlier — what's most useful right now?". Don't loop.

## Failure modes

- **User picks a track whose plugin isn't installed.** Tell them: "That track lives in `<plugin-name>` — not installed yet. Want the install line?". Do not auto-install.
- **User asks about `welcome` itself.** One sentence: "This plugin is just the opening you're reading — there isn't more to it." Loop back to the open question.
- **Multilingual mid-flow switch.** Match the new language; don't force consistency the user didn't ask for.
- **Skill fires on a question that was actually about a specific plugin.** Recognize and hand off: "That's specifically about personal-coach — want the personal-coach tour? `/personal-coach:tour`."

## Sources

- **Value-first opening** — Nielsen Norman Group, [*Mobile App Onboarding*](https://www.nngroup.com/articles/mobile-app-onboarding/) (Pernice & Budiu): "Avoid feature-promotion onboarding at first launch. Users rarely download an app for no reason; therefore, lengthy promotional onboarding will likely be skipped."
- **Three example moments instead of a feature list** — pattern observed in ChatGPT and Perplexity first-run UX (empty composer + three example prompts as invitation), summarized in [DemoKraft's *Conversational Onboarding* guide](https://demokraft.ai/conversational-onboarding-beginners-guide/): "What would you like to do today?" as the canonical opener.
- **Catalog on request, not upfront** — Nielsen Norman Group, [*Onboarding Tutorials vs. Contextual Help*](https://www.nngroup.com/articles/onboarding-tutorials/): "Highlight features while the user is in the app, through contextual help" — pull-revelation, not push-walkthrough.
- **Two disclosure levels max** — Nielsen, [*Progressive Disclosure*](https://www.nngroup.com/articles/progressive-disclosure/) (NN/g, 2006, updated 2024).
- **Benefit copy beats feature copy** — Yifrah, *Microcopy: The Complete Guide* (2nd ed., 2019): "Upgrade your productivity" beats "Upgrade plan."
- **No auto-chain** — Nielsen, *10 Usability Heuristics* (1994), heuristic 3 (User control and freedom).
- **Activation = first useful turn** — Sean Ellis, *Hacking Growth* (2017); summarized in [Amplitude's *aha moment*](https://amplitude.com/blog/aha-moment): "the moment that the utility of the product really clicks for the users."
