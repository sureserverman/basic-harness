# Vault Schema (CLAUDE.md) — Writer / Journalist

This file is the schema for the notes vault at `__VAULT_PATH__`. It tells the `obsidian-wiki` plugin how this vault is organized and how to operate on it.

This vault is set up for **long-form writing and journalism**: pieces in flight, sources you'll go back to, people you've spoken with, places and topics you cover repeatedly, pitches and ledes parked for later.

This file is **read by every wiki operation** (ingest, query, lint) and is **edited only by the `vault-schema-maintain` skill**, never in ad-hoc sessions. Edit it surgically — do not rewrite the whole document. Each change is recorded in `log.md` with type `schema`.

---

## Vault structure

| Path | Purpose |
|---|---|
| `Pieces/` | One page per article, essay, or chapter (in any state from pitch to published). |
| `Sources/` | One page per source (a person, document, dataset, archive). Where every quote and fact eventually traces back. |
| `People/` | Per-person notes — interviewees, characters, reporters, editors. Includes contact, prior interactions, on-the-record status. |
| `Places/` | Per-place notes for geography you cover repeatedly. Local context that's expensive to re-derive. |
| `Topics/` | Per-topic notes (a beat, a recurring theme, a specialized vocabulary). The reusable knowledge layer. |
| `Pitches/` | Pitch ideas, ledes, and angles parked for later. Promote to `Pieces/` when you start work. |
| `Home.md` | Hand-curated Map-of-Content. |
| `log.md` | Append-only activity log. |
| `raw/` | Source inbox. Interview audio, document scans, downloaded transcripts. **Immutable** — never edited. |
| `raw/assets/` | Images and binary assets referenced from `raw/`. |

The wiki layer is the six category directories. Everything else is infrastructure.

---

## Page naming

- **Title Case with spaces** allowed and preferred. Example: `Maria Hernandez (city councilor).md`.
- **One topic per file.** A profile of a person is one file; their organization is another file with `[[wikilinks]]`.
- **No date prefixes** in wiki page filenames. Dates live in frontmatter. Dates in `Pieces/` filenames are also avoided — the piece is the same piece if it slips a week.
- **No subdirectories inside category dirs.** Use tags.

---

## Frontmatter schema

```yaml
---
title: <Page Title>
aliases: []
tags: [<beat>, <topic>]
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
sources: [Sources/<page-name>]    # source pages this rests on
related: [[Other Page]]
---
```

Pieces add:

```yaml
status: pitch | drafting | filed | published | killed
outlet: <name or "unassigned">
embargo: <YYYY-MM-DD or "none">
deadline: <YYYY-MM-DD or "none">
word-target: <number or "none">
```

Sources add:

```yaml
on-record: yes | no | background | partial   # default and per-quote disagreements both noted in body
last-contact: <YYYY-MM-DD>
```

Required fields: `title`, `created`. Pieces require `status`. Sources require `on-record`.

`updated` is bumped on every edit by the editing skill (never manually).

---

## Category decision rules

When ingesting a source, pick exactly one category. Tie-breakers in order:

1. **A piece in any state** (pitch / draft / filed / published) → `Pieces/`. The page tracks the piece across its lifecycle.
2. **A primary source** (interview transcript, document, dataset) → `Sources/`. Source files live in `raw/`; the Source page is metadata + key excerpts.
3. **A person's profile** → `People/`. Includes contact, prior interactions, conflicts.
4. **A place** you cover repeatedly → `Places/`. Local context, recurring institutions, geography.
5. **A topic / beat** with reusable knowledge → `Topics/`. The vocabulary, the players, the running questions.
6. **A pitch idea, lede, or angle** → `Pitches/`. Promote to `Pieces/` when you start work.

If a source could fit two categories, pick the more specific one. If still tied, ask the user.

---

## Ingest procedure (mirrors the `ingest` skill)

1. Confirm the source is in `raw/`. If not, copy it in first.
2. Read the source.
3. Pick the category using the rules above.
4. Grep the target category for an existing page on the same topic. **Prefer updating over creating.**
5. Write or update the page using the frontmatter schema. Body has TL;DR + Key points + Details + Sources.
6. Quotes are attributed inline with `[[Sources/X]]` cites and the on-record status if it differs from the default.
7. Scan the rest of the vault for entities the source mentions. Add `[[wikilinks]]` only where the source contributes new information — not for name-drops.
8. If a brand-new page was created in a category that has a table in `Home.md`, add one row. **Add only — never reorder, rename, or remove.**
9. Append a `[YYYY-MM-DD] ingest |` entry to `log.md`.

---

## Cross-reference rules

- A backlink is justified when the source contributes new information to the linked page. A pure name-drop is not.
- Use `[[Page Name]]` (Obsidian wikilinks). Aliases via `[[Page Name|alias]]`.
- Insert backlinks topically, not at the end of the file.
- If the link would create a contradiction with existing content, **do not silently resolve it.** Add a `## Conflicts` section instead, and ask the user.
- Quotes that change attribution between background / on-record / partial are flagged on the `Sources/` page itself, not silently re-used.

---

## Embargo and on-record handling

- A `Pieces/` page with a future `embargo:` date must not be returned in queries until that date has passed, unless the query explicitly opts in (`/obsidian-wiki:ask --include-embargoed`).
- A `Sources/` page with `on-record: background` returns its non-attributable facts but the source's identity is not surfaced unless the query is run with `--unmask` (and you should think twice before doing that).
- These rules are enforced by the `ask` skill, not by lint — don't rely on file-system permissions.

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
3. **Missing frontmatter** — pages without the fields required for their category.
4. **Unattributed quotes** (writer-specific) — quoted text in `Pieces/` without an inline `[[Sources/X]]` cite.
5. **Embargo violations** (writer-specific) — `Pieces/` pages whose `embargo:` is in the past but `status:` is still `pitch` or `drafting` (probably stale state).
6. **Stale source contact** (writer-specific) — `People/` or `Sources/` pages with `last-contact:` older than 12 months that are referenced by an in-flight `Pieces/` page.

Lint runs in **report-only mode by default.** Fix mode requires explicit user request and confirms each edit individually.

---

## Vault index

`<vault root>/index.md` is a derived, machine-readable digest of every wiki page. It is written **only** by the `index` skill (`/obsidian-wiki:index`). Hand-edits are pointless — every run is a full rewrite. Excluded from lint orphan / broken-link detection.

---

## Schema evolution

This file is edited only by the `vault-schema-maintain` skill. Edits are surgical — minimal diffs, never whole-file rewrites. Every schema edit belongs to exactly one section above, is confirmed with the user, and produces a `[YYYY-MM-DD] schema |` entry in `log.md` with a `Reason:` line.
