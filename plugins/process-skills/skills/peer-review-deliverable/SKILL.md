---
name: peer-review-deliverable
description: Use when reviewing a deliverable in flight — a draft chapter, a proposal section, a brief, an interim report, an updated SOP — and what matters is what's NEW or CHANGED since a baseline. Triggers on "review this draft before I send it", "what changed since the last version", "review my chapter against the outline", "check this brief against the plan", "is this ready to circulate". Scopes the change set, runs an opinionated review against it, and surfaces a Critical / Important / Suggestion triage. For a holistic single-document review, use review-document instead.
---

# Peer Review of a Deliverable

Review a deliverable focused on **what's new or changed against a baseline** — not the document as a whole. The baseline can be the prior version, a parent plan, a template the deliverable should conform to, or "blank" for a brand-new piece reviewed for the first time.

This is the analog of a code review for any reviewable artifact. For a holistic structure / brevity / clarity pass on a single document, use `review-document` instead.

**Announce at start:** "Using the peer-review-deliverable skill to review `<scope>`."

## Scope selection

Ask, or infer from the request, which slice to review:

| Scope | What it means | Baseline |
|---|---|---|
| Working draft | Uncommitted / unsaved changes since the last save | Last saved version |
| Sent-for-comment | This version vs. the version that went out for comment | The sent version |
| Vs. prior version | Current draft vs. the prior version on disk | The prior file (e.g. `*-v2.md` vs. `*-v1.md`) |
| Vs. template | Current draft vs. the template / boilerplate it should follow | The template file |
| Vs. plan | Current draft vs. the plan / outline / brief that authorized it | The plan or outline |
| Vs. branch base | Git: this branch vs. main | `git diff <base>...HEAD` |
| Whole-document | No baseline — first review | (none — but consider `review-document` instead) |

If the scope is ambiguous, ask. The right baseline changes what counts as a finding: a sentence that's fine in isolation may be a problem in a v2 if it contradicts v1.

## Workflow

1. **Collect scope.** Build the change set: the new file(s), the baseline (path or "(new)"), and a 1–3 sentence statement of intent (what the change is meant to accomplish).
2. **Check change-set size.** If the change set is huge (e.g. >2000 lines of net new content, or >40% of a long document rewritten), warn the user: large reviews produce shallow findings. Offer to split — by section, by chapter, by stage of a `planning-projects` plan.
3. **Redact secrets.** Scan the new content for obvious credentials, embargoed information, on-background quotes that have been re-attributed without the source's consent, or other content that should not be circulated. If present, flag to the user *before* doing any external dispatch.
4. **Assemble the brief.** Even for an in-skill review, articulate what you're checking against. Include: the change-set itself, the baseline, the stated intent, and any constraints (deadline, audience, length limit, embargo).
5. **Run the review.** Walk the protocols below in order. Each protocol produces zero or more findings.
6. **Surface the triage.** Present the Critical / Important / Suggestion table with file:line (or section:paragraph) citations. Do not re-rank or paraphrase severity.
7. **Offer next steps.** For each Critical or Important, offer to open the location, propose a fix, or defer to the user. Do NOT auto-edit without confirmation — a peer review is informational by default.

## Review protocols

Run these in order on the change set. Each is a lens; together they cover the common failure modes.

### 1. Intent alignment

Does the change actually do what the stated intent says? If the intent is "add evidence to support claim X" and the change adds prose without new sources, that's a Critical. The most common kind of bad deliverable is a clean-looking one that solves the wrong problem.

### 2. Baseline consistency

Read the baseline. Does the new content contradict, repeat, or undermine what's already there? Three failure modes:

- **Contradiction.** v2 claims X but v1 said not-X, with no acknowledgement of the change.
- **Silent overwrite.** v2 drops a section v1 had, with no note of why or where it went.
- **Drift.** Terminology, voice, or structure has slipped between versions in a way the reader will notice.

### 3. Structure and signposting

Walk the change-set's headings. New sections should be at the right depth, in the right order, with the right labels. A section called "Methods" that contains results is worse than no Methods section at all.

