---
name: personal-profile
description: Use to build, update, or read the user's persistent personal profile — the substrate every other personal-coach skill depends on. Captures values, goals, voice, recurring stakeholders, working style, sensitivities, and explicit do-not-do entries. Triggers on "remember about me", "save this to my profile", "update my profile", "what do you know about me", "act as my personal assistant", "I want you to know me as", or whenever the assistant notices durable user information that future sessions will need. Never edits the profile silently — every write is shown and confirmed.
---

# Personal Profile

The substrate. Every other skill in `personal-coach` (`reflection-session`, `business-mentoring`, `fintech-legal-triage`, `morning-briefing`) reads from this file. Without it, each new session is amnesiac and the assistant role collapses into generic chat.

**Announce at start:** "Using the personal-profile skill to <read | update | initialize> your profile."

<HARD-GATE>
Never write or modify profile fields without showing the exact change to the user and getting an explicit "yes". Profile drift caused by silent edits is the failure mode this skill exists to prevent.
</HARD-GATE>

## Where the profile lives

Resolution order (use the first that exists):

1. **Vault path** — if a personal vault is bootstrapped (see `vault-librarian`'s `personal` schema), the file is `<vault>/Profile/profile.md`.
2. **Project path** — `.claude/personal-profile.md` at the project root, when the user wants the profile scoped to one engagement.
3. **Home path** — `~/.claude/personal-profile.md`, the cross-project default.

If none exists yet, ask the user which scope fits and create it there. Default to (3) for new users.

The file is **not committed to git automatically**. If the user keeps the profile in a project directory, remind them to add `.claude/personal-profile.md` to `.gitignore` unless they explicitly want it shared.

## Profile schema

Plain Markdown with named sections. Each section is optional but ordered. Do not invent new sections — if a fact doesn't fit, propose a section and confirm before adding.

```markdown
# Personal Profile — <user's preferred name>
Updated: <YYYY-MM-DD>

## Identity
- **Preferred name:** <how the assistant should address them>
- **Pronouns:** <if shared>
- **Location / timezone:** <city, IANA TZ>
- **Languages:** <primary, secondary — note which the user prefers for which contexts>

## Roles and responsibilities
<one bullet per current role: company, position, scope, who reports to them, who they report to>

## Values and operating principles
<3-7 bullets. The things the user has explicitly said matter. Each bullet should be defensible from a quote, not inferred from vibes.>

## Goals
- **12-month horizon:** <2-4 outcomes>
- **90-day horizon:** <2-4 outcomes>
- **This week:** <1-3 outcomes; refreshed by morning-briefing>

## Working style
- **Communication:** <terse / detailed; English / Russian / mixed; bullets / prose>
- **Decision-making:** <fast and reversible / slow and considered; data-led / intuition-led; consensus / sole>
- **Stress signals:** <what shows up when overloaded — overcommitting, late-night work, snapping at meetings>
- **Recovery patterns:** <what the user has said helps — sleep, walks, journaling, conversations with X>

## Stakeholders
<one bullet per recurring person: name, relationship, current state of the relationship if relevant>

## Business context
- **Company:** <name, sector, stage>
- **Domain:** <fintech sub-vertical — payments, lending, crypto, neobank, B2B SaaS for banks, etc.>
- **Jurisdictions:** <where the company operates — the legal-triage skill needs this>
- **Recurring decisions:** <2-3 topics the user keeps wrestling with>

## Sensitivities
<topics the user has flagged as off-limits, painful, or requiring care. The reflection-session skill checks this before opening hard topics.>

## Explicit do-not-do
<things the user has told the assistant to never do. Each entry has a "Why" line so future sessions can judge edge cases.>

## Artifacts
<paths to recurring documents the assistant should know exist — strategy doc, OKRs, decision journal, founder's notebook>

## Change log
- <YYYY-MM-DD> — <what changed, in one line>
```

## Phase 1 — Read before write

If the profile already exists, **read it first** and summarize back what's currently there in 4-6 lines before proposing any change. The user should be able to confirm the assistant has the right baseline before the assistant tries to extend it.

If the profile doesn't exist, ask the four bootstrap questions in order — one per message:

1. "What name should I use for you?"
2. "One sentence on what you do — your role and the company or context it lives in."
3. "What's a value or operating principle you want me to keep in mind across sessions? (just one for now — we can add more later)"
4. "What's the single most important goal on your plate this quarter?"

Write a minimal profile from those four answers. Everything else is added incrementally through real conversation, not a long onboarding form.

## Phase 2 — Capture during conversation

The user rarely says "save this to my profile" out loud. Watch for durable signal in normal conversation:

- **Identity facts** — name, role, company, location, languages.
- **Repeated preferences** — "I always do X", "I never do Y", "I want you to always Z".
- **Stakeholder mentions** with relationship context — "my co-founder Anna who handles ops".
- **Sensitivities** — "this is hard for me", "I don't want to talk about X right now", "let's not bring up the lawsuit".
- **Goals** stated as commitments, not idle wishes.

When you notice signal, **propose** the profile change. Format:

```
I'd like to add to your profile:

  Section: <Working style>
  Add bullet: "<exact text>"
  Reason: <one line — what the user said that justifies this>

Save? (yes / no / edit)
```

Wait for "yes" or an edit. Do not write to the file on "ok" or silence. If the user says "no", drop it without arguing — sometimes the user knows the signal was situational.

## Phase 3 — Write surgically

When the user approves an edit:

- Use the `Edit` tool, never `Write` — overwriting the whole file is how change-log discipline gets lost.
- Touch exactly the lines named in the proposal.
- Bump `Updated:` to today's date in `YYYY-MM-DD`.
- Append a one-line entry to `## Change log` with the date and what changed.

## Phase 4 — Read on demand

Other skills will ask the user (or this skill) to surface profile content. Common queries:

- "What does my profile say about my working style?" → quote the **Working style** section verbatim.
- "What do you know about my fintech context?" → quote the **Business context** section verbatim.
- "What are my sensitivities?" → quote the **Sensitivities** section verbatim. **Do this without summarizing or softening.** The exact wording matters because it was the user's wording.

If the file exists but a queried section is empty, say so explicitly — "your profile doesn't have a Sensitivities section yet" — rather than inferring from elsewhere.

## Phase 5 — Sensitivity and refusal

The profile is a high-trust object. Hard rules:

- **Never share profile contents with subagents unless the user has approved that delegation.** If `business-mentor` or `fintech-legal-analyst` is being dispatched, ask before passing the profile.
- **Never include profile content in commit messages, PR descriptions, issue bodies, or external messages.** It's local-only.
- **Treat the Sensitivities section as read-only outside this skill** — other skills may consult it but never edit it. Edits to Sensitivities require an explicit "update my sensitivities" from the user.
- **Diagnostic claims belong nowhere in this profile.** "User is anxious" is a diagnostic claim. "User has said anxiety shows up before board meetings" is a fact about something the user said.

## What this skill is NOT

- **Not a therapist's intake form.** Don't ask the bootstrap questions in clinical phrasing.
- **Not a CRM.** Stakeholder bullets stay short — one or two lines. If you find yourself writing meeting notes about a stakeholder, that belongs in the journal, not the profile.
- **Not a kitchen sink.** Profile bloat makes the assistant less useful, not more. Cap each section at the smallest amount the assistant needs to act in character. If a section grows past ~10 bullets, ask the user what to retire.

## Examples

### Adding a sensitivity mid-conversation

> User: "Honestly I don't want to keep workshopping the co-founder thing every session. It's exhausting."

Right reaction:

> Noted. I'd like to add this to your **Sensitivities** section so future sessions don't reopen it without you bringing it up first:
>
>   - **Co-founder relationship:** user has asked the assistant not to revisit this topic unless they raise it. Why: explicitly named as exhausting on 2026-05-06.
>
> Save? (yes / no / edit)

Wrong reaction: silently writing a "the user seems stressed about co-founder" entry, or pushing further into the topic.

### Refusing a write the user didn't authorize

> User: "I think I'm probably ADHD."

Right reaction: do not add a Health or Conditions section to the profile. Reflect back what the user said as a self-described observation, and only add to **Sensitivities** if the user explicitly asks ("write that down"). Diagnostic labels are not profile-grade facts.

## In Cowork (connector-aware enrichment)

When this skill runs in Claude Cowork and the user has granted connectors, it can enrich profile capture without changing the core flow. Connectors are **optional** — never gate the skill on them.

- **Google Calendar** — when the user mentions "I have a meeting with X tomorrow", verify the stakeholder's name against actual calendar events before adding to the **Stakeholders** section. Surfaces typos and disambiguates between people of the same first name.
- **Gmail** — when the user references a recurring correspondent, the skill may consult inbox metadata (sender, frequency) to suggest a Stakeholders entry. Always confirm with the user before writing — Gmail content does not flow into the profile silently.
- **Google Drive / Notion** — `Artifacts` section can link to canonical doc URLs the user names. Never index Drive at large; only the specific docs the user calls out.
- **Slack** — out of scope for personal-profile. The profile is a long-lived document; Slack data ages out and would create false claims about the user.

In Code (no connectors), all of the above happen via the user typing the answer. The output file is identical either way.

## Sources and rationale

- **Confirm-before-write** — basic version-control discipline; analogous to `git add -p` over `git add .`. Prevents silent drift.
- **Read-before-edit** — Beck's CBT principle of "collaborative empiricism" applied to memory artifacts: the patient (user) sees and corrects the therapist's (assistant's) understanding before it's used.
- **Identity-fact / preference / sensitivity / goal taxonomy** — adapted from Don Norman, *The Design of Everyday Things* (user model), and Alan Cooper, *About Face* (persona modeling), constrained to facts the user explicitly asserted rather than inferred traits.
- **Change-log discipline** — Architecture Decision Record (ADR) practice; Michael Nygard, "Documenting Architecture Decisions" (2011).
- **Sensitivities section** — adapted from trauma-informed-care intake practice (SAMHSA, *Trauma-Informed Approach* concept paper, 2014): record user-stated limits, not clinician-inferred ones.
