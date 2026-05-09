# Changelog

## v0.4.0 — 2026-05-09

Three fixes shipped together because they share the personal-coach artifact and a re-validation pass.

### Fixed
- **`personal-coach` failed to install via Cowork's "upload custom plugin file" UI** ("Plugin validation failed"). Two contributing factors fixed in this release: (1) tightened every `SKILL.md` and agent `description` field to ≤600 chars, well below the ~800-char ceiling implied by the recent `marketplace-tour` description tightening; (2) relocated the `plugins/personal-coach/routines/` directory to `docs/personal-coach-routines/` at the repo root. Cowork's plugin validator differs from the documented Claude Code CLI validator (see [#24328](https://github.com/anthropics/claude-code/issues/24328)) and the safest posture is to ship only the recognized component dirs (`skills/`, `agents/`, `commands/`, `hooks/`, `assets/`).
- **The notes vault no longer requires a separate bootstrap command.** When a `personal-coach` skill needs to save something durable for the first time, it asks once where to put it (recommended `~/Notes/personal-coach`, custom path, or fall back to `~/.claude/personal-coach/`) and writes the schema + log + Home + `obsidian-wiki/config.json` itself. `vault-librarian` is no longer a prerequisite — it remains useful for users who want richer schemas (writer / researcher) or a vault to share across plugins.

### Changed
- **Onboarding rewritten to be value-first.** Both `welcome:marketplace-tour` and `personal-coach:onboarding` are now built around one short opening message — three sentences plus one open question (*"What's on your mind right now?"*) — and engage whatever the user actually says. Time-to-first-value drops from "five short phases, about five minutes" to one tangible artifact in turn 2-3. The persona quiz, the name question, the upfront skill list, and the five-phase scaffolding are all removed; the catalog appears only when the user explicitly asks. Sources: NN/g [*Mobile App Onboarding*](https://www.nngroup.com/articles/mobile-app-onboarding/), NN/g [*Onboarding Tutorials vs. Contextual Help*](https://www.nngroup.com/articles/onboarding-tutorials/), Sean Ellis *Hacking Growth* (2017), [ProductLed *AI Onboarding in 60 seconds*](https://productled.com/blog/ai-onboarding), [Appcues on Notion](https://goodux.appcues.com/blog/notions-lightweight-onboarding), Yifrah *Microcopy* (2nd ed., 2019), Krug *Don't Make Me Think* (3rd ed., 2014).
- **Plugin versions:** `personal-coach` 0.1.0 → 0.2.0, `welcome` 0.1.0 → 0.2.0.
- **Marketplace version:** 0.3.0 → 0.4.0; tightened `personal-coach` marketplace description to 433 chars (was 706).

### Plan
Full plan and citations: `docs/plans/onboarding-fixes-v0.4.0.md`.

---

## v0.3.0 — earlier

Switched release artifact to a single all-in-one zip; Cowork-first refactor, purge of Claude-Code-CLI-only paths, addition of `release.yml` workflow.
