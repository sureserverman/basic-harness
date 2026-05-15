# news-digest

Personalized daily news digest for Claude Cowork. The whole plugin is structured around **filtering by what the user has explicitly told it to track** — without that filter, a "news digest" is just a slower version of every news app you already ignore.

This plugin works standalone. If you also have `personal-coach` installed, the digest can be folded into the morning briefing; if you don't, the digest stands on its own.

## Install

See the [top-level basic-harness README](../../README.md#install) for the canonical install flow (Cowork → Customize → upload zip from this repo's GitHub releases).

## Start here

After installing, run:

```text
/news-digest:setup-news-digest
```

It will gate on `news-preferences` existing first — if you haven't captured your topics, sources, and exclusions yet, the setup command refuses to schedule and hands you over to the `news-preferences` skill. Once preferences exist, the setup walks you through Cowork's Scheduled Tasks UI so the digest lands every morning before you open anything else.

If you'd rather just produce one digest interactively without scheduling, ask in plain language ("give me my news", "news digest for today") — the `news-digest` skill fires directly. The same preferences gate applies.

## Skills

| Skill | Purpose |
|---|---|
| `news-preferences` | Capture, update, and read the user's news preferences — topics of interest with weights, topics to exclude, hard exclusions, preferred sources, sources to avoid, language, length, style, cadence, and an open-questions watchlist. Never edits silently — every write is proposed and confirmed. |
| `news-digest` | Produce a personalized daily digest against the saved preferences. Plans queries from topic weights, fetches via WebSearch / WebFetch (and Gmail / Drive connectors when granted), filters strictly (hard exclusions are absolute), synthesizes per format prefs, checks the open-questions watchlist, and writes a dated digest to disk. |

## Slash command

| Command | What it does |
|---|---|
| `/news-digest:setup-news-digest` | Schedule the daily digest as a Cowork Scheduled Task. Gates on `news-preferences` existing first; reads cadence from the preferences file if present; walks the user through `Scheduled → + New task` with a pre-filled prompt. |

## Where things live

| Artifact | Path (vault) | Path (no vault) |
|---|---|---|
| News preferences | `<vault>/Profile/news-preferences.md` | `~/.claude/news-preferences.md` |
| Daily digests | `<vault>/News/YYYY-MM-DD-digest.md` | `~/.claude/news/YYYY-MM-DD-digest.md` |
| Plugin state | — | `~/.claude/news-digest.local.md` |

If `vault-librarian` has bootstrapped a vault and a `<vault>/.obsidian-wiki.yaml` config exists, the vault path is used. Otherwise the home path. No separate vault setup is required to use this plugin.

## Cowork Routines (optional cloud automation)

A copy-paste Routine template lives at `docs/routines/news-digest-routine.md`. Routines run in Anthropic's cloud with the laptop closed; for news digests this is generally fine because search queries hit the public web anyway. **Read the privacy note in the routine doc before installing** — if your Hard exclusions touch identity-sensitive topics (health, sexuality, substance recovery), the cloud-routed search for the *interest* topic still logs through Anthropic even though the digest excludes the *result* topic. Desktop Scheduled Tasks avoid that.

## Hard limits

- **No scheduling without preferences.** The setup command refuses if `news-preferences` doesn't exist.
- **No fabricated items, ever.** If sources are unreachable, the digest is empty and says so explicitly with the queries attempted. Fake "based on general knowledge" items are the failure mode this plugin exists to prevent.
- **Cite every item.** Every bullet has a source name and a working URL. Items without a verifiable link don't ship.
- **Hard exclusions are not negotiable.** Even when the day's biggest story is on the Hard-excluded list, it stays out. The user has chosen to not read about it.

## Sources and rationale

- **Refuse without preferences** — Cal Newport, *Digital Minimalism* (2019): undirected information consumption is the failure mode of every news tool; preferences are the discipline.
- **Per-topic cap and weighting** — Tetlock & Gardner, *Superforecasting* (2015), Ch. 4 and Ch. 6: signal density beats coverage; capped lists outperform exhaustive ones; source quality dominates volume.
- **Open-questions watchlist** — Heuer, *Psychology of Intelligence Analysis* (CIA, 1999), Ch. 10 on Indicators and Warnings.
- **Hard-exclusion strictness** — Tversky & Kahneman, "Judgment under Uncertainty: Heuristics and Biases" (1974): even a single mention reshapes interpretation; soft filters do not work.
