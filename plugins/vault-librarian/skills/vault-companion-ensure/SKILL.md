---
name: vault-companion-ensure
description: Internal helper skill for other basic-harness plugins. Resolves the user's notes vault — checks an existing config, asks one question if none, persists the choice — and returns a vault-handle JSON object the caller passes through its phases. Handles Cowork (Drive folder) and Claude Code CLI (local path) transparently. Auto-bootstraps a personal-schema vault on first need. Not user-facing — invoked by reflection-session, business-mentoring, news-digest, fintech-legal-triage, and personal-coach:onboarding before they write durable output. Triggers on "ensure vault", "vault-companion-ensure", or programmatic calls from other skills.
---

# Vault Companion — Ensure

Resolve the user's notes vault. Return a vault-handle JSON object. Bootstrap the vault if it doesn't exist yet, asking the user exactly one question. Persist the choice so subsequent calls are silent.

This is **not a user-facing skill.** It's a subroutine other plugins' substantive skills call before they save durable output. The caller passes the returned vault-handle through its phases — the handle's `transport` field tells downstream phases whether to write via `Write` tool (local) or via the Drive connector (Cowork).

## Vault-handle shape

Every call returns (or behaves as if it returned) a JSON object of this shape:

```json
{
  "path": "/abs/path/to/vault" | "Claude Vault/" | "Drive folder name",
  "transport": "local" | "drive",
  "schema": "personal" | "generic" | "researcher" | "writer",
  "bootstrapped": true | false
}
```

- `path` — absolute filesystem path (transport=local) OR Drive folder path relative to the Drive root (transport=drive).
- `transport` — how downstream phases should write. `local` ⇒ use `Write` tool. `drive` ⇒ use the Drive connector (caller is responsible for the connector call; this skill doesn't write through Drive itself).
- `schema` — which `CLAUDE.md` schema the vault uses. For new vaults bootstrapped here, default is `personal`.
- `bootstrapped` — `true` if this call just created the vault; `false` if it pre-existed.

## Resolution order

1. **Persisted handle.** Read `~/.claude/vault-companion.local.md` (or its Cowork-side equivalent — the file lives in the agent state directory, not the vault). If a `vault-handle:` YAML block is present and `path` resolves, return it immediately. No question to the user.
2. **`resolve-vault.sh` fallback (CLI only).** If `${CLAUDE_PLUGIN_ROOT}/scripts/resolve-vault.sh` exists OR the obsidian-wiki plugin's `scripts/resolve-vault.sh` is reachable, run it. If it prints an absolute path that exists, build a vault-handle (transport=local, schema=inferred from the vault's `CLAUDE.md` if present, else `personal`), persist it, return it.
3. **No vault found — ask the bootstrap question.** Proceed to **Phase 1 — Bootstrap**.

## Phase 1 — Bootstrap (only if Phase 0 / Phase 1 above didn't resolve)

Detect environment:

- **Cowork** — heuristics: presence of Drive connector signals in the session; user mentions "in Cowork" or the conversation has used a Drive folder previously; `${COWORK_SESSION}` or similar env var (if Anthropic ever surfaces one).
- **Claude Code CLI** — default if none of the Cowork heuristics hit.

Ask the user **exactly one question** in their language, calibrated to environment:

### Cowork branch

