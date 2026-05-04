# Vault Schema (CLAUDE.md) — Researcher / Analyst

This file is the schema for the notes vault at `__VAULT_PATH__`. It tells the `obsidian-wiki` plugin how this vault is organized and how to operate on it.

This vault is set up for **research and analysis work**: investigations, papers, evidence trails, source-grounded claims that someone else may need to verify.

This file is **read by every wiki operation** (ingest, query, lint) and is **edited only by the `vault-schema-maintain` skill**, never in ad-hoc sessions. Edit it surgically — do not rewrite the whole document. Each change is recorded in `log.md` with type `schema`.

---

## Vault structure

| Path | Purpose |
|---|---|
| `Sources/` | One page per primary source (a paper, dataset, interview, document). Where every claim eventually traces back. |
| `Findings/` | One page per substantive finding. Each cites the sources it rests on. The most reusable layer. |
| `Briefs/` | Composed deliverables — memos, summaries, draft sections of a longer piece. |
| `People/` | Per-person notes (interviewees, authors, subjects). Includes contact, prior work, conflicts. |
| `Methods/` | Reusable approaches: how a particular dataset is queried, how a sample was drawn, how a coding scheme was applied. |
| `Questions/` | Open research questions that don't yet have a Finding. Promote to `Findings/` once answered. |
| `Home.md` | Hand-curated Map-of-Content. |
| `log.md` | Append-only activity log. |
| `raw/` | Source inbox. Articles, PDFs, downloaded data. **Immutable** — never edited. |
| `raw/assets/` | Images and binary assets referenced from `raw/`. |

The wiki layer is the six category directories. Everything else is infrastructure.

---

## Page naming

- **Title Case with spaces** allowed and preferred. Example: `Bureau of Labor Statistics QCEW Methodology.md`.
- **One topic per file.** If you want to write `X and Y.md`, you probably want two files plus cross-links.
- **No date prefixes** in wiki page filenames. Dates live in frontmatter.
- **No subdirectories inside category dirs.** Flat structure: `Findings/X.md`, not `Findings/economic/X.md`. Use tags.

---

## Frontmatter schema

```yaml
---
title: <Page Title>
aliases: []
tags: [<topic>, <subtopic>]
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
sources: [Sources/<page-name>]    # other Source pages this rests on
cited-by: []                      # back-pointers; appended automatically
confidence: high | medium | low   # how strong is the underlying evidence
verified-on: <YYYY-MM-DD>         # last time the claim was checked
related: [[Other Page]]
---
```

Required fields: `title`, `created`. For `Findings/`, also require `sources` and `confidence`. For `Sources/`, also require `verified-on`.

`updated` is bumped on every edit by the editing skill (never manually). `cited-by` is maintained by the `lint` skill, not by hand.

---

## Category decision rules

When ingesting a source, pick exactly one category. Tie-breakers in order:

1. **A primary source itself** (paper, dataset, interview transcript, official document) → `Sources/`. The page is metadata + key excerpts; the source file lives in `raw/`.
2. **A substantive finding or claim** built from one or more sources → `Findings/`. Must cite at least one Source.
3. **A composed deliverable** (memo, summary, brief) → `Briefs/`. Briefs cite Findings, not raw sources directly.
4. **A person's profile or contact** → `People/`.
5. **A reusable method** (data-collection approach, coding scheme, query template) → `Methods/`.
6. **An open question without a current answer** → `Questions/`. Promote to `Findings/` once you can answer it.

If a source could fit two categories, pick the more specific one. If still tied, ask the user.

---

## Ingest procedure (mirrors the `ingest` skill)

1. Confirm the source is in `raw/`. If not, copy it in first.
2. Read the source.
3. Pick the category using the rules above.
4. Grep the target category for an existing page on the same topic. **Prefer updating over creating.**
5. Write or update the page using the frontmatter schema. Body has TL;DR + Key points + Details + Sources.
6. For `Findings/` pages, every claim in Key points / Details has an inline cite to a Source page (`[[Sources/X]]`).
7. Scan the rest of the vault for entities the source mentions. Add `[[wikilinks]]` only where the source contributes new information — not for name-drops.
8. If a brand-new page was created in a category that has a table in `Home.md`, add one row. **Add only — never reorder, rename, or remove.**
9. Append a `[YYYY-MM-DD] ingest |` entry to `log.md`.

---

## Cross-reference rules

- A backlink is justified when the source contributes new information to the linked page. A pure name-drop is not.
- Use `[[Page Name]]` (Obsidian wikilinks). Aliases via `[[Page Name|alias]]`.
- Insert backlinks topically, not at the end of the file.
- If the link would create a contradiction with existing content, **do not silently resolve it.** Add a `## Conflicts` section instead, and ask the user.
- For `Findings/` pages, every factual claim cites its Source inline; never aggregate cites at the end.

---

## Home.md update rules

`Home.md` is hand-curated. The plugin may add rows but never reorder, rename, or remove them. Match style: same column count, same link format. If the relevant table doesn't exist, skip — don't invent tables.

---

## log.md format

Append-only. Newest entries at the bottom.

```
## [YYYY-MM-DD] <type> | <title>
- <detail>
```

`<type>` ∈ `ingest`, `query`, `lint`, `schema`, `merge`, `gaps`, `session-import`, `session-capture`, `index`.

`query` entries are only logged when a new page was filed back. Plain queries that just produced an answer are not logged.

---

## Lint criteria (mirrors the `lint` skill)

1. **Orphans** — pages with zero inbound `[[wikilinks]]`. Excludes `Home.md` and `raw/`.
2. **Broken wikilinks** — `[[Name]]` references whose target file does not exist.
3. **Missing frontmatter** — pages without a YAML frontmatter block containing the required fields for their category.
4. **Unsupported claims** (researcher-specific) — `Findings/` pages whose Key points lack inline source cites.
5. **Stale verification** (researcher-specific) — `Sources/` pages with `verified-on:` older than 12 months.
6. **Possible contradictions** (heuristic) — pages with overlapping topics that make opposing factual claims.

Lint runs in **report-only mode by default.** Fix mode requires explicit user request and confirms each edit individually.

---

## Vault index

`<vault root>/index.md` is a derived, machine-readable digest of every wiki page. It is written **only** by the `index` skill (`/obsidian-wiki:index`). Hand-edits are pointless — every run is a full rewrite. Excluded from lint orphan / broken-link detection.

---

## Schema evolution

This file is edited only by the `vault-schema-maintain` skill. Edits are surgical — minimal diffs, never whole-file rewrites. Every schema edit belongs to exactly one section above, is confirmed with the user, and produces a `[YYYY-MM-DD] schema |` entry in `log.md` with a `Reason:` line.
