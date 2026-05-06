# personal-coach

Personal-use plugin for `basic-harness`. Turns Claude Code into a structured companion for the parts of life that aren't shipping software:

- **Personal assistant that actually knows you** — a persistent profile that compounds across sessions instead of starting from zero every chat.
- **Reflective listener** — guided journaling and CBT-style reframing for personal development. Not therapy; a structured thinking partner with explicit limits.
- **Business mentor** — strategic-decision frameworks (OKR, Jobs-to-be-Done, Cynefin, decision journals) to think through hard calls before you commit.
- **Fintech legal-issue spotter** — surfaces the regulatory questions you should be asking your real lawyer about KYC/AML, payment licensing, data protection, and customer contracts. Issue-spotter, not advice.

This plugin is opinionated about scope. It will refuse to play "diagnose me" or "tell me what to do legally" — those are jobs for a licensed therapist and a licensed lawyer. It is good at being the thing in between: a structured, citation-grounded thinking partner that helps you arrive prepared.

## Install

```text
/plugin install personal-coach@basic-harness
```

Pairs especially well with the optional vault stack — see the top-level basic-harness README — because the **personal** vault schema gives Profile/Journal/Goals/Business/Legal/People/Decisions a real home on disk.

## Skills

All five skills follow the same shape as `process-skills`: a checklist, phased prompts, a hard handoff at the end, and primary-source citations for the methodology.

| Skill | Purpose |
|---|---|
| `personal-profile` | Build and maintain a persistent profile of the user — values, goals, voice, history, recurring stakeholders. The substrate the other skills read from. |
| `reflection-session` | Structured journaling and CBT-style reflection. Emotion labelling → thought record → cognitive reframe → committed action. |
| `business-mentoring` | Frame a strategic decision: pick a framework, work it, write a decision-journal entry that you can grade later. |
| `fintech-legal-triage` | Issue-spotter for fintech work: KYC/AML, PSD2, MiCA, GDPR, data localisation, customer-contract gotchas. Produces a question list for a real lawyer. |
| `morning-briefing` | Daily standup with yourself: yesterday's commits, today's intentions, blockers, one reflection prompt. |

## Subagents

Model-pinned worker agents you can dispatch directly when the corresponding skill is overkill:

| Agent | Model | Role |
|---|---|---|
| `psychologist-listener` | Sonnet | Reflective listening only — mirrors, names emotions, asks one question at a time. Refuses diagnosis. |
| `business-mentor` | Sonnet | Strategic-decision sparring partner. Pushes back on premises. Names the framework being applied. |
| `fintech-legal-analyst` | Opus | Careful regulatory analysis. Asks for jurisdiction first. Outputs an issue list, never a verdict. |

## Hard limits

- **No medical, psychiatric, or psychological diagnosis.** Reflective listening, never assessment. Crisis content triggers a stop-and-redirect to local emergency lines.
- **No legal advice.** Issue-spotting and question-formulation only. Every fintech-legal output ends with "take this to a licensed lawyer in <jurisdiction>".
- **No silent edits to the personal profile.** The user sees and confirms every profile write.
- **No data exfiltration.** Profile and journal entries stay on disk under the user's vault path. Nothing is sent anywhere.

## Sources and rationale

The skills cite their methodology so the shape isn't arbitrary. See each `SKILL.md`'s "Sources" section. Anchor references:

- CBT thought records — Aaron T. Beck, *Cognitive Therapy of Depression* (1979); Judith Beck, *Cognitive Behavior Therapy: Basics and Beyond* (3rd ed., 2020).
- Decision journals — Annie Duke, *Thinking in Bets* (2018); Daniel Kahneman, *Thinking, Fast and Slow* (2011).
- Jobs-to-be-Done — Clayton Christensen, *Competing Against Luck* (2016).
- Cynefin framework — David Snowden & Mary Boone, "A Leader's Framework for Decision Making" (Harvard Business Review, 2007).
- Stoic journaling — Marcus Aurelius, *Meditations*; Ryan Holiday, *The Daily Stoic* (2016).
- AML/KYC fundamentals — FATF *International Standards on Combating Money Laundering* (40 Recommendations, latest revision).
- EU payments and crypto — PSD2 (Directive (EU) 2015/2366); MiCA (Regulation (EU) 2023/1114).
- Data protection — GDPR (Regulation (EU) 2016/679).