> I'd like to keep a personal knowledge vault for you — a Markdown notes folder that grows from our conversations (reflections, decisions, regulatory triages, news digests) and that I can recall context from in future sessions.
>
> Default: a Drive folder called `Claude Vault/` (I'll create it via the Drive connector). Alternatives: pick a different Drive folder name, or skip the vault entirely for now.
>
> **OK with `Drive/Claude Vault/`, or different name, or skip?**

Branches:

- `OK` / `yes` / `Drive/Claude Vault/ is fine` → set `path="Claude Vault/"`, `transport="drive"`, `schema="personal"`. **The caller is responsible for the Drive connector call** that actually creates the folder; this skill cannot itself touch Drive. Persist the handle anyway with `bootstrapped: pending-connector`; downstream `vault-companion-append` will complete the bootstrap on its first write.
- A different name (e.g. `Notes/Personal`) → same as above with that path.
- `skip` → return `null`. Persist `vault-handle: null` in the state file so we don't re-ask. The caller's skill must then fall back to its own non-vault path (e.g., `~/.claude/journal/`).

### CLI branch

> I'd like to keep a personal knowledge vault for you — a Markdown notes folder that grows from our conversations (reflections, decisions, regulatory triages, news digests) and that I can recall context from in future sessions.
>
> Default: `~/dev/knowledge/`. Alternatives: any local path you'd rather use, or skip the vault entirely for now.
>
> **OK with `~/dev/knowledge/`, different path, or skip?**

Branches:

- Default / a custom path → write the personal-schema vault files at that path using the templates in `${CLAUDE_PLUGIN_ROOT}/assets/`:
  - `CLAUDE.md` ← `${CLAUDE_PLUGIN_ROOT}/assets/vault-CLAUDE-personal.md` (with `__VAULT_PATH__` substituted to the chosen vault path)
  - `log.md` ← `${CLAUDE_PLUGIN_ROOT}/assets/log-template.md`
  - `Home.md` ← `${CLAUDE_PLUGIN_ROOT}/assets/home-template.md`
  - `raw/` and `raw/assets/` — empty directories (write a `.gitkeep` file if the bootstrap layer needs a file to materialize the directory)
  - Then write `~/.config/obsidian-wiki/config.json` (or `%APPDATA%\obsidian-wiki\config.json` on Windows) with `{ "default_vault": "<path>", "vaults": { "<basename>": "<path>" } }`
- `skip` → return `null`. Persist `vault-handle: null` so we don't re-ask.

Use **only the `Read`, `Write`, and `Glob` tools** — no `Bash`, no `mkdir`, no `cp`. Every file write goes through `Write` so the user sees and approves it. This mirrors the discipline of `/vault-librarian:bootstrap-vault`.

### Schema choice

vault-librarian ships four persona-tuned schemas in `${CLAUDE_PLUGIN_ROOT}/assets/`: `vault-CLAUDE-personal.md`, `vault-CLAUDE-generic.md`, `vault-CLAUDE-researcher.md`, `vault-CLAUDE-writer.md`. **vault-companion-ensure always uses `personal`** — it's the schema basic-harness's substantive skills (reflection-session, business-mentoring, news-digest, fintech-legal-triage) are wired against. Users who want a different persona should invoke `/vault-librarian:bootstrap-vault` explicitly before any substantive skill triggers the auto-path.

## Phase 2 — Persist

Write the resolved handle to `~/.claude/vault-companion.local.md`:

```markdown
---
vault-handle:
  path: <resolved path>
  transport: local | drive
  schema: personal | generic | researcher | writer
  bootstrapped-at: <YYYY-MM-DD>
auto-ingest-pref: ask-each   # set by vault-companion-append on first run; default ask-each
---

# Vault Companion — local state

(Do not edit this file by hand. It's managed by vault-companion-ensure and vault-companion-append.)
```

If the user chose `skip`, write `vault-handle: null` instead.

## Phase 3 — Return the handle

The skill's "return value" is the vault-handle. In practice, the caller reads `~/.claude/vault-companion.local.md` after this skill finishes — that's the contract.

## Hard rules

- **Ask exactly one question on bootstrap.** Don't run a multi-step persona wizard — that's what `/vault-librarian:bootstrap-vault` is for, and users who want depth can invoke it explicitly. This skill is the auto-path.
- **Never re-ask.** If `~/.claude/vault-companion.local.md` exists and is parseable, the answer it carries is final until the user explicitly invokes a re-setup command.
- **`skip` is a first-class outcome.** Don't nag, don't print install hints, don't repeat the question in a different form. Return `null` and let the caller fall back.
- **No Drive writes from this skill.** If `transport=drive` and the Drive folder doesn't exist yet, mark the handle as `bootstrapped: pending-connector` and let `vault-companion-append`'s first write trigger the actual Drive folder creation (which it does via the Drive connector the caller has access to).
- **No vault content writes from this skill.** Bootstrap writes only the skeleton (CLAUDE.md, log.md, Home.md, raw/). Any user-content writes are `vault-companion-append`'s job.

## What this skill is NOT

- Not a personality wizard (use `/vault-librarian:bootstrap-vault` for the 4-persona flow).
- Not a migration tool (no moving of existing `~/.claude/journal/` etc. into the vault — that's a separate `/vault-companion:migrate-local-history` skill, out of scope for v0.7.0).
- Not a vault validator (use obsidian-wiki's `lint` skill for that).

## Caller pattern

A typical substantive skill (e.g. `fintech-legal-triage`) calls this skill as a sub-skill at the start of its work:

```text
Phase 0 (or earlier): Invoke vault-companion-ensure.
  Read ~/.claude/vault-companion.local.md after it returns.
  If vault-handle is null, fall back to the skill's non-vault path.
  Otherwise, hold the handle through the phases and pass it to vault-companion-append at Phase N.
```

This pattern is documented in each retrofitted skill's Phase 0.5 / Phase N text.
