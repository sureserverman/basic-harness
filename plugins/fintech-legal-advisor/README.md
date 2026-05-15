# fintech-legal-advisor

Fintech regulatory **issue-spotter** for Claude Cowork. The plugin exists because most legal exposure in fintech doesn't come from a single bad clause — it comes from a missing question. The plugin's job is to surface the questions that belong on the agenda for the company's lawyer, and the regulations they live under, before anything ships.

**Not legal advice. Not a substitute for outside counsel.** Every output the plugin produces ends with a non-negotiable take-to-counsel block.

This plugin works standalone. If you also have `personal-coach` installed, the legal-triage results are written to the same vault path as your other personal notes; if you don't, they live under `~/.claude/legal-triage/`.

## Relationship to Anthropic's Claude for Legal

Anthropic launched [Claude for Legal](https://www.anthropic.com/) on 2026-05-12 — a Cowork-centred set of 20 MCP connectors and 12 practice-area plugins (commercial, corporate, employment, privacy, IP, litigation, plus tools for law students and legal clinics). It is built around **citation grounding**: regulation and case-law references must come from live verified sources — Westlaw, Practical Law, CourtListener, the Free Law Project — rather than from the model's training data.

`fintech-legal-advisor` is a **fintech-specific complement** to that offering: the named six practice-area plugins do not cover fintech regulation (PSD2, EMD2, MiCA, AMLR/AMLD6, PSA 2019, VARA, FCA CONC/PRIN/PSRs, FinCEN MSB rules, 161-FZ/115-FZ, etc.), and a generic commercial-law plugin will not have the issue-checklists this plugin ships. When Claude for Legal MCP connectors are granted to your Cowork session, this plugin **prefers them** over generic `WebFetch` for every phase it touches:

| Phase | Without Claude for Legal | With Claude for Legal granted |
|---|---|---|
| Phase 1 — match (jurisdiction × activity) cell | Anchor list of regulations the plugin ships with. | Same anchor list + verify each named regulation against **Westlaw / Practical Law** (or the relevant primary-law connector for the jurisdiction). Stale citation? Flagged and replaced; never silently used. |
| Phase 2 — walk the issue checklist | User answers from the contract or feature description. | Contract pulled directly from **Box / iManage / NetDocuments / Docusign / Google Drive** — whichever the user has granted. Checklist walks against actual clauses, not summaries. |
| Phase 3 — output issue list | Markdown file, every issue cites the regulation by name. | Same Markdown file + (optional) **Microsoft Word** tracked-change pass against the contract draft, surfacing the issue list as in-line comments and proposed redlines for attorney review before acceptance. |
| Citation rule | "Anchor list; additions require a citation." | **Hardened**: regulation citations must resolve against a granted primary-law connector (Westlaw / Practical Law / EUR-Lex via WebFetch / official regulator site) or carry `confidence: low pending verification`. No exceptions for "the model is sure". |

The plugin runs fine without Claude for Legal — it falls back to `WebFetch` against EUR-Lex, FCA, FinCEN, MAS, CBR, DFSA, FSRA, VARA, etc. But the citation-grounding discipline is the most important part of Claude for Legal for fintech work, where a stale regulation reference can produce a confidently wrong issue list. Install Claude for Legal (and grant the connectors the plugin asks for) if you do any meaningful volume of triage.

## Install

### Step 1 (strongly recommended) — install Claude for Legal first

In Cowork: **Customize → Browse plugins → Legal → Install** (or visit [claude.com/plugins/legal](https://claude.com/plugins/legal)). That gives this plugin the Westlaw / Practical Law / CoCounsel primary-law connectors and the Box / iManage / NetDocuments / Docusign / Microsoft Word document connectors. The plugin's `plugin.json` declares `regulatory-legal` (a Claude for Legal practice-area plugin) as a soft dependency; if your Cowork build auto-installs declared cross-marketplace dependencies, it will handle this for you, but at the time of writing that path is undocumented for Cowork — so the safe move is to install Claude for Legal explicitly from the Browse plugins UI before installing this plugin.

### Step 2 — install `fintech-legal-advisor`

From the top-level basic-harness GitHub release: download `basic-harness-<version>.zip`, unzip, and upload `fintech-legal-advisor-<version>.zip` via Cowork → **Customize** → **Browse plugins** → **upload custom plugin file**. See the [top-level basic-harness README](../../README.md#install) for the canonical install flow.

### Skipping Step 1 is OK, but lossy

`fintech-legal-advisor` runs without Claude for Legal. It falls back to `WebFetch` against EUR-Lex / FCA / FinCEN / MAS / CBR / DFSA / FSRA / VARA / CBUAE, and the source connector shrinks to Google Drive only. The triage methodology is the same; the citation confidence and the source surface shrink. The first time you invoke the `fintech-legal-triage` skill (or run `/fintech-legal-advisor:setup-legal-triage-routine`), the plugin will print a one-shot install nudge pointing you back to this step.

## Two surfaces, same engine

The same Opus-pinned `fintech-legal-analyst` subagent powers both surfaces. Pick the one that matches the document's privacy posture:

| Surface | When to use | Privacy posture |
|---|---|---|
| **Cowork Routine** — set up via `/fintech-legal-advisor:setup-legal-triage-routine`. The Routine watches an inbound folder (Google Drive / Box / iManage / NetDocuments / Docusign envelope feed) and triages every new file automatically. | Vendor T&Cs, public-facing partnership templates, routine procurement contracts. High throughput, low touch. | Document text passes through Anthropic's cloud during Routine execution. Acceptable only for non-sensitive contracts. |
| **Interactive skill** — invoke `fintech-legal-triage` in Cowork ("review this contract", "is this GDPR-compliant", "what regs apply to <feature>"). Run with any of the document connectors granted ad-hoc to pull a specific file in. | Client contracts under NDA, M&A documents, regulator correspondence, anything where confidentiality is load-bearing. You decide doc-by-doc whether to grant access. | The user controls each call; no standing cloud watch on a folder. |

The Cowork Routine is the **primary deployment surface** for high-volume work. The interactive skill is the safety valve for sensitive material.

See the [top-level basic-harness README](../../README.md#install) for the canonical install flow (Cowork → Customize → upload zip from this repo's GitHub releases).

## Start here

If you have a steady inflow of non-sensitive contracts (vendor T&Cs, procurement, partnership templates) landing in a Drive folder, run:

```text
/fintech-legal-advisor:setup-legal-triage-routine
```

It walks you through the privacy gate (mandatory), the Drive folder pair, the jurisdiction context, and the Cowork **Routines** UI to wire the watch. The Routine fires on file-created events and writes a structured issue list to a paired output folder, one file per contract, before you sign anything.

If you don't have that kind of inflow — or the documents are confidential — invoke the analyst interactively. Just say "review this contract" or "what regs apply to this feature" in Cowork, and `fintech-legal-triage` fires. Grant the Drive connector when it asks, point at the file, get the issue list.

## Skill

| Skill | Purpose |
|---|---|
| `fintech-legal-triage` | Issue-spotter walked through Phase 0 (jurisdiction) → Phase 1 (match the (jurisdiction × activity) cell) → Phase 2 (walk the issue checklist) → Phase 3 (output structured issue list) → Phase 5 (take-to-counsel block). Covers EU/EEA, UK, US (federal), Russia/EAEU, Singapore, UAE (DIFC / ADGM / Mainland). Categories: KYC/AML, payment licensing, crypto (MiCA/TFR), GDPR, customer T&Cs, marketing / financial promotions. |

## Subagent

| Agent | Model | Role |
|---|---|---|
| `fintech-legal-analyst` | **Opus** | The careful regulatory tier. Pinned to Opus because the cost of regulatory misreads in fintech is high enough that the careful tier is worth the spend. Asks jurisdiction first, refuses to proceed without it. Outputs an issue list, never a verdict. Ends every output with a take-to-counsel block. |

## Slash command

| Command | What it does |
|---|---|
| `/fintech-legal-advisor:setup-legal-triage-routine` | Wire `fintech-legal-triage` into a Cowork Routine that watches an inbound document source (Google Drive / Box / iManage / NetDocuments / Docusign envelope feed). Includes a mandatory privacy gate (Step 0) that refuses to proceed without explicit non-sensitive-only confirmation. Optionally emits Word tracked-changes output if the Microsoft connector is granted. |

## Where things live

| Artifact | Path (vault) | Path (no vault) |
|---|---|---|
| Triage outputs (interactive) | `<vault>/Legal/YYYY-MM-DD-<slug>.md` | `~/.claude/legal-triage/YYYY-MM-DD-<slug>.md` |
| Triage outputs (Routine) | Output folder you named at setup (in whichever document system you used as the source — Drive / Box / iManage / NetDocuments) | — |
| Tracked-change Word output (optional) | Saved next to the source contract in the same document system, suffixed `-redline-<YYYY-MM-DD>.docx` | — |
| Plugin state | — | `~/.claude/fintech-legal-advisor.local.md` |

## Cowork Routine template

A copy-paste template lives at `docs/routines/legal-triage-on-drive-routine.md`. The slash command builds on this template — if you want to wire the Routine by hand instead of through the command, the template plus the prompt block is self-contained. **Read the privacy header before installing.**

## Hard limits

- **No "is this legal" yes/no answers.** The plugin returns issue lists, not verdicts.
- **No drafting of regulated documents in final form.** The plugin can sketch a structure or surface a missing clause. Final ToS, KIDs, prospectuses, regulator filings need a lawyer's signoff.
- **No case-law interpretation.** Citations to specific cases are out of scope; the plugin stays at the regulation level.
- **No enforcement-strategy advice.** What to say to a regulator under investigation is privileged territory.
- **No evasion advice.** "Set up in <jurisdiction> to avoid AML" — refuse and explain.
- **Take-to-counsel block on every output.** No exceptions. No softening.
- **Jurisdiction first, every time.** No regulatory analysis without (a) where the company is incorporated, (b) where the customers sit, (c) what the activity is.

## Sources and rationale

The skill and the subagent cite their methodology so the issue checklists aren't arbitrary. See `skills/fintech-legal-triage/SKILL.md` for the full source list. Anchor references:

- **EU payments** — PSD2 (Directive (EU) 2015/2366); EMD2 (Directive 2009/110/EC); EBA Guidelines.
- **EU AML** — 5AMLD (2018/843); 6AMLD (2018/1673); EU AML Package 2024 (AMLR + AMLD6 + AMLA).
- **EU crypto** — MiCA (Regulation (EU) 2023/1114); TFR (Regulation (EU) 2023/1113); DAC8 (Directive (EU) 2023/2226).
- **EU data** — GDPR (Regulation (EU) 2016/679); EDPB Guidelines (post-Schrems II).
- **UK** — FSMA 2000; PSRs 2017; EMRs 2011; MLRs 2017; FCA Handbook (CONC, DISP, PRIN, COBS); Consumer Duty (PRIN 2A, July 2023).
- **US** — Bank Secrecy Act; 31 CFR ch. X; OFAC; CFPB enforcement priorities; SEC v. *Howey* (1946) framework.
- **Russia** — 161-FZ; 115-FZ; 152-FZ; 259-FZ; 31-FZ.
- **Singapore** — Payment Services Act 2019; PDPA 2012; MAS notices.
- **UAE** — DFSA Rulebook (DIFC); FSRA Rulebook + Crypto Asset Framework (ADGM); VARA Regulations (Dubai); CBUAE.
- **AML/KYC fundamentals** — FATF *International Standards on Combating Money Laundering* (40 Recommendations, latest revision); Basel CDD principles.
