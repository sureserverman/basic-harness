# Vault Schema (CLAUDE.md) — Personal

This file is the schema for the notes vault at `__VAULT_PATH__`. It tells the `obsidian-wiki` plugin how this vault is organized and how to operate on it.

This is the **personal** schema — the substrate that the `personal-coach` plugin reads from and writes to. It is shaped for one person using the vault as a long-running personal companion: a profile that compounds, journal entries from reflection sessions, decision-journal entries with grading, a legal-triage log, goals, recurring stakeholders.

This file is **read by every wiki operation** (ingest, query, lint) and is **edited only by the `vault-schema-maintain` skill**, never in ad-hoc sessions. Edit it surgically — do not rewrite the whole document. Each change is recorded in `log.md` with type `schema`.

---

## Vault structure

| Path | Purpose |
|---|---|
| `Profile/` | The user's persistent profile (`profile.md`). One file. Edited only via the `personal-profile` skill, never in ad-hoc sessions. |
| `Journal/` | Reflection-session entries (`YYYY-MM-DD-<slug>.md`) and morning-briefing entries. Append-only — old entries are not edited. |
| `Goals/` | One file per active goal or theme (`Goal — <name>.md`). When a goal is closed, prepend `done/` to the filename. |
| `Business/` | Per-company / per-engagement notes; one file per company or initiative. Where business-mentoring context lives between decisions. |
| `Decisions/` | Decision-journal entries from the `business-mentoring` skill (`YYYY-MM-DD-<slug>.md`). Status field tracks `pending → committed → in_review → graded`. |
| `Legal/` | Fintech-legal-triage outputs (`YYYY-MM-DD-<slug>.md`). One file per triage. Cumulative record of regulatory diligence over time. |
| `People/` | One file per recurring stakeholder. Working relationship state, what was last discussed, what they're working on. **Not** a CRM — minimal. |
| `Reading/` | Books, articles, podcasts that shaped the user's thinking. One file per source. |
| `Home.md` | Hand-curated Map-of-Content. |
| `log.md` | Append-only activity log. |
| `raw/` | Source inbox. Articles, PDFs, clipped pages, emails. **Immutable** — never edited. |
| `raw/assets/` | Images and binary assets referenced from `raw/`. |

The wiki layer is the eight category directories. Everything else is infrastructure.

---

## Page naming

- **Title Case with spaces** allowed and preferred for `Goals/`, `Business/`, `People/`, `Reading/`. Example: `Goal — Launch B2B Tier.md`.
- **Date-prefixed files** (`YYYY-MM-DD-<slug>.md`) for `Journal/`, `Decisions/`, `Legal/` because they're chronological by nature.
- **`profile.md` is the only file in `Profile/`** — the singular form is intentional. If a second profile is ever needed (e.g., personal vs founder identity), open the question with the user; do not create silently.
- **One topic per file.** If you find yourself wanting `X and Y.md`, you probably want two files plus cross-links.
- **No subdirectories inside category dirs**, except `Goals/done/` for closed goals.

---

## Frontmatter schema

The personal vault has more frontmatter variants than the generic schema because the file types differ in shape.

### Profile (`Profile/profile.md`)

```yaml
---
title: Personal Profile — <name>
type: profile
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
---
```

### Journal entry (reflection or briefing)

```yaml
---
date: <YYYY-MM-DD>
type: reflection | briefing
emotion: <one or two words; only for reflections>
distortions: [<list>; only for reflections]
action: <one line; only for reflections>
---
```

### Decision-journal entry

```yaml
---
date: <YYYY-MM-DD>
type: decision
status: pending | committed | in_review | graded
reversibility: easy | hard | one-way
deadline: <YYYY-MM-DD>
framework: <which one>
prior_lean: <A or B and confidence%>
---
```

### Legal-triage log

```yaml
---
date: <YYYY-MM-DD>
type: legal-triage
jurisdictions: [<list>]
activity: <category>
issue_count: <n>
high_confidence_issues: <n>
---
```

### Goal / Business / People / Reading

```yaml
---
title: <Page Title>
type: goal | business | person | reading
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
status: <active | paused | closed; for goals only>
tags: [<topic>, <subtopic>]
related: [[Other Page]]
---
```

`updated` is bumped on every edit by the editing skill (never manually).

---

## Category decision rules

