# vault-librarian

Companion plugin to the upstream [`obsidian-wiki`](https://github.com/sureserverman/obsidian-wiki-plugin). Provides one cross-platform slash command that bootstraps a fresh **local** Markdown notes vault from inside Claude Code — no shell snippets, no remote storage, no cloud account, works on Claude Code Desktop on macOS and Windows.

**Optional.** The rest of `basic-harness` (process-skills, delegation-agents) works without a vault. Install this only if you want to keep notes that the upstream `obsidian-wiki` plugin can ingest, query, and lint from inside Claude Code.

## Why

Upstream `obsidian-wiki` ships a great workflow over a Markdown vault, but its first-time setup assumes a Linux/macOS terminal:

```bash
mkdir -p ~/dev/knowledge/raw/assets
curl -fLsS .../vault-CLAUDE.md -o ~/dev/knowledge/CLAUDE.md
# ... etc
```

That blocks Windows desktop users and forces non-coders to copy-paste shell commands they don't trust. `vault-librarian` replaces it with a single in-Claude command.

## Install

```text
/plugin marketplace add sureserverman/basic-harness
/plugin install vault-librarian@basic-harness
/plugin install obsidian-wiki@obsidian-wiki        # the upstream plugin this complements
```

## Use

```text
/vault-librarian:bootstrap-vault
```

The command will:

1. Ask which schema fits your work — researcher, writer/journalist, or generic.
2. Ask where the vault should live (default `~/dev/knowledge`).
3. Show you exactly which files it will create.
4. Wait for your `yes`.
5. Write `CLAUDE.md`, `log.md`, `Home.md`, and the small `config.json` the upstream plugin reads.
6. Tell you the next two commands to run.

Every file is written through Claude Code's `Write` tool, so you see the path and contents before each one lands. **Nothing touches the network.** Nothing runs `bash`. The vault is purely local — a directory of Markdown files at the path you choose. If you want sync (Obsidian Sync, Syncthing, iCloud Drive, Dropbox), layer it on yourself; vault-librarian doesn't do sync and doesn't care how you sync.

## What's inside

```
vault-librarian/
├── .claude-plugin/plugin.json
├── README.md
├── commands/
│   └── bootstrap-vault.md          # the only slash command
└── assets/
    ├── vault-CLAUDE-researcher.md  # schema for research / analysis work
    ├── vault-CLAUDE-writer.md      # schema for long-form writing / journalism
    ├── vault-CLAUDE-generic.md     # mixed-purpose schema (the upstream default shape)
    ├── log-template.md
    └── home-template.md
```

The schema templates differ in:

- Top-level categories (`Sources/Findings/Briefs/...` vs. `Pieces/Sources/People/...` vs. `Architecture/Patterns/Gotchas/...`).
- Frontmatter fields (`confidence` + `verified-on` for researchers; `status` + `embargo` + `on-record` for writers).
- Lint criteria specific to the persona (unsupported claims, embargo violations, stale source contact).

The shared infrastructure (ingest procedure, log format, index, schema evolution) is identical across all three.

## After bootstrap

Run upstream `obsidian-wiki` commands as documented:

- `/obsidian-wiki:ingest raw/<filename>` — file a source into the right category
- `/obsidian-wiki:ask <question>` — query your notes with citations
- `/obsidian-wiki:lint` — find orphans, broken wikilinks, persona-specific issues
- `/obsidian-wiki:rebuild-home` — refresh the `Home.md` map-of-content

To switch schemas later without losing your vault, use the upstream `vault-schema-maintain` skill — it edits `CLAUDE.md` surgically and records every change in `log.md`.

## Scope

`vault-librarian` is intentionally tiny. It only ships what's missing for non-coders on desktop Claude Code. All actual vault operations come from the upstream `obsidian-wiki` plugin.