### 4. Claim integrity

Every concrete claim added in this change has either:

- A cited source the reader can reach, or
- A clear "this is the author's judgment" framing, or
- A flag that it needs a source still.

Bare claims with no support are the easiest finding to make and the most-often-skipped review pass.

### 5. Audience fit

Read the change as the document's actual audience. Two failure modes:

- **Too much for the audience.** Jargon they don't share; assumptions about background they don't have.
- **Not enough for the audience.** Conclusions that need a chain of reasoning the audience expects.

### 6. Style and consistency

Match the surrounding document. Voice (we / you / passive), tense, list-marker style, citation format, heading capitalization. Stylistic drift inside a single document is a reader-trust problem.

### 7. Risk and constraints

Does the change introduce risk the rest of the document doesn't already account for? Embargo problems, attribution problems, factual claims that go beyond the evidence, commitments the author can't deliver on, scope creep against the plan. Flag these as Critical even when small — they're the kind that get retracted later.

## Severity rubric

- **Critical** — the deliverable cannot ship in this state. Wrong intent, contradiction with baseline, unsupported substantive claim, embargo or attribution problem, factual error, scope-creep that breaks the plan.
- **Important** — the deliverable can ship but the reader will notice and the author will regret it. Drift, missing signposting, audience-fit problem.
- **Suggestion** — non-blocking improvement. Style consistency, an easier-to-follow ordering, a better example.

Present:

```
| # | Sev        | Protocol         | Location           | Finding                                             |
|---|------------|------------------|--------------------|-----------------------------------------------------|
| 1 | CRITICAL   | Intent alignment | §2 ¶3              | Section claims to support X but the new evidence is for Y |
| 2 | IMPORTANT  | Baseline drift   | §4 vs v1 §4        | Terminology shifted from "subject" to "respondent" without note |
| 3 | SUGGESTION | Style            | §3 throughout      | Voice switches from "we" to passive twice            |
```

## When the change-set is whole-document

If there is no baseline (genuinely first review), most protocols still apply but the bar shifts. Without a baseline you can't catch contradictions or drift; you can still catch intent misalignment, structure problems, claim integrity, audience fit, and risk. If the user really wants a holistic review, point them to `review-document` — that skill is built for the no-baseline case.

## Chaining with other skills

- **Before sending / publishing**: run on the working draft, fix Criticals, then send.
- **Between plan stages**: `executing-plans` can name this skill as the optional stage-gate review of in-flight deliverables.
- **Second opinion**: after this review, optionally run an external-review tool (if available) on the same change-set for cross-reviewer agreement.
- **Routing structural rewrites**: an Important / Critical finding that requires significant rewriting can be handed to the `document-rewriter` agent (Sonnet, in the `delegation-agents` plugin) with the specific finding as the spec. This skill produces the spec; `document-rewriter` applies it.

## When NOT to use this skill

- For a holistic single-document review with no baseline — use `review-document`.
- For deciding *whether* a deliverable should exist at all — that's a brainstorming question, use `brainstorming`.
- For ingesting a finished source into a vault — use the upstream `obsidian-wiki:ingest`.
- For investigating *why* a deliverable went wrong after the fact — use `evidence-first-investigation`.

## Reference

- **Fagan inspection** — Michael Fagan, "Design and code inspections to reduce errors in program development" (IBM Systems Journal, 1976). The canonical multi-reviewer defect-detection study; the protocol-based review pattern generalizes from code to any reviewable artifact.
- **Google Code Review Developer Guide** — https://google.github.io/eng-practices/review/ — the source of the "review what changed against intent" framing this skill borrows.
- **Verification Handbook** (European Journalism Centre, 2014) — the source-claim-evidence discipline used in the Claim Integrity protocol.
- **Editorial style consistency** — *The Chicago Manual of Style*, Ch. 2 ("Manuscript Preparation, Manuscript Editing, and Proofreading") on consistent voice, tense, and terminology across a single piece.
