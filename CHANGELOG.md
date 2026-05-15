# Changelog

## v0.7.0 — 2026-05-15

Vault as substrate. The obsidian-wiki notes vault becomes the universal long-term memory shared across every substantive skill in basic-harness. Reflections, decisions, regulatory triages, and news digests no longer live in plugin-specific corners — they all flow through one shared vault-companion surface that auto-bootstraps on first need, recalls relevant prior pages before each substantive skill runs, and writes new pages after. The vault grows from every conversation that triggers a vault-citizen skill.

### Added
- **`vault-companion-ensure`** sub-skill under `vault-librarian` (new). Resolves the user's notes vault — checks an existing config, asks exactly one question if none, persists the choice, returns a vault-handle JSON ({path, transport, schema, bootstrapped}). Cowork branch defaults to a Drive folder (`Claude Vault/`); Claude Code CLI branch defaults to `~/dev/knowledge/`. Auto-bootstraps with the personal schema. Never re-asks. Called by reflection-session, business-mentoring, news-digest, fintech-legal-triage, and personal-coach:onboarding as the single source of truth for vault bootstrapping.
- **`vault-companion-recall`** sub-skill under `vault-librarian` (new). Takes a topic, returns up to 5 relevant pages with one-line excerpts. Delegates to `obsidian-wiki:ask` when the upstream plugin is installed; `grep` fallback over the personal-schema category directories otherwise. Read-only. Silent on empty.
- **`vault-companion-append`** sub-skill under `vault-librarian` (new). Takes a vault-handle, category, and body, writes to `<vault>/<category>/YYYY-MM-DD-<slug>.md`, appends `log.md`. Optionally chains `obsidian-wiki:ingest` based on a session-persisted `auto-ingest-pref` (`ask-each` default, `yes-pref`, `no-pref`). Schema-validates against `CLAUDE.md` — refuses categories not in the vault's schema.
- **`Phase 0.5 (or 1.5) — Vault Recall`** in all four substantive skills. Each retrofitted skill scans the vault for related prior pages before its main work — fintech-legal-triage on (jurisdiction × activity) cells, reflection-session on the user's opening sentence (asks before surfacing on tender topics), business-mentoring on decision name with `Decisions`/`Business` category bias (surfaces prior grades as feedback signal), news-digest on each open-question watchlist item (continuity prose, not cold starts).
- **`News/` category** in the personal schema (`vault-CLAUDE-personal.md`). Personal vault now has 9 category dirs, not 8. News-digest frontmatter schema added.
- **Wikilink enrichment** in every retrofitted skill's save phase. Named regulations (`[[MiCA]]`, `[[PSD2]]`, `[[GDPR]]`), stakeholders, companies, and watchlist topics are wrapped in `[[wikilink]]` form when written to the vault, so cross-references accumulate as backlinks across the vault's growth.

### Changed
- **`vault-librarian` 0.1.0 → 0.2.0.** Plugin upgraded from a thin one-command shell to the substrate plugin for basic-harness. Description rewritten. Three new sub-skills shipped (ensure / recall / append). The user-facing `/vault-librarian:bootstrap-vault` command's frontmatter and prose now distinguish the explicit-setup path (this command, 4-persona choice) from the auto-bootstrap path (vault-companion-ensure, personal schema default).
- **`personal-coach` 0.3.0 → 0.4.0.** `reflection-session` and `business-mentoring` retrofitted as vault-citizens. Onboarding Step 3 (vault setup sub-step) no longer carries inline bootstrap logic — delegates to vault-companion-ensure. The v0.4.0 promise ("auto-bootstraps its own notes vault on first save") is preserved; the implementation moves out of personal-coach into the shared substrate.
- **`news-digest` 0.1.0 → 0.2.0.** `news-digest` retrofitted as a vault-citizen. Phase 1.5 vault recall on each open-question watchlist item turns recurring topics into continuity threads across the News/ folder.
- **`fintech-legal-advisor` 0.3.0 → 0.4.0.** `fintech-legal-triage` retrofitted as a vault-citizen. Phase 0.5 vault recall surfaces prior triages on the same (jurisdiction × activity) cell — a recurring EU/payments shop builds up a shared `[[MiCA]]` / `[[PSD2]]` backlink graph over time.
- **Marketplace 0.6.0 → 0.7.0.** Top-level description names the shared notes-vault substrate. All four touched plugins' marketplace entries describe the vault-citizen behavior in their pitch.

