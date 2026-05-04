---
name: review-document
description: Use when reviewing, auditing, or improving any reader-facing document — a README, a one-page brief, an executive summary, a project memo, a vault page body, a proposal, a chapter introduction. Triggers on "this is too long", "the brief is a mess", "clean up this memo", "make this readable", "what's wrong with this page", or a user sharing a document and asking for a critique. Holds the line on the 30-second rule: a reader must understand what this is and what to do about it within 30 seconds.
---

# Review Document

Analyze and improve a reader-facing document so it is useful, brief, and easy to navigate.

## Core Principle: The 30-Second Rule

A reader must understand **what this is** and **what they should do with it** within 30 seconds of opening the file. Everything else is secondary. When in doubt, cut.

This applies whether the document is a README, a brief, a memo, a one-pager, a chapter introduction, or any other reader-facing piece. The skill is named "review-document" not "review-readme" on purpose.

## Instructions

1. Read the document and scan its surroundings (sibling documents in the same folder, any template the project uses, the prior version if one exists).
2. Run all checks below, classifying findings as **ISSUE** / **IMPROVE** / **STYLE**.
3. Present the findings table, ask the user which to apply, then make the changes.

## Structure check

**Target: ≤6 top-level sections** for any single document. Evaluate against the essential skeleton:

| Section | Rule |
|---------|------|
| **Title + one-liner** | ≤15 words answering "What is this and who is it for?" directly under the title |
| **Quick orientation** | What the reader should do first, in **≤3 lines or steps**. The most important section after the title |
| **Body** | The core content — primary use case, primary argument, primary findings, with one concrete example |
| **Closing / next** | What to do after reading: where to go next, who to contact, what's outstanding |

Optional sections — add **only** when clearly needed:

| Section | When |
|---------|------|
| Background | Substantive context the reader actually needs to act. Not "history of the project". |
| Detailed sub-sections | The body has 3+ distinct facets a reader will want to skim |
| References / sources | The document makes specific claims that the reader may want to verify |
| FAQ / common questions | The same questions come up repeatedly from real readers |

**Anti-patterns:** table of contents in a sub-200-line document, more than 6 top-level sections, a separate "About" or "Introduction" when the one-liner already answers it, empty or boilerplate sections, backstory ("The team started this in 2024..."), throat-clearing ("This document aims to..."), nested sub-sub-sections that no reader follows.

## Content audit

### Accuracy

- [ ] Every concrete claim is verifiable — a cited source, a linked file, a specific data point with a date
- [ ] Names, titles, dates, file paths, URLs, version numbers match reality
- [ ] Prerequisites are current; terminology is consistent across the document
- [ ] If the document references commands, scripts, or external tools, those still exist and behave as described

### Brevity — cut before adding

**This is the discipline that matters most.** The natural tendency is to expand. Resist it.

- [ ] One-liner ≤15 words — no filler ("designed to be", "aims to provide", "is meant to help")
- [ ] Quick orientation ≤3 lines or steps. If it takes more, the skill is failing
- [ ] No paragraph longer than 3 sentences; no section longer than ~40 lines
- [ ] Cut filler phrases: "This section describes...", "It is worth noting that...", "As mentioned above..."
- [ ] Single primary path through the document; alternatives in collapsibles or moved to siblings
- [ ] Examples: minimum needed to show the shape — not every variation
- [ ] Reference material: 1–2 illustrative entries; link to the full reference

### Completeness — flag only if genuinely missing

- [ ] Answers: What is this? Who is it for? What should I do with it?
- [ ] If the document promises action ("here's how to apply"), the steps are present and usable
- [ ] If the document makes a claim that requires evidence ("we found X"), the evidence or its source is reachable

## Visual / readability check

- [ ] **Above the fold** (first ~25 lines): title, one-liner, quick orientation — nothing else
- [ ] One H1 in the file; no skipped heading levels (H1 → H3)
- [ ] Long optional content folded into collapsibles (`<details><summary>`) or moved to siblings
- [ ] Walls of text broken with bullets, tables, or short examples
- [ ] Consistent voice and tense throughout — no abrupt switches between "we", "you", and passive

## Report and apply

Present findings:

```
| # | Sev     | Phase     | Finding                                                  |
|---|---------|-----------|----------------------------------------------------------|
| 1 | ISSUE   | Structure | Missing one-liner — title is bare; reader can't orient   |
| 2 | IMPROVE | Brevity   | Background section is 60 lines — most can move to siblings|
| 3 | STYLE   | Visual    | Three abrupt voice switches between "we" and "you"        |
```

Use AskUserQuestion with options: "All", "ISSUE + IMPROVE only", "Let me pick". After applying, show a before / after summary.

**Key constraint: never grow the document to fix it.** If applying an edit adds content, cut or collapse matching content elsewhere in the same edit. The goal is *shorter and better*, not longer and more complete.

## Delegation (Claude Code only)

> **Skip this section unless you are Claude Code.** The Agent tool with `subagent_type:` parameters is a Claude Code feature. Other hosts do not have it — run the full workflow yourself instead.

Two phases can move off Opus.

**Evidence phase (to Haiku).** The scan pass — reading the document, walking the surrounding folder, checking that cited links / paths / sources exist, counting words and lines per section — is pure read work. Delegate to the **`bulk-reader`** subagent (model: haiku, ships in the `delegation-agents` plugin) via the Agent tool with `subagent_type: bulk-reader`. Ask it to return:

- Document structure: heading list with line ranges, word counts per section, link / URL targets with HTTP status.
- Surrounding evidence: which referenced files, sources, or commands actually exist (cross-check against the folder, the project's templates, or a prior version of the document).

**Rewrite phase (to Sonnet).** After the user has picked which findings to apply (the AskUserQuestion step), the actual edits are a Sonnet-tier rewrite job. Delegate to the **`document-rewriter`** subagent (model: sonnet, ships in the `delegation-agents` plugin) via the Agent tool with `subagent_type: document-rewriter`. Give it:

- the document path,
- the approved findings list (each with severity, section, specific instruction),
- the hard constraint: the document may only shrink — if an edit adds content, cut or collapse matching content elsewhere in the same edit.

Keep the ISSUE / IMPROVE / STYLE classification and the before / after summary in this session.

## When NOT to use this skill

- For peer review of a deliverable in flight (proposal, draft chapter, interim report) — use `peer-review-deliverable`. That skill is built around reviewing what's *new* against a baseline, not the document as a whole.
- For the actual ingestion and filing of a document into a vault — use the upstream `obsidian-wiki:ingest` skill.
- For deciding whether a document should exist at all — that's a brainstorming / planning question, use `brainstorming`.
