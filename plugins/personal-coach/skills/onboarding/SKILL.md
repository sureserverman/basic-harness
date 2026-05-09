---
name: onboarding
description: Use to onboard a new user to personal-coach with one short opening — three concrete user moments (decide / reflect / fintech triage), one open question, language-aware, time-to-first-value under 60 seconds. Does NOT walk through phases or list skills. Triggers on "onboard me", "set me up", "I'm new here", "where do I start", "help me get started", "introduce yourself", "what can you do", "познакомь меня", "помоги начать", "как начать", "comencemos", "empezar", "anfangen", "commençons", or whenever the user appears to be a first-time user. Never auto-fires; the user opts in.
---

# Onboarding (value-first)

The first 60 seconds for someone arriving at personal-coach. The job is **not** to enumerate features. The job is to invite the user into their own use case in one short message, and then run the matching skill on whatever they actually say — producing one tangible artifact in turn 2 or 3.

NN/g calls feature-promotion onboarding the antipattern that gets skipped. Sean Ellis calls activation "the moment the utility clicks for the user." Both point at the same thing: get out of the way and let the user try the product on a real situation of theirs.

<HARD-GATE>
This skill never auto-runs. It is invoked explicitly by the user (typically via `/personal-coach:onboard`) or in response to a clear first-time signal. If you're unsure whether the user is being onboarded or is mid-task, ask before starting.
</HARD-GATE>

<HARD-GATE>
The whole flow runs in the user's preferred language. The SKILL.md is in English, but every user-facing line — questions, confirmations, the wrap-up — is in their language. If the user switches mid-flow, switch with them.
</HARD-GATE>

## Operating principles

1. **Time to first value under 60 seconds.** The user is invited to bring a real situation; turn 2 starts engaging it.
2. **No feature list upfront.** No skill names, slash commands, or component types in the opening message. They appear only as quiet handoffs after the user has named a situation.
3. **No name or persona quiz at turn 1.** A name does not advance the user's goal (ProductLed: "click tax"). Ask only when it would actually help an answer.
4. **One question per message.** Plugin house style.
5. **Skip is always valid.** Every choice point includes a neutral "not now" option.
6. **Calm voice.** Short sentences. No emojis. No exclamation marks. No FOMO.
7. **Save with consent.** When something durable will be written (profile, vault, journal entry), say so before doing it.

---

## Step 0 — Detect language (silent if clear)

Read the user's invocation message.

- **High confidence detection** (clearly Russian / Spanish / German / French / Mandarin / Hindi / Arabic / Portuguese / Italian / Japanese / Korean / etc.) — proceed in that language. Do not ask.
- **Low confidence** — ask in three languages briefly (English / Russian / Spanish, plus contextual hints).
- **Save the preference** to working memory; persist to profile when (and if) the profile is created.

---

## Step 1 — One opening message

