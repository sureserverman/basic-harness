---
name: vault-companion-append
description: Internal helper skill for other basic-harness plugins. Takes a vault-handle, a category name, a markdown body, and optional frontmatter, and writes a new page to the user's notes vault — to <vault>/<category>/YYYY-MM-DD-<slug>.md, with a log.md append, and optionally a copy to raw/ for obsidian-wiki ingest. Honors a session-persisted auto-ingest preference (ask-each / yes / no) so the user controls whether obsidian-wiki:ingest chains automatically after every append. Used by reflection-session, business-mentoring, news-digest, and fintech-legal-triage in their Phase N save step. Triggers on "append to vault", "vault-companion-append", or programmatic calls.
---

# Vault Companion — Append

Write a markdown page to the user's vault in the right category, append a log entry, and optionally trigger obsidian-wiki's ingest. This is the unified save surface every substantive plugin skill calls in its final phase.

This is **not a user-facing skill.** It's a sub-skill substantive skills call when they're ready to save durable output. The caller passes a vault-handle (from `vault-companion-ensure`), the category, and the body.

## Input

- **vault-handle** — JSON from `vault-companion-ensure` via `~/.claude/vault-companion.local.md`. If null, this skill returns `{ "written": false, "reason": "no-vault" }` and the caller falls back to its own non-vault path.
- **category** — one of the personal-schema categories: `Profile`, `Journal`, `Goals`, `Business`, `Decisions`, `Legal`, `News`, `People`, `Reading`. Or `raw` for inbox-only writes (skipping a category page). If the category isn't in the vault's `CLAUDE.md`, refuse and surface a hint — the schema is authoritative.
- **body** — the markdown body of the page (without frontmatter; this skill adds the frontmatter).
- **frontmatter** (optional) — a YAML-shaped key/value object the caller wants in the page's frontmatter. The skill normalizes against the schema's frontmatter template for that category (e.g., reflection-session sends `emotion`, `distortions`, `action` for `Journal/` entries; this skill validates against `Journal`'s schema and writes the union).
- **slug** (optional) — a kebab-case identifier for the filename. If absent, derive from the first 5–7 meaningful words of the body's first heading.
- **source_skill** (optional but recommended) — the caller's skill name (e.g., `fintech-legal-triage`). Used in the `log.md` entry.

## Output

```json
{
  "written": true,
  "path": "<vault>/Legal/2026-05-15-estonia-emi.md",
  "raw_copy": "<vault>/raw/2026-05-15-estonia-emi.md" | null,
  "log_entry_added": true,
  "ingest_chained": "ask-each-yes" | "ask-each-no" | "yes-pref" | "no-pref" | "not-installed"
}
```

If `written: false`, also include `reason` (`no-vault`, `category-not-in-schema`, `user-declined-save`, `drive-write-failed`).

## Phase 1 — Resolve and validate

Read `~/.claude/vault-companion.local.md`. If `vault-handle` is null or unreadable, return `{ "written": false, "reason": "no-vault" }`.

Validate the category against the vault's `CLAUDE.md` schema:

- Read `<vault>/CLAUDE.md` (or its Drive equivalent). Find the vault-structure table.
- Confirm the category has a row. If not, return `{ "written": false, "reason": "category-not-in-schema" }`. Surface a hint to the caller: "Category `<X>` is not in this vault's schema. Run `obsidian-wiki:vault-schema-maintain` to add it."

For Drive transport: validate the Drive folder exists. If not (handle was `bootstrapped: pending-connector`), create it via the Drive connector now and update the handle to `bootstrapped: true`.

## Phase 2 — Build the page

Filename: `YYYY-MM-DD-<slug>.md`. If a file with the same path already exists, append `-2`, `-3`, etc. — never overwrite.

Frontmatter:

1. Read the schema's frontmatter template for the category from `<vault>/CLAUDE.md`.
2. Merge the caller's `frontmatter` field into the template. Caller wins on conflicts.
3. Always include `date: <YYYY-MM-DD>` and `source-skill: <source_skill>` (if provided).
4. Validate required fields are present (e.g., `Journal` requires `type: reflection | briefing`; `Decisions` requires `status` and `reversibility`). If missing, surface to the caller; don't silently default.

Body:

- Pass-through from the caller, **with `[[wikilink]]` enrichment for named entities** the caller flagged in the frontmatter (e.g., `regulations: [MiCA, PSD2]` → wrap `MiCA` and `PSD2` in `[[...]]` form in the body where they appear in prose).
- If the body already has wikilinks, leave them alone — no double-wrapping.

## Phase 3 — Write

**Local transport.** Use the `Write` tool: one call for `<vault>/<category>/<filename>`, then a `Read` + `Write` cycle on `<vault>/log.md` to append:

```markdown
- <YYYY-MM-DD> auto-capture (<source_skill>) — <category>/<filename> — <first-line-of-body>
```

**Drive transport.** The caller is responsible for the Drive connector write. This skill returns the intended path and body; the caller's skill must invoke the Drive connector and confirm the write succeeded. If the caller signals success, this skill then appends to `log.md` (also via Drive). If the Drive write fails, return `{ "written": false, "reason": "drive-write-failed" }`.

## Phase 4 — Optional `raw/` copy and ingest chaining

Check `auto-ingest-pref` in `~/.claude/vault-companion.local.md`:

- **`yes-pref`** (user set "always ingest"): always drop a copy to `<vault>/raw/<filename>` and chain-call `obsidian-wiki:ingest <vault>/raw/<filename>`. Return `ingest_chained: "yes-pref"`.
- **`no-pref`** (user set "never ingest"): skip both the raw copy and the ingest. Return `ingest_chained: "no-pref"`.
- **`ask-each`** (default on first run): ask the user ONCE per session:

> Should I also ingest this into the obsidian-wiki layer? Ingest cross-links it with related vault pages (e.g., this triage's `[[MiCA]]` mention will surface as a backlink on the MiCA page next time). One-shot question — I'll remember your answer for this session. Options: **yes** (ingest this and ask again next time) / **no** (skip this one) / **always** (yes-pref, never ask again) / **never** (no-pref, never ask again).

  - yes → drop raw/ copy + chain ingest. Return `ingest_chained: "ask-each-yes"`.
  - no → skip both. Return `ingest_chained: "ask-each-no"`.
  - always → set `auto-ingest-pref: yes-pref` in the state file, then drop raw/ + chain ingest.
  - never → set `auto-ingest-pref: no-pref`, then skip both.

- **obsidian-wiki not installed**: skip the question and skip ingest. Return `ingest_chained: "not-installed"`. Don't print a nudge — that's vault-librarian's README's job.

The ingest call is a skill handoff: `obsidian-wiki:ingest <path>`. The caller in the parent session executes it; this skill's role is to decide whether to request it.

## Phase 5 — Return

Build the result JSON per the Output section. Return.

## Hard rules

- **Never overwrite.** Same-name files get `-2`, `-3`, etc.
- **Schema is authoritative.** If `<vault>/CLAUDE.md` doesn't list the category, refuse — don't silently extend the vault.
- **One log entry per write.** Even if a write triggers a raw/ copy and an ingest, `log.md` gets one auto-capture line, not three.
- **No silent ingest.** The first time per session, ask. After that, honor the persisted preference.
- **Honor `transport`.** Local writes go through `Write`; Drive writes are the caller's responsibility (the connector lives in the parent session, not in this sub-skill). This skill returns enough info for the caller to do the Drive write itself.

## What this skill is NOT

- Not a vault editor (only appends new pages; doesn't modify existing ones — that's `vault-schema-maintain` or `obsidian-wiki:lint`).
- Not a draft cache (writes are durable on first call; no "are you sure" confirmation beyond the ingest question).
- Not a sync layer (Drive-vs-local is decided at vault-handle creation; this skill just dispatches).

## Caller pattern

```text
Phase N (save): Build the body and category. Invoke vault-companion-append with:
  - vault-handle (read from ~/.claude/vault-companion.local.md)
  - category = "Legal" | "Journal" | "Decisions" | "News" | ...
  - body = <the markdown the skill produced>
  - frontmatter = { type: ..., status: ..., [regulations: [...] | distortions: [...]] }
  - source_skill = "<skill-name>"

  Read the returned JSON.
  If written: false, fall back to the skill's non-vault path or surface the reason.
  If written: true, tell the user the path and the ingest status.
```
