---
description: Set up a fresh Markdown notes vault for use with the obsidian-wiki plugin — picks a persona-tuned schema (researcher, writer, generic), writes CLAUDE.md / log.md / config.json into the right places, and works on Claude Code CLI and Desktop (macOS / Windows / Linux). No shell snippets to copy-paste.
---

# Bootstrap a notes vault

Set up a Markdown notes vault and the small config file the upstream `obsidian-wiki` plugin needs to find it. Cross-platform: works on Claude Code CLI and Desktop on macOS, Windows, and Linux. **You will only use the `Read`, `Write`, `Edit`, and `Glob` tools** — no `bash` / `mkdir` / `curl` / `cp`. Every file write goes through `Write` so the user sees and approves it.

## What this command does

1. Asks the user three questions (persona → vault location → confirmation).
2. Writes three files into the vault:
   - `CLAUDE.md` — the schema (one of three persona-tuned templates).
   - `log.md` — the append-only activity log.
   - `Home.md` — a starter map-of-content the upstream plugin's `rebuild-home` can later regenerate.
3. Writes one config file the upstream plugin reads:
   - `~/.config/obsidian-wiki/config.json` (Linux/macOS) or `%APPDATA%\obsidian-wiki\config.json` (Windows).
4. Reports what was written and gives the user the next two commands to run.

## Step 1 — Persona

Ask the user (multiple choice, one question):

> Which schema fits how you'll use this vault?
>
> **A — researcher / analyst.** Categories: `Sources/`, `Findings/`, `Briefs/`, `People/`, `Methods/`, `Questions/`. Frontmatter includes `cited-by`, `confidence`, `verified-on`. For investigations, papers, evidence trails.
>
> **B — writer / journalist.** Categories: `Pieces/`, `Sources/`, `People/`, `Places/`, `Topics/`, `Pitches/`. Frontmatter includes `status` (draft / filed / published), `outlet`, `embargo`. For long-form work with sources you'll come back to.
>
> **C — generic.** Categories: `Architecture/`, `Patterns/`, `Gotchas/`, `Platforms/`, `Projects/`, `Technologies/`. The upstream default. For mixed-purpose knowledge management.

Persona maps to one of the three template files in `${CLAUDE_PLUGIN_ROOT}/assets/`:

- A → `vault-CLAUDE-researcher.md`
- B → `vault-CLAUDE-writer.md`
- C → `vault-CLAUDE-generic.md`

If the user picks something else (e.g. "a hybrid"), recommend C as the starting point and tell them the schema is editable in place — they can switch later via the upstream `vault-schema-maintain` skill.

## Step 2 — Vault location

Ask:

> Where should the vault live? Default: `~/dev/knowledge`. Press enter to accept, or give a different path.

Resolve `~` against the user's home directory. On Windows desktop, `~` is the user profile (e.g. `C:\Users\Name`). Use forward slashes in paths you write to disk where the OS accepts them; on Windows, both `C:/Users/Name/dev/knowledge` and `C:\Users\Name\dev\knowledge` work in modern tools.

If the chosen path **already contains** any of `CLAUDE.md`, `log.md`, or `Home.md`, **stop and ask the user before overwriting**. Show the existing file's first 10 lines so they can decide whether to back up first. Never silently overwrite an existing vault.

## Step 3 — Confirm

Show the plan before writing anything:

```
About to create:
  <vault>/CLAUDE.md         (from <persona-template>)
  <vault>/log.md
  <vault>/Home.md
  <vault>/raw/.gitkeep
  <vault>/raw/assets/.gitkeep
  <config-dir>/config.json   ({ "default_vault": "<vault>", "vaults": { "<short-name>": "<vault>" } })

Proceed? (yes / no)
```

`<short-name>` is the basename of the vault path (e.g. `knowledge` for `~/dev/knowledge`).

`<config-dir>` is platform-specific:

- macOS / Linux: `~/.config/obsidian-wiki/`
- Windows: `%APPDATA%\obsidian-wiki\` (resolves to `C:\Users\<name>\AppData\Roaming\obsidian-wiki\`)

Wait for explicit `yes` before proceeding.

## Step 4 — Write the files

For each target file, use the `Write` tool. The persona templates ship as plain markdown under `${CLAUDE_PLUGIN_ROOT}/assets/`; read each one with the `Read` tool, then `Write` it to the destination unchanged. Do not edit template content during bootstrap — that happens later via the upstream `vault-schema-maintain` skill.

Order:

1. **`<vault>/CLAUDE.md`** ← read from `${CLAUDE_PLUGIN_ROOT}/assets/<persona-template>`. If the template has the literal token `__VAULT_PATH__` in it, replace with the absolute vault path on the way through.
2. **`<vault>/log.md`** ← read from `${CLAUDE_PLUGIN_ROOT}/assets/log-template.md`. Append a single bootstrap entry with today's date in `YYYY-MM-DD` format and the persona chosen:
   ```
   ## [YYYY-MM-DD] schema | Initialized vault via /vault-librarian:bootstrap-vault
   - Persona: <A | B | C>
   - Vault path: <vault>
   ```
3. **`<vault>/Home.md`** ← read from `${CLAUDE_PLUGIN_ROOT}/assets/home-template.md`. (Empty Map-of-Content; upstream `rebuild-home` populates it after the first ingest.)
4. **`<vault>/raw/.gitkeep`** and **`<vault>/raw/assets/.gitkeep`** ← write empty files (zero bytes) so the directories exist for the upstream `ingest` flow.
5. **`<config-dir>/config.json`** ← write JSON:
   ```json
   {
     "default_vault": "<absolute-vault-path>",
     "vaults": { "<short-name>": "<absolute-vault-path>" }
   }
   ```
   Use absolute paths — never `~`. Pretty-print with 2-space indent.

If the config file **already exists**, read it, parse the JSON, add the new vault entry under `vaults.<short-name>`, and only set `default_vault` if it isn't already set. Never clobber an existing config.

## Step 5 — Report and hand off

Tell the user:

> Vault bootstrapped at `<vault>`. Schema template: `<persona-template>`.
>
> Next steps:
>
> 1. Drop a source file (article, PDF, clipped page) into `<vault>/raw/`.
> 2. Run `/obsidian-wiki:ingest raw/<filename>` — the upstream plugin will file it into the right category.
> 3. Run `/obsidian-wiki:ask <question>` to query your notes with citations.
>
> If you haven't installed the upstream plugin yet:
>
> ```
> /plugin marketplace add sureserverman/obsidian-wiki-plugin
> /plugin install obsidian-wiki@obsidian-wiki
> ```

Do not invoke any obsidian-wiki command from here — the user runs them when they're ready.

## Failure modes

- **Vault path is unwritable** (permissions): stop, report which path failed, ask for a different location. Do not retry silently.
- **Config dir doesn't exist**: write the file anyway — `Write` creates parent directories. If that fails, report the path so the user can create it manually.
- **User aborts at Step 3**: report "no files written" and exit. Don't leave half-bootstrapped state.
- **Template asset missing under `${CLAUDE_PLUGIN_ROOT}/assets/`**: stop, report which template, tell the user to reinstall vault-librarian. Do not invent a template.

## What this command will NOT do

- Run shell commands (`mkdir`, `curl`, `cp`) — every file goes through `Write` so the user can audit.
- Reach the network. All assets ship inside the plugin.
- Modify an existing vault's schema in place. Use the upstream `vault-schema-maintain` skill for that.
- Install the upstream `obsidian-wiki` plugin. The user does that with `/plugin install`.