### Rationale
The user's spec was *"obsidian-wiki should engage for any conversation the user has with Cowork, not only fintech. Auto-create vault in a Cowork-accessible directory. Auto-bootstrap the vault while the user communicates with Cowork. Build it up from every conversation. Make vault knowledge kick in when appropriate in dialog context."* The Cowork plugin model can't deliver this with hooks ([anthropics/claude-code#27398](https://github.com/anthropics/claude-code/issues/27398) — plugin-scope hooks don't fire in Cowork), and a broad-trigger "global observer" skill competes-and-loses against specific skills in Cowork's matcher. The honest Cowork pattern is "every substantive skill owns its own vault touch-points." That's what this release delivers — same observable outcome (the vault grows from every conversation that triggers a substantive skill; relevant prior knowledge kicks in before each substantive turn), Cowork-honest implementation.

### Plan
Full plan and per-stage gates: `docs/plans/vault-as-substrate-2026-05-15.md`.

---

## v0.6.0 — 2026-05-15

Soft-dependency wiring between `fintech-legal-advisor` and Anthropic's Claude for Legal — the plugin now declares the relationship and guides the user to install Claude for Legal from Cowork's plugin browser at the right moments.

### Added
- **`fintech-legal-advisor` declares `regulatory-legal` from the `claude-for-legal` marketplace as a soft dependency** in `plugin.json` `dependencies`. Claude Code CLI v2.1.110+ auto-installs declared dependencies on `/plugin install`; Cowork's dependency-resolution behavior for cross-marketplace declarations is undocumented as of release, so the declaration is best-effort + future-proof, not a hard contract. The basic-harness root `marketplace.json` whitelists `claude-for-legal` in `allowCrossMarketplaceDependenciesOn`.
- **Detect-and-guide install nudge in `fintech-legal-triage`** (new Phase -1, fires before Phase 0 jurisdiction). On first invocation, if no Claude for Legal connector is detected in the session, the skill prints a one-shot pointer to Cowork → Customize → Browse plugins → Legal, records the shown date to `~/.claude/fintech-legal-advisor.local.md`, and continues without blocking. The nudge does not repeat.
- **Step 0.5 install nudge in `/fintech-legal-advisor:setup-legal-triage-routine`** (between the privacy gate and source selection). When wiring the Routine, if no Claude for Legal connector is detected, the user is offered `continue` / `pause to install Claude for Legal first` / `cancel`. Installing first unlocks the Westlaw / Practical Law / Box / iManage / NetDocuments / Docusign options in Step 1; skipping degrades gracefully to Drive + `WebFetch`.
- **README install section** now lists Claude for Legal as recommended Step 1 with the explicit Cowork path, and `fintech-legal-advisor` as Step 2 — with a clear "skipping Step 1 is OK, but lossy" footnote.

### Changed
- **Plugin versions:** `fintech-legal-advisor` 0.2.0 → 0.3.0.
- **Marketplace version:** 0.5.0 → 0.6.0; tightened the `fintech-legal-advisor` marketplace entry description to surface the Claude for Legal recommendation and the declared soft dependency.

### Rationale
The plugin always positioned as a fintech-specific complement to Claude for Legal, but the install relationship was buried in prose. With Cowork's UI-only plugin install and no documented cross-marketplace auto-install path, the only reliable way to land Claude for Legal alongside this plugin is to (a) declare the dependency for any future-Cowork or CLI-side auto-resolution, and (b) actively guide the user to the **Customize → Browse plugins → Legal** path at the two natural triggering moments (first triage, Routine setup). Citation grounding stops being optional for production fintech triage; the install nudges make that the default behavior.

---

## v0.5.0 — 2026-05-14

Split `personal-coach` into three plugins: a slimmer `personal-coach` plus two new companions.

### Added
- **`news-digest`** plugin (new). Skills `news-preferences` + `news-digest`, command `/news-digest:setup-news-digest`, and the news-digest Cowork Routine template at `plugins/news-digest/docs/routines/`. Standalone — works fine without `personal-coach`; integrates softly with `morning-briefing` (briefing pairs with the digest) and `personal-profile` (profile may propose news-preferences edits) when both are present.
- **`fintech-legal-advisor`** plugin (new, v0.2.0). Skill `fintech-legal-triage`, Opus-pinned subagent `fintech-legal-analyst`, command `/fintech-legal-advisor:setup-legal-triage-routine`, and the document-source-watch Routine template at `plugins/fintech-legal-advisor/docs/routines/`. The plugin treats Cowork Routines as the primary deployment surface for non-sensitive triage volume; the interactive skill remains the safety valve for confidential documents. The setup command has a mandatory privacy gate (Step 0) and refuses to wire a Routine for sensitive-content folders.

  **Claude for Legal integration** (Anthropic launched Claude for Legal 2026-05-12, [TechCrunch](https://techcrunch.com/2026/05/12/the-ai-legal-services-industry-is-heating-up-anthropic-is-getting-in-on-the-action/) / [ABA Journal](https://www.abajournal.com/news/article/anthropic-launches-claude-for-legal-giving-lawyers-20-new-program-integrations-and-12-practice-area-plugins)): the plugin positions as a fintech-specific complement to Claude for Legal's general practice-area plugins (commercial / corporate / employment / privacy / IP / litigation — none of them fintech). When the user has Claude for Legal MCP connectors granted, the plugin prefers them: Westlaw / Practical Law / CoCounsel for live primary-law verification (Phase 1 cell-match → Phase 3 issue list); Box / iManage / NetDocuments / Docusign as contract sources alongside Drive (interactive skill + Routine setup command both accept any of them); Microsoft Word for optional tracked-change output. **Citation grounding is hardened into a non-negotiable hard rule** in the skill, the agent, and the Routine prompt: every named regulation must be verified live this run, or be flagged `confidence: low pending verification`. The plugin runs fine without Claude for Legal, falling back to `WebFetch` against EUR-Lex / FCA / FinCEN / MAS / CBR / DFSA / FSRA / VARA / CBUAE.

### Changed
- **`personal-coach` slimmed.** The `news-digest`, `news-preferences`, and `fintech-legal-triage` skills, the `fintech-legal-analyst` agent, and the `setup-news-digest` command moved out into the two new companion plugins. The personal-coach onboarding opening message drops the news + fintech moments; Tracks C and D now hand off to the companion plugins instead of running internally. Profile, business-mentoring, and morning-briefing soft-reference the companions where applicable but never hard-depend.
- **Plugin versions:** `personal-coach` 0.2.0 → 0.3.0; `news-digest` and `fintech-legal-advisor` debut at 0.1.0.
- **Marketplace version:** 0.4.0 → 0.5.0; added two plugin entries; trimmed `personal-coach` description.
- **Welcome marketplace-tour** updated to surface seven plugins (was five), with explicit branches for "give me my news" → `news-digest` and "review this contract" → `fintech-legal-advisor`.
- **Docs reshuffle.** The two Routine templates moved out of `docs/personal-coach-routines/` into their respective plugin trees. `docs/personal-coach-routines/README.md` updated to reflect what stayed (morning-briefing, decision-grading) and where the moved ones went.

### Rationale
News digest and fintech legal triage were always plugin-shaped: each has its own substrate (preferences file, jurisdiction profile), its own deployment surface (digest schedule, Drive-folder watch), and its own hard limits. Bundling them inside `personal-coach` forced users who only wanted one of the four faces to install all four. The split also lets `fintech-legal-advisor` lean fully into the Cowork-Routine + Drive surface as a first-class deployment path, rather than tucking it away as an optional add-on.

---

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
