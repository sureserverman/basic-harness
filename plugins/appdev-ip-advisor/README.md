# appdev-ip-advisor

App-development **IP issue-spotter** for Claude Cowork. The plugin exists because most IP exposure in app development doesn't come from a deliberate copy — it comes from a missing question. Someone picked an app name that's too close to a competitor's filed mark, embedded a font under a desktop-only licence, used a sound effect from a YouTube tutorial, modelled a camera-app UI on a real Leica dial without checking whose design that is, or shipped an AI-generated illustration in the marketing splash. The plugin's job is to surface the questions that belong on the agenda for the company's lawyer, and the IP regimes they live under, **before** the App Store listing goes live.

**Not legal advice. Not a substitute for outside counsel.** Every output the plugin produces ends with a non-negotiable take-to-counsel block.

This plugin works standalone. If you also have `personal-coach` installed, the IP-triage results are written to the same vault path as your other personal notes (`<vault>/Legal/`); if you don't, they live under `~/.claude/appdev-ip-triage/`.

## What it covers

The skill walks an issue-checklist across these IP regimes for an app-development context:

| Regime | Typical app-dev question |
|---|---|
| **Trademark / brand naming** | Is the app name (or logo, or tagline) confusingly similar to a filed mark in your target jurisdictions? Do you need to file in Class 9 / 42 / 38? Madrid Protocol vs. national filings? |
| **Trade dress / visual-style mimicry** | Does your UI copy distinctive elements of another product — a competing app's onboarding flow, a physical camera's dial layout, a console's button glyphs, a recognisable game's HUD? Trade dress + design patent + copyright in skeuomorphic art can all attach to the same UI. |
| **Copyright — icons, illustrations, fonts** | Stock-asset licence terms (Iconfinder / Noun Project / Streamline / Lottiefiles); font embedding rights (desktop vs. web vs. app); image licences (Getty / Shutterstock / Unsplash terms vary). |
| **Copyright — music and sound effects** | BGM, button-tap sounds, notification chimes, app-launch stingers. Sync rights vs. mechanical rights; whether your music library's licence covers in-app use; whether you're inadvertently using a trademarked sound (NBC chime, Skype tone, T-Mobile jingle). |
| **Copyright — code** | Snippets pulled from GitHub / Stack Overflow / AI assistants; obligations of the licence on each dependency; AI-generated code from Copilot / Claude / Cursor and the upstream training-data debate. |
| **Open-source licence compliance** | GPL / AGPL contagion when statically linked into a closed-source app; LGPL static-vs-dynamic linking; AGPL killing SaaS/server-side use; MIT / BSD / Apache-2.0 attribution in the in-app Acknowledgements screen; "source-available" non-OSS licences (BSL, SSPL, Elastic License) masquerading as open source. |
| **Right of publicity / likeness / voice** | Celebrity names or likenesses in marketing; real people in user-generated content (face filters, deepfakes); voice clones that sound like a specific actor or singer. State-by-state in the US (CA / NY recognise post-mortem rights, many don't); EU has GDPR overlay; UK has passing-off + a developing image-rights doctrine. |
| **AI-generated content** | Copyright status of AI images / video / music (US Copyright Office: unprotectable without human authorship; EU broadly similar); training-data IP claims; AI Act Article 50 (EU) disclosure-of-AI-content rules in force 2026-08-02; commercial-use grants in OpenAI / Anthropic / Stability / Midjourney terms. |
| **App Store / Play Store IP rules** | Apple App Store Review Guideline 5.2 (Intellectual Property); Google Play "Deceptive Behavior" / "Impersonation" policies; pre-emptive removal risk when you ship near a Sherlocked feature. |
| **Acquired third-party components** | UI kits, icon packs, sound libraries, white-label / template starter apps, SDKs — every one has its own IP grant and its own attribution requirement. |

**Out of scope:** software patents. App-dev patent thickets (gesture patents, in-app-purchase patents, AR/VR specific patents) need a registered patent attorney, not an issue-spotter. The skill will explicitly refuse and route to "this is a patent question — out of scope; surface to a registered patent attorney" if the user raises one.

## Relationship to Anthropic's Claude for Legal

Anthropic launched [Claude for Legal](https://www.anthropic.com/) on 2026-05-12 — a Cowork-centred set of 20 MCP connectors and 12 practice-area plugins. The **`intellectual-property`** practice-area plugin covers general IP work (filing strategy, portfolio management, due diligence, opposition / cancellation proceedings) but does **not** ship app-development-specific checklists — App Store guideline 5.2, OSS-licence contagion at static-link time, sound-effect chain-of-title in a `.caf` bundle, AI-content disclosure under AI Act Art. 50. `appdev-ip-advisor` is the app-developer complement.

Claude for Legal is built around **citation grounding**: trademark filings, copyright case law, and statute references must come from live verified sources — Westlaw, Practical Law, CourtListener, the Free Law Project, USPTO TESS / EUIPO eSearch / WIPO Madrid Monitor / UKIPO databases — rather than from the model's training data. When the Claude for Legal MCP connectors are granted to your Cowork session, this plugin **prefers them** over generic `WebFetch` for every phase it touches:

| Phase | Without Claude for Legal | With Claude for Legal granted |
|---|---|---|
| Phase 1 — match (jurisdiction × asset-type × use) cell | Anchor checklist the plugin ships with. | Same anchor checklist + verify every named statute / treaty / app-store-policy line against **Westlaw / Practical Law** (or the relevant primary-law connector for the jurisdiction). Stale citation? Flagged and replaced; never silently used. |
| Phase 2 — walk the issue checklist | User answers from the asset description / screenshot / draft marketing copy. | Asset bundle pulled directly from **Box / iManage / NetDocuments / Docusign / Google Drive** — whichever the user has granted. Checklist walks against the actual files (icon pack licence PDFs, font EULAs, asset metadata), not summaries. |
| Phase 3 — output issue list | Markdown file, every issue cites the statute / treaty / app-store guideline by name. | Same Markdown file + (optional) **Microsoft Word** tracked-change pass against draft marketing copy / T&Cs, surfacing the issue list as in-line comments and proposed redlines for attorney review before acceptance. |
| Trademark clearance | Plugin asks the user to manually check USPTO TESS / EUIPO eSearch / WIPO Madrid Monitor. | Plugin asks Claude for Legal to run the clearance against the same databases via the connector, where available — and otherwise the same manual hand-back. Either way, **a clearance search is never a freedom-to-use opinion** — that's an IP attorney's call. |
| Citation rule | "Anchor list; additions require a citation." | **Hardened**: every regulation / case / app-store-policy citation must resolve against a granted primary-law connector (Westlaw / Practical Law / EUR-Lex via WebFetch / Apple / Google policy docs) or carry `confidence: low pending verification`. No exceptions for "the model is sure". |

The plugin runs fine without Claude for Legal — it falls back to `WebFetch` against EUR-Lex, USPTO, EUIPO, WIPO, UKIPO, Rospatent, IPOS, MOEM, the App Store Review Guidelines, the Play Store Developer Policy Center. But the citation-grounding discipline is the most important part of Claude for Legal for IP work, where a stale precedent or a guideline section number that moved last quarter can produce a confidently wrong issue list.

## Install

### Step 1 (strongly recommended) — install Claude for Legal first

In Cowork: **Customize → Browse plugins → Legal → Install** (or visit [claude.com/plugins/legal](https://claude.com/plugins/legal)). That gives this plugin the Westlaw / Practical Law / CoCounsel primary-law connectors, the CourtListener / Free Law Project case-law connectors, and the Box / iManage / NetDocuments / Docusign / Microsoft Word document connectors. The plugin's `plugin.json` declares `intellectual-property` (a Claude for Legal practice-area plugin) as a soft dependency; if your Cowork build auto-installs declared cross-marketplace dependencies, it will handle this for you, but the safe move is to install Claude for Legal explicitly from the Browse plugins UI first.

### Step 2 — install `appdev-ip-advisor`

From the top-level basic-harness GitHub release: download `basic-harness-<version>.zip`, unzip, and upload `appdev-ip-advisor-<version>.zip` via Cowork → **Customize** → **Browse plugins** → **upload custom plugin file**. See the [top-level basic-harness README](../../README.md#install) for the canonical install flow.

### Skipping Step 1 is OK, but lossy

`appdev-ip-advisor` runs without Claude for Legal. It falls back to `WebFetch` against USPTO / EUIPO / WIPO / UKIPO / Rospatent / IPOS / MOEM and the live App Store / Play Store policy pages, and the asset source shrinks to Google Drive only. The triage methodology is the same; the citation confidence and the source surface shrink. The first time you invoke the `appdev-ip-triage` skill (or run `/appdev-ip-advisor:setup-ip-triage-routine`), the plugin will print a one-shot install nudge pointing you back to this step.

## Two surfaces, same engine

The same Opus-pinned `appdev-ip-analyst` subagent powers both surfaces. Pick the one that matches the asset's privacy posture and your throughput:

| Surface | When to use | Privacy posture |
|---|---|---|
| **Cowork Routine** — set up via `/appdev-ip-advisor:setup-ip-triage-routine`. The Routine watches an inbound folder (Google Drive / Box / iManage / NetDocuments / Docusign envelope feed) and triages every new asset bundle automatically — UI screenshots, icon packs, font bundles with EULAs, marketing copy drafts, T&Cs drafts, third-party licence packs. | Marketing assets, public-facing UI mocks, third-party licence bundles, generic app-name shortlists. High throughput, low touch. | Asset text and image OCR pass through Anthropic's cloud during Routine execution. Acceptable only for non-sensitive assets. |
| **Interactive skill** — invoke `appdev-ip-triage` in Cowork ("can we name our app this", "review this icon set's licence", "did we just clone Halide's UI", "is this AI illustration safe to ship", "is our GPL-bundled SDK safe in a closed-source app"). Run with any of the document connectors granted ad-hoc to pull a specific file in. | Pre-launch app names under embargo, unannounced features, M&A asset-portfolio diligence, internal design docs. You decide doc-by-doc whether to grant access. | The user controls each call; no standing cloud watch on a folder. |

The Cowork Routine is the **primary deployment surface** for high-volume work (a steady inflow of third-party asset bundles into a Drive folder is the canonical use case). The interactive skill is the safety valve for sensitive material.

## Start here

If you have a steady inflow of non-sensitive asset bundles (icon packs you're evaluating, font EULAs, sound libraries, draft marketing copy) landing in a Drive folder, run:

```text
/appdev-ip-advisor:setup-ip-triage-routine
```

It walks you through the privacy gate (mandatory), the Drive folder pair, the jurisdiction context, your distribution channels (App Store / Play Store / web / desktop / direct sideload), and the Cowork **Routines** UI to wire the watch. The Routine fires on file-created events and writes a structured issue list to a paired output folder, one file per asset bundle, before you ship.

If you don't have that kind of inflow — or the assets are confidential — invoke the analyst interactively. Just say "is this app name safe", "review this icon-pack licence", "did we just clone the Halide camera UI", or "is this AI splash image OK to use" in Cowork, and `appdev-ip-triage` fires. Grant the Drive connector when it asks, point at the file, get the issue list.

## Vault integration

`appdev-ip-triage` is a **vault-citizen** — it uses the shared `vault-companion` surface from `vault-librarian`:

- **Phase 0.5 — Vault Recall** runs after jurisdiction and asset-type are named. It calls `vault-companion-recall` with topic = `<jurisdiction> + <asset-type> + IP`. If you've previously triaged the same (jurisdiction × asset-type) cell — e.g., another US/icon-pack triage, or another EU/trade-dress review — the prior issue list and decision history surface as context before the new triage walks the checklist. Returns silently if the vault has no matches.
- **Phase 4 — Save** delegates to `vault-companion-append` with `category=Legal`. Output writes to `<vault>/Legal/YYYY-MM-DD-<slug>.md`, with every named statute / treaty / app-store-policy section wrapped in `[[wikilink]]` form so `[[Lanham Act §43(a)]]` / `[[GDPR]]` / `[[AI Act Art. 50]]` / `[[App Store Review Guideline 5.2]]` / `[[GPL-3.0]]` / `[[AGPL-3.0]]` accumulate backlinks across your `Legal/` folder over time. Frontmatter carries `type: ip-triage, jurisdictions, asset_type, distribution_channels, issue_count, regimes`.
- **Fallback.** If you've declined a vault (or `vault-librarian` isn't installed), the triage falls back to `~/.claude/appdev-ip-triage/YYYY-MM-DD-<slug>.md` via plain file write. Same triage quality, no cross-linking.

Install `vault-librarian` alongside this plugin to get the vault-companion surface. It auto-bootstraps a personal-schema vault on first triage save — no separate command needed.

The `Legal/` category is **shared** with `fintech-legal-advisor`. The two plugins' triage logs sit side by side in the same folder; the frontmatter `type` field (`legal-triage` vs. `ip-triage`) and the wikilink surface (regulation names vs. IP-statute / app-store-policy names) distinguish them.

## Skill

| Skill | Purpose |
|---|---|
| `appdev-ip-triage` | Issue-spotter walked through Phase 0 (jurisdiction + asset type + distribution channels) → Phase 0.5 (vault recall, optional) → Phase 1 (match the (jurisdiction × asset-type × channel) cell) → Phase 2 (walk the issue checklist) → Phase 3 (output structured issue list) → Phase 4 (save via vault-companion-append) → Phase 5 (take-to-counsel block). Covers US (federal + key state right-of-publicity), EU (EUTM + national copyright + AI Act), UK (post-Brexit UKTM + CDPA), Russia (Civil Code Part IV + Rospatent), Singapore (TMA 2005 + Copyright Act 2021), UAE (Federal Decree-Law 36/2021 + DIFC + ADGM). Regimes: trademark, trade dress, copyright (icons/fonts/music/code/photo/AI), open-source licence compliance, right of publicity, app-store IP rules. |

## Subagent

| Agent | Model | Role |
|---|---|---|
| `appdev-ip-analyst` | **Opus** | The careful IP tier. Pinned to Opus because the cost of IP misreads (App Store rejection, takedown, opposition, a cease-and-desist on the launch-day press release) is high enough that the careful tier is worth the spend. Asks jurisdiction + asset type + distribution channel first, refuses to proceed without them. Outputs an issue list, never a verdict. Refuses to opine on software patents — routes to a patent attorney. Ends every output with a take-to-counsel block. |

## Slash command

| Command | What it does |
|---|---|
| `/appdev-ip-advisor:setup-ip-triage-routine` | Wire `appdev-ip-triage` into a Cowork Routine that watches an inbound asset / spec / licence-bundle source (Google Drive / Box / iManage / NetDocuments / Docusign envelope feed). Includes a mandatory privacy gate (Step 0) that refuses to proceed without explicit non-sensitive-only confirmation. Optionally emits Word tracked-changes output if the Microsoft connector is granted (for draft marketing copy / T&Cs / acknowledgements screens). |

## Where things live

| Artifact | Path (vault) | Path (no vault) |
|---|---|---|
| Triage outputs (interactive) | `<vault>/Legal/YYYY-MM-DD-<slug>.md` | `~/.claude/appdev-ip-triage/YYYY-MM-DD-<slug>.md` |
| Triage outputs (Routine) | Output folder you named at setup (in whichever document system you used as the source — Drive / Box / iManage / NetDocuments) | — |
| Tracked-change Word output (optional) | Saved next to the source draft in the same document system, suffixed `-redline-<YYYY-MM-DD>.docx` | — |
| Plugin state | — | `~/.claude/appdev-ip-advisor.local.md` |

## Cowork Routine template

A copy-paste template lives at `docs/routines/ip-triage-on-drive-routine.md`. The slash command builds on this template — if you want to wire the Routine by hand instead of through the command, the template plus the prompt block is self-contained. **Read the privacy header before installing.**

## Hard limits

- **No "is this safe to ship" yes/no answers.** The plugin returns issue lists, not verdicts.
- **No software-patent analysis.** Patent freedom-to-operate searches, infringement opinions, and patent-clearance work are out of scope. The plugin will refuse and route to a registered patent attorney.
- **No trademark freedom-to-use opinions.** The plugin can flag that a name looks too close to an existing mark; only an IP attorney issues a freedom-to-use opinion.
- **No drafting of final-form regulated content** (final App Store metadata, final T&Cs, final EULAs, final Acknowledgements screens, final OSS-attribution text for a regulated product). The plugin can sketch a structure or surface a missing clause; final text needs counsel sign-off.
- **No case-law interpretation.** Citations to specific cases (Sony v. Universal, Authors Guild v. Google, Star Athletica v. Varsity Brands, Andy Warhol Foundation v. Goldsmith, the pending generative-AI cases) are read for *naming* — the analyst surfaces the case name as a flag for counsel, never as a holding.
- **No DMCA-strategy advice** (whether to issue or contest a takedown). That's outside-counsel work.
- **No evasion advice.** "Set up in Country X to avoid GPL contagion / right-of-publicity rules" — refuse and explain.
- **Take-to-counsel block on every output.** No exceptions. No softening.
- **Jurisdiction + asset type + distribution channel first, every time.** No IP analysis without all three — they bound the relevant law.

## Sources and rationale

The skill and the subagent cite their methodology so the issue checklists aren't arbitrary. See `skills/appdev-ip-triage/SKILL.md` for the full source list. Anchor references:

- **International treaties** — Berne Convention (1886, as revised); Paris Convention (1883, as revised); TRIPS Agreement (1994); WIPO Copyright Treaty (1996); WIPO Performances and Phonograms Treaty (1996); Madrid Protocol (1989).
- **EU** — Trade Mark Regulation (EU) 2017/1001 (EUTM); Design Regulation (EC) 6/2002; InfoSoc Directive 2001/29/EC; DSM Directive (EU) 2019/790; Database Directive 96/9/EC; AI Act (Regulation (EU) 2024/1689), esp. Art. 50 (AI-generated-content disclosure); GDPR Art. 6/9 overlap with biometric likeness.
- **UK (post-Brexit)** — Trade Marks Act 1994; Copyright, Designs and Patents Act 1988 (CDPA); Registered Designs Act 1949; UK GDPR; UK passing-off doctrine.
- **US (federal)** — Lanham Act (15 USC §§ 1051–1141n), esp. §43(a) on trade dress and false designation; Copyright Act (17 USC), esp. §§ 102, 107 (fair use), 1201 (DMCA anti-circumvention), 512 (DMCA safe harbour); Visual Artists Rights Act (17 USC § 106A); state right-of-publicity statutes — Cal. Civ. Code § 3344 (CA), N.Y. Civ. Rights Law §§ 50/51 (NY).
- **US case anchors** (named, not interpreted) — *Sony v. Universal* (1984), *Two Pesos v. Taco Cabana* (1992), *Wal-Mart v. Samara Bros.* (2000), *Authors Guild v. Google* (2015), *Star Athletica v. Varsity Brands* (2017), *Apple v. Samsung* (2018), *Andy Warhol Foundation v. Goldsmith* (2023), and the pending generative-AI training-data line (*Thomson Reuters v. Ross*, *Andersen v. Stability AI*, *NYT v. OpenAI*).
- **Russia** — Civil Code Part IV (Federal Law 230-FZ, 2006), Chs. 70 (copyright), 76 (trademarks); Rospatent procedures; 152-FZ overlay on likeness data.
- **Singapore** — Trade Marks Act 2005; Copyright Act 2021; PDPA 2012 overlay.
- **UAE** — Federal Decree-Law 36/2021 on Trademarks; Federal Decree-Law 38/2021 on Copyright and Neighbouring Rights; DIFC Law 4/2021 (data protection overlay for likeness); ADGM IP regime.
- **App-store policy** — Apple App Store Review Guidelines, esp. § 5.2 (Intellectual Property); Google Play Developer Policy Center — Impersonation, Intellectual Property, Deceptive Behavior.
- **OSS licence canon** — GPL-2.0 / GPL-3.0 / LGPL-2.1 / LGPL-3.0 / AGPL-3.0 (FSF texts); MIT / BSD-2-Clause / BSD-3-Clause / Apache-2.0 / MPL-2.0 / EPL-2.0 (OSI texts); "source-available" non-OSS: Business Source License 1.1, SSPL-1.0, Elastic License 2.0.
- **AI-content policy** — US Copyright Office *Guidance on Works Containing AI-Generated Material* (37 CFR Part 202, 2023, and the 2026 update); EU AI Act Art. 50 (in force 2026-08-02); OpenAI / Anthropic / Stability / Midjourney commercial-use terms (versioned, verify live).
