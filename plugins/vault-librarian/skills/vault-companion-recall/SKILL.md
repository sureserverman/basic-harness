---
name: vault-companion-recall
description: Internal helper skill for other basic-harness plugins. Takes a topic string (or extracts one from the current dialog turn) and returns up to five relevant pages from the user's notes vault, with a one-line excerpt each. Delegates to obsidian-wiki's ask skill when obsidian-wiki is installed; otherwise falls back to a grep over the personal-schema category directories. Used by reflection-session, business-mentoring, news-digest, and fintech-legal-triage in their Phase 0.5 (vault recall) step. Returns an empty list silently if the vault is absent, empty, or has no matches. Triggers on "recall from vault", "vault-companion-recall", or programmatic calls.
---

# Vault Companion — Recall

Find pages in the user's vault that relate to a topic, before a substantive skill does its main work. Return up to five pages with one-line excerpts. Fail silently if the vault is empty.

This is **not a user-facing skill.** Substantive skills (`reflection-session`, `business-mentoring`, `news-digest`, `fintech-legal-triage`) call it as a sub-skill in their Phase 0.5 step. The caller decides what to do with the result — surface to the user, weave into the response, or silently use as context.

## Input

The caller provides:

- A **vault-handle** (from `vault-companion-ensure`). If the handle is null or absent, this skill returns an empty list immediately — don't prompt, don't bootstrap.
- A **topic string** — either pre-extracted by the caller (e.g., `fintech-legal-triage` passes "Estonia + payments") or left to the skill to extract from the current dialog turn.

## Output

A JSON-shaped result the caller can read or surface:

```json
{
  "matches": [
    { "path": "Decisions/2026-03-10-fire-cto.md", "excerpt": "Decision: replace the CTO. Status: graded. Reversibility: hard." },
    { "path": "Legal/2026-04-30-estonia-emi.md", "excerpt": "Estonia × payments cell. Issues identified: 7. Take-to-counsel block at bottom." }
  ],
  "vault_path": "/home/user/dev/knowledge",
  "transport": "local"
}
```

- `matches` — up to 5 items, ranked by relevance (most-related first). Empty list if no matches.
- `vault_path` — echo of the handle for caller convenience.
- `transport` — echo of the handle's transport.

## Phase 1 — Resolve vault, decide search strategy

Read `~/.claude/vault-companion.local.md` to get the current vault-handle. If null or unreadable, return `{ "matches": [], "vault_path": null, "transport": null }` — silent.

Detect whether obsidian-wiki is installed:

- Local: a `${CLAUDE_PLUGIN_ROOT}/../../obsidian-wiki/.claude-plugin/plugin.json` or similar marker exists.
- Cowork: the `obsidian-wiki:ask` skill is reachable.

Two search strategies:

- **Strategy A — obsidian-wiki delegate (preferred when available).** Invoke `obsidian-wiki:ask` with the topic. obsidian-wiki's `ask` skill is purpose-built for vault retrieval — it knows the schema, uses the frontmatter, respects category boundaries.
- **Strategy B — grep fallback (always available).** A direct case-insensitive search over the personal-schema category directories. Cheap, no LLM round-trip.

## Phase 2 — Topic extraction (if not provided)

If the caller didn't pass an explicit topic, infer one from the user's most recent message:

1. Pull noun phrases — proper nouns (named regulations / people / projects), domain words (jurisdiction names, regulation acronyms, role titles), and a small handful of theme words ("anxious", "decision", "launch", etc.).
2. Filter out generic words ("the", "want", "think").
3. Combine up to 3 most salient into a search string.

If the message has no extractable topic (e.g., it's purely procedural — "yes" / "continue"), return `{ "matches": [] }` immediately. The vault has nothing to add to a procedural exchange.

## Phase 3 — Search

### Strategy A — obsidian-wiki delegate

Hand off to `obsidian-wiki:ask` with the topic and the vault-handle. Read its response. Reshape into the `matches` list:

- Each page `ask` returns becomes one `match` entry.
- `excerpt` is the most relevant sentence from that page (`ask` typically returns one).
- Cap at 5.

### Strategy B — grep fallback

Use `Bash` with a single ripgrep / grep invocation:

```bash
grep -ril "<topic>" "<vault>/Profile/" "<vault>/Journal/" "<vault>/Goals/" "<vault>/Business/" "<vault>/Decisions/" "<vault>/Legal/" "<vault>/News/" "<vault>/People/" "<vault>/Reading/" 2>/dev/null | head -5
```

For each match, read the file's first 20 lines (Read with limit=20), extract the first prose sentence that contains the topic, build the `match` entry. If a match's file has frontmatter, prefer the line that contains the topic over the frontmatter for the excerpt.

For Cowork (transport=drive): grep doesn't work directly. The fallback when obsidian-wiki isn't installed AND transport=drive is to ask the user to install obsidian-wiki, then return an empty match list for this run. Don't try to invent a search over the Drive connector — that's expensive and there's a purpose-built skill (obsidian-wiki:ask) for it.

## Phase 4 — Rank and return

If there are more than 5 matches, rank by:

1. Recency — newer files (by date prefix in filename, or by frontmatter `updated:`) rank higher.
2. Specificity — files whose filename contains the topic rank higher than files where the topic appears only in body.
3. Category — for `fintech-legal-triage` callers, Legal/ ranks higher; for `reflection-session`, Journal/ ranks higher. The caller may pass a `category_preference` field to bias.

Cap at 5. Return.

## Hard rules

- **Silent on empty.** No "vault is empty, run X" hints from this skill — the caller decides whether to surface anything. This skill's job is to fetch, not to nag.
- **Never write to the vault.** Read-only. Even if the search reveals an obvious cross-reference gap, that's `obsidian-wiki:related`'s job, not this skill's.
- **Bounded cost.** At most one `obsidian-wiki:ask` invocation OR one grep + 5 small file reads. No LLM-tier ranking on grep results — recency + filename + category is the only ranking algorithm.
- **Honor the vault-handle's transport.** If `transport=drive`, never attempt local-filesystem reads of the Drive path — that won't resolve in Cowork's sandbox.

## What this skill is NOT

- Not a vault-wide question-answering skill (use `obsidian-wiki:ask` for that — this skill *delegates* to it).
- Not a backlink finder (use `obsidian-wiki:related` for that).
- Not a vault explorer (use `obsidian-wiki:gaps` or `obsidian-wiki:index` for that).

## Caller pattern

```text
Phase 0.5 (vault recall): Invoke vault-companion-recall with topic = <extracted from user message>.
  Read its returned matches.
  If matches is non-empty, weave into the next phase ("you've reflected on this before — want me to surface what you said?").
  If matches is empty, proceed silently.
```

Recall is a context enrichment step — it's never a gate.