When ingesting a source or filing new content, pick exactly one category. Tie-breakers in order:

1. **The user's own profile / identity / preferences** → `Profile/`. Only `personal-profile` writes here.
2. **A reflection or daily entry** → `Journal/`.
3. **A specific decision being made or already made** → `Decisions/`.
4. **A regulatory triage** → `Legal/`.
5. **A recurring stakeholder** → `People/`.
6. **A goal or theme that spans many sessions** → `Goals/`.
7. **A company-level note** → `Business/`.
8. **A book / article / podcast that shaped thinking** → `Reading/`.

If a source could fit two categories, prefer the more specific (e.g., a decision about a hire goes to `Decisions/`, not `People/`). If still tied, ask the user.

---

## Cross-reference rules

- A backlink is justified when the source contributes new information to the linked page. A pure name-drop is not.
- Decision entries should backlink to the affected `Goals/`, `Business/`, or `People/` pages.
- Reflection entries should backlink to a person in `People/` only when the user explicitly named them — never inferred.
- Legal-triage entries should backlink to the affected `Business/` company page.
- If a link would create a contradiction with existing content, **do not silently resolve it.** Add a `## Conflicts` section, ask the user.

---

## Sensitivity rules

This vault is highly personal. Hard rules that override normal vault behavior:

- **Profile and journal content never leaves the vault.** No commit messages, no PR bodies, no external messages. The vault path should be in `.gitignore` if the parent directory is a git repo.
- **`Profile/profile.md` Sensitivities section is read-only outside the `personal-profile` skill.** Other skills may consult it but never edit.
- **Reflection entries are never auto-summarized into other pages without the user's explicit ask.** A reflection is a private record; promotion to a Goal or Decision is a user-driven act.
- **Legal-triage logs always include the take-to-counsel block.** If a legal log is missing it, lint flags the page.

---

## Home.md update rules

`Home.md` is hand-curated. The plugin may add rows but never reorder, rename, or remove them. Match style: same column count, same link format. If the relevant table doesn't exist, skip — don't invent tables.

For the personal schema, Home.md typically opens with **Active Goals** (with status), **Open Decisions** (with deadline), and **Recent Journal** (last 7 days).

---

## log.md format

Append-only. Newest entries at the bottom.

```
## [YYYY-MM-DD] <type> | <title>
- <detail>
```

`<type>` ∈ `ingest`, `query`, `lint`, `schema`, `merge`, `gaps`, `session-import`, `session-capture`, `index`, `profile-update`, `reflection`, `decision`, `legal-triage`, `briefing`.

The personal schema's extra log types let `morning-briefing` and the grading hook find recent activity quickly without parsing every file.

---

## Lint criteria (mirrors the upstream `lint` skill, with personal-specific additions)

1. **Orphans** — pages with zero inbound `[[wikilinks]]`. Excludes `Home.md`, `raw/`, `Journal/` (journal entries are intentionally orphan-by-default), and `Profile/profile.md`.
2. **Broken wikilinks** — `[[Name]]` references whose target file does not exist.
3. **Missing frontmatter** — pages without a YAML frontmatter block matching the type-specific schema above.
4. **Decisions past deadline without grading** — decision entries with `status: pending` whose `deadline` is in the past. The `morning-briefing` skill should have surfaced these; if it hasn't been run in a while, lint flags them.
5. **Legal-triage missing take-to-counsel** — any file in `Legal/` whose body does not contain the take-to-counsel block.
6. **Possibly stale goals** — goals with `status: active` and `updated:` older than 90 days.

Lint runs in **report-only mode by default.** Fix mode requires explicit user request and confirms each edit individually.

---

## Vault index

`<vault root>/index.md` is a derived, machine-readable digest of every wiki page. It is written **only** by the `index` skill (`/obsidian-wiki:index`). Hand-edits are pointless — every run is a full rewrite. Excluded from lint orphan / broken-link detection.

---

## Schema evolution

This file is edited only by the `vault-schema-maintain` skill. Edits are surgical — minimal diffs, never whole-file rewrites. Every schema edit belongs to exactly one section above, is confirmed with the user, and produces a `[YYYY-MM-DD] schema |` entry in `log.md` with a `Reason:` line.

The personal schema in particular is not generic — adding categories, frontmatter fields, or types should be conservative. The point of the schema is that the user can predict where things land six months from now.