Send something equivalent to (in the user's language):

> Hi. I'm here as a thinking partner across sessions — quiet, calm, and structured around a few things people actually use me for: a decision they're stuck on, something on their mind they want to reflect on, or a fintech regulatory question they need triaged before talking to a lawyer. There's also a daily news digest tailored to what you actually care about.
>
> **What's on your mind right now?**

Three sentences plus one question. **No skill names. No slash commands. No phase numbering. No bullet list.** This is the only opening message; don't add a "here's what I can do" appendix.

If the user answers in a way that doesn't match any track ("I'm just looking", "show me what's around"), see Step 4. Otherwise jump to Step 2.

---

## Step 2 — Engage the actual situation (turn 2-3)

The user has named a situation. Don't switch them into a tutorial — engage it. Pick the matching lite track:

### Track A — Reflection (user said something emotional, "on my mind", "stuck", a hard situation)

Run a four-step compressed reflection:

1. "What's on your mind?" (one or two sentences)
2. "What's the feeling, in one or two words?"
3. "What's the thought running through your head about this — exact wording?"
4. "If a friend brought you this exact thought, what would you say to them?"

Mirror back the user's answer to question 4 as the reframe candidate. Then offer to save: "Want me to save this as your first journal entry? It would go to `<journal-path>/YYYY-MM-DD-reflection.md`. (yes / not now)". On yes → run the **vault-setup** sub-step (below) if no vault exists yet, then write the file.

The artifact is their first journal file on disk.

**Safety check applies.** If anything in the user's answers indicates crisis content, stop the lite flow and use the crisis redirect from `reflection-session`. Don't run a one-shot demo on something that needs the full skill.

### Track B — Decision (user said "I'm trying to decide", "should I", a real call they're wrestling with)

Run a five-question compressed decision frame:

1. "What's the decision, in one sentence with a verb?"
2. "How reversible is it — easy / hard / one-way?"
3. "When does it actually need to be made?"
4. "What's your current lean, and how confident, 0-100%?"
5. "What's one premise that has to be true for this to be the right call — and how confident are you in *that* premise?"

Summarize the frame and offer to save it as a decision-journal entry. Mention that the full `business-mentoring` skill goes deeper (framework selection, pre-mortem, second-order effects) but the lite frame is a real artifact they can build on.

### Track C — News digest (user said "give me daily news", "what's happening with X", "I want a digest")

Three questions:

1. "Three to five topics you want me to track. Topic names — examples: 'EU fintech regulation', 'AI policy', 'crypto enforcement', 'my company: <name>'."
2. "Anything you specifically don't want to see? Examples: 'no celebrity', 'no sports', 'no US partisan politics'."
3. "What time of day, what days, what timezone?"

Save to `news-preferences.md`. Then offer: "Want a sample one-topic digest right now? About a minute." On yes → run the **vault-setup** sub-step if needed, then invoke `news-digest` for a single-topic sample.

### Track D — Fintech regulation (user said "is this legal", "what regs", "do we need a license", "review this contract")

Run only Phase 0 of `fintech-legal-triage`:

1. "Where is the company incorporated?"
2. "Where do customers sit?"
3. "What's the activity?" (the eight-option list from the skill)

Show the matched cell — the regulation list and the issue-checklist categories — without running the full triage. Tell the user: "When you're ready to run the full triage on a real question, say so or invoke `fintech-legal-triage` directly. The matched cell for your context is `<jurisdiction × activity>`, which lives in regulations like `<3-5 named regs>`."

If a profile exists or is being created, save the (jurisdiction × activity) tuple to it.

### Track E — Multiple at once

If the user says they want more than one thing, pick the one most clearly stated in their first message. Tell them: "Let's start there — the others are one short message away whenever you're ready." Don't try to run all of them.

---

## Step 3 — Vault setup sub-step (only when first save happens)

Triggered the first time any track wants to save a real artifact. Do not run preemptively.

Read `~/.config/obsidian-wiki/config.json` (Linux/macOS) or `%APPDATA%\obsidian-wiki\config.json` (Windows). If the file is present and the vault path it points at exists on disk, use it — no prompt.

If missing, ask once, in line:

> I'd like to save this somewhere durable. Three options:
>
> 1. Set up a small notes folder at `~/Notes/personal-coach` (recommended).
> 2. I'll give you a different path.
> 3. Just save in `~/.claude/personal-coach/` — no vault, fewer features later.
>
> Which? (1 / 2 / 3)

Per choice:

- **(1)** → Write `~/Notes/personal-coach/CLAUDE.md` from `assets/vault-CLAUDE-personal.md`, `~/Notes/personal-coach/log.md` from `assets/log-template.md`, `~/Notes/personal-coach/Home.md` from `assets/home-template.md`, and `~/.config/obsidian-wiki/config.json` (or the Windows equivalent) pointing at the vault path. Show the file paths before writing; confirm with one "save?".
- **(2)** → Take the path. Do the same writes there.
- **(3)** → Skip vault setup entirely. Write the artifact to `~/.claude/personal-coach/<artifact>.md` instead.

Update `~/.claude/personal-coach.local.md` with the chosen path so subsequent skill invocations know where to write.

After vault setup, return to whatever track was running and complete the save.

---

## Step 4 — User said "just looking" or asked for the catalog

If the user opened with "show me what's around" / "what can you do" / "I'm just looking" rather than naming a situation, give one short answer — and end it with the same invitation back to a real use case:

> A few concrete things I'm useful for:
>
> - Working through a decision you're stuck on (a frame, a pre-mortem, a saved decision-journal entry to grade later).
> - Reflecting on something on your mind (CBT-style structure, a saved journal entry).
> - Triaging a fintech regulatory question (issue list for a real lawyer, never advice).
> - A daily news digest tailored to topics you actually care about.
> - A persistent profile that means you don't start from zero next session.
>
> Anything in particular pulling at you right now? Or want a wider read of the marketplace? (`/welcome:tour` covers the whole thing in about a minute.)

This is the **only** place a list appears in onboarding — and it's benefit copy (Yifrah), not feature copy. No skill names, no slash commands except the one pointer to `/welcome:tour`.

---

## Step 5 — Closing (after a track ran)

When a track has produced an artifact, close calmly:

> Saved at `<path>`. Whenever you want to come back to this — or to anything else — just say what's on your mind, and the right thing will pick up. No commands to remember.

If the artifact was a reflection or a decision frame and the user might want a daily rhythm:

> Some of this works on a daily rhythm if you want — a morning briefing or the news digest. Not now is fine; whenever you're ready, just say so.

After the close, **append a one-line entry** to `~/.claude/personal-coach.local.md`:

```markdown
## Onboarding history
- <YYYY-MM-DD> onboarded — language: <lang>, track: <A/B/C/D>, vault: <path-or-none>, saved: <yes/no>
```

This is what `/personal-coach:onboard` reads on subsequent runs to know whether the user is new or returning.

Stop. Do not chain. The user opens the next conversation when they're ready.

---

## Multilingual notes

- Whatever Phase 0 detected is the user-facing language. The SKILL.md is in English; you produce localized output.
- Region-grounded examples: when offering example topics in Track C, lean on what makes sense in the user's likely market. Tallinn → EU regulation; Singapore → MAS; São Paulo → Banco Central; etc.
- If the user code-switches mid-flow, match. Don't force consistency they didn't ask for.

---

## What this skill is NOT

- **Not a tutorial.** No phases, no five-step walkthrough.
- **Not a feature tour.** Skill names appear only as quiet handoffs once a track is engaged.
- **Not a config wizard.** Defaults are picked. Deeper config is deferred to the relevant skill.
- **Not a re-onboarding for returning users.** That branching lives in the `/personal-coach:onboard` command. This skill, when invoked, runs the first-time flow.
- **Not a sales pitch.** No "look how much you can do!". The plugin's value is in use, not in being impressive on a landing page.

---

## Failure modes

- **User abandons mid-flow.** Save state at every save point. Next session: read `personal-coach.local.md`, see partial state, ask "do you want to pick up from where we stopped, or start fresh?". Default to picking up.
- **User goes very fast / very slow.** Match their pace; don't push or stall.
- **Crisis content during Track A.** Use the `reflection-session` crisis redirect immediately. Stop the onboarding flow. Save what was captured so far. The user can resume later or never; both are fine.
- **User wants to do all four tracks.** Pick the most relevant for now; the others are one short message away.
- **No vault, no `~/.claude` write access.** Tell the user where files would have gone, capture preferences in working memory for the session, and tell them they can re-run onboarding once writability is sorted.
- **User declines to save anything.** Honor it. The flow still ran; nothing on disk. Some users want to look around before committing files.

---

## Sources

- **No feature promotion at first launch** — Nielsen Norman Group, [*Mobile App Onboarding*](https://www.nngroup.com/articles/mobile-app-onboarding/) (Pernice & Budiu).
- **Walkthroughs don't help; pull-revelations do** — Nielsen Norman Group, [*Onboarding Tutorials vs. Contextual Help*](https://www.nngroup.com/articles/onboarding-tutorials/).
- **Activation = first useful turn** — Sean Ellis, *Hacking Growth* (2017); operationalized in [Amplitude's *aha moment* writeup](https://amplitude.com/blog/aha-moment).
- **60-seconds-to-value as the new bar** — [ProductLed, *AI Onboarding in 60 seconds*](https://productled.com/blog/ai-onboarding) — also where the "click tax" framing comes from.
- **Two disclosure levels maximum** — Jakob Nielsen, [*Progressive Disclosure*](https://www.nngroup.com/articles/progressive-disclosure/) (NN/g, 2006, updated 2024).
- **Conversational opener pattern** — [DemoKraft's *Conversational Onboarding* guide](https://demokraft.ai/conversational-onboarding-beginners-guide/): "What would you like to do today?"; Notion / Miro have switched to conversational-first.
- **Learn-by-doing, not feature recital** — [Appcues, *Notion's Lightweight Onboarding*](https://goodux.appcues.com/blog/notions-lightweight-onboarding).
- **Benefit copy beats feature copy** — Kinneret Yifrah, *Microcopy: The Complete Guide* (2nd ed., 2019).
- **One question at a time** — Krug, *Don't Make Me Think* (3rd ed., 2014), Ch. 2.
- **Pre-fill defaults** — Thaler & Sunstein, *Nudge* (2008), Ch. 5.
- **Save-and-resume / skip is valid** — Nielsen, *10 Usability Heuristics* (1994, still current), heuristics 3 (user control and freedom) and 5 (error prevention).
- **Multilingual onboarding** — W3C *Internationalization Best Practices for Spec Developers* (2017): language preference captured early, persisted, not re-asked.
- **Crisis-content redirect** — same standard as `reflection-session`. Onboarding is not a "lite" version of the safety check; the safety check is the same.
