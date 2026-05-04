# Vault Schema (CLAUDE.md) — Generic

This file is the schema for the notes vault at `__VAULT_PATH__`. It tells the `obsidian-wiki` plugin how this vault is organized and how to operate on it.

This is the **generic** schema — a flexible six-category layout that fits mixed-purpose knowledge management. If your work is more specifically research / analysis or writing / journalism, the `researcher` and `writer` templates from `vault-librarian` may fit better.

This file is **read by every wiki operation** (ingest, query, lint) and is **edited only by the `vault-schema-maintain` skill**, never in ad-hoc sessions. Edit it surgically — do not rewrite the whole document. Each change is recorded in `log.md` with type `schema`.

---

## Vault structure

| Path | Purpose |
|---|---|
| `Architecture/` | System-level designs that span multiple components or topics. |
| `Gotchas/` | Surprises, footguns, non-obvious failure modes you want to remember. |
| `Patterns/` | Reusable approaches and conventions that recur across projects. |
| `Platforms/` | Per-platform notes (one file per platform / environment). |
| `Projects/` | Per-project notes (one file per project / engagement). |
| `Technologies/` | Per-tool / per-protocol / per-domain notes (one file per topic). |
| `Home.md` | Hand-curated Map-of-Content. |
| `log.md` | Append-only activity log. |
| `raw/` | Source inbox. Articles, PDFs, clipped pages. **Immutable** — never edited. |
| `raw/assets/` | Images and binary assets referenced from `raw/`. |

The wiki layer is the six category directories. Everything else is infrastructure.

---

## Page naming

- **Title Case with spaces** allowed and preferred. Example: `Quarterly Review Process.md`.
- **One topic per file.** If you find yourself wanting to write `X and Y.md`, you probably want two files plus cross-links.
- **No date prefixes** in wiki page filenames. Dates live in frontmatter, not filenames. Source files in `raw/` may have date prefixes for sorting; wiki pages do not.
- **No subdirectories inside category dirs.** Flat structure. Use tags for subcategorization.

---

## Frontmatter schema

```yaml
---
title: <Page Title>
aliases: []
tags: [<topic>, <subtopic>]
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
sources: [raw/<filename>]
related: [[Other Page]]
---
```

Required fields: `title`, `created`. Everything else is optional but encouraged.

`updated` is bumped on every edit by the editing skill (never manually). `sources` is appended to (not replaced) when a new source contributes to an existing page.

---

## Category decision rules

When ingesting a source, pick exactly one category. Tie-breakers in order:

1. **Per-tool / per-protocol / per-domain info** → `Technologies/`.
2. **Per-platform / per-environment info** → `Platforms/`.
3. **Per-project / per-engagement info** → `Projects/`.
4. **Reusable approach across projects/tools** → `Patterns/`.
5. **A specific surprise or footgun you want to remember** → `Gotchas/`.
6. **A multi-component system design** → `Architecture/`.

If a source could fit two categories, pick the more specific one. If still tied, ask the user.

---

## Ingest procedure (mirrors the `ingest` skill)

1. Confirm the source is in `raw/`. If not, copy it in first.
2. Read the source.
3. Pick the category using the rules above.
4. Grep the target category for an existing page on the same topic. **Prefer updating over creating.**
5. Write or update the page using the frontmatter schema. Body has TL;DR + Key points + Details + Sources.
6. Scan the rest of the vault for entities the source mentions. Add `[[wikilinks]]` only where the source contributes new information — not for name-drops.
7. If a brand-new page was created in a category that has a table in `Home.md`, add one row. **Add only — never reorder, rename, or remove.**
8. Append a `[YYYY-MM-DD] ingest |` entry to `log.md`.

---

## Cross-reference rules

- A backlink is justified when the source contributes new information to the linked page. A pure name-drop is not.
- Use `[[Page Name]]` (Obsidian wikilinks). Aliases via `[[Page Name|alias]]`.
- Insert backlinks topically, not at the end of the file.
- If the link would create a contradiction with existing content, **do not silently resolve it.** Add a `## Conflicts` section instead, and ask the user.

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
3. **Missing frontmatter** — pages without a YAML frontmatter block containing at least `title` and `created`.
4. **Possible contradictions** (heuristic) — pages with overlapping topics that make opposing factual claims.
5. **Possibly stale** (heuristic) — pages with `updated:` older than 6 months that reference fast-moving topics.

Lint runs in **report-only mode by default.** Fix mode requires explicit user request and confirms each edit individually.

---

## Vault index

`<vault root>/index.md` is a derived, machine-readable digest of every wiki page. It is written **only** by the `index` skill (`/obsidian-wiki:index`). Hand-edits are pointless — every run is a full rewrite. Excluded from lint orphan / broken-link detection.

---

## Schema evolution

This file is edited only by the `vault-schema-maintain` skill. Edits are surgical — minimal diffs, never whole-file rewrites. Every schema edit belongs to exactly one section above, is confirmed with the user, and produces a `[YYYY-MM-DD] schema |` entry in `log.md` with a `Reason:` line.
