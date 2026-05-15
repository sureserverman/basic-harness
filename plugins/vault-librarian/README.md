# vault-librarian

Substrate plugin for basic-harness. As of v0.2.0 (basic-harness v0.7.0), this is **the shared vault-companion surface** that `personal-coach`, `news-digest`, and `fintech-legal-advisor` call as sub-skills for every vault touch-point: bootstrap-on-first-need, recall before each substantive turn, append after.

Three internal sub-skills:

| Sub-skill | What it does |
|---|---|
| `vault-companion-ensure` | Resolves an existing vault from the user's config, or asks one question and bootstraps a personal-schema vault if none. Returns a vault-handle JSON. Cowork branch creates a Drive folder; CLI branch creates a local folder. Never re-asks. |
| `vault-companion-recall` | Takes a topic, returns up to 5 relevant pages with one-line excerpts. Delegates to `obsidian-wiki:ask` when installed; `grep` fallback otherwise. Read-only. Silent on empty. |
| `vault-companion-append` | Writes a page to `<vault>/<category>/YYYY-MM-DD-<slug>.md`, appends `log.md`, optionally chains `obsidian-wiki:ingest` based on a session-persisted preference. Schema-validates against `CLAUDE.md`. |

Plus one user-facing slash command:

| Command | What it does |
|---|---|
| `/vault-librarian:bootstrap-vault` | The explicit-setup path. Asks the user which of four persona-tuned schemas to use (researcher / writer / generic / personal) and writes the vault skeleton. Use this before any substantive skill triggers the auto-path if you want a non-personal schema. |

Companion plugin to the upstream [`obsidian-wiki`](https://github.com/sureserverman/obsidian-wiki-plugin). Without obsidian-wiki, recall falls back to grep and ingest chaining is skipped; the substantive vault writes still happen. Install obsidian-wiki for the full workflow.

**Optional** — but the four substantive vault-citizen skills (reflection-session, business-mentoring, news-digest, fintech-legal-triage) all degrade to non-vault fallbacks when this plugin is absent. Install this to get the substrate.

## Why

Upstream `obsidian-wiki` ships a great workflow over a Markdown vault, but its first-time setup assumes a Linux/macOS terminal:

```bash
mkdir -p ~/dev/knowledge/raw/assets
curl -fLsS .../vault-CLAUDE.md -o ~/dev/knowledge/CLAUDE.md
# ... etc
```

That blocks Windows desktop users and forces non-coders to copy-paste shell commands they don't trust. `vault-librarian` replaces it with a single in-Claude command.

## Install

See the [top-level basic-harness README](../../README.md#install) for the canonical install flow (Cowork → Customize → upload zip from this repo's GitHub releases). You also need the upstream `obsidian-wiki` plugin — install its zip from the [obsidian-wiki-plugin releases](https://github.com/sureserverman/obsidian-wiki-plugin/releases) the same way.

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
