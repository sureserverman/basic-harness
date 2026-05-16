# appdev-ip-advisor

App-development **IP issue-spotter** AND **app-store pre-submission / rejection-triage reviewer** for Claude Cowork. The plugin exists because most exposure in app development doesn't come from a deliberate violation — it comes from a missing question. Someone picked an app name that's too close to a competitor's filed mark, embedded a font under a desktop-only licence, used a sound effect from a YouTube tutorial, modelled a camera-app UI on a real Leica dial without checking whose design that is, shipped an AI-generated illustration in the marketing splash, added a Required Reason API to an iOS framework without updating `PrivacyInfo.xcprivacy`, drifted the Google Play Data Safety form during an SDK upgrade, forgot the in-app Account Deletion requirement under Apple §5.1.1(v), or shipped a child-targeted app with third-party analytics still wired in. The plugin's job is to surface the questions that belong on the agenda for the company's lawyer **and** for the company's app-store policy lead, and the IP regimes / store-policy clauses they live under, **before** the App Store / Play Store / Microsoft Store / etc. submission goes in.

**Not legal advice. Not a substitute for outside counsel. Not a freedom-to-ship guarantee.** Every output the plugin produces ends with a non-negotiable take-to-counsel block (IP surface) or take-to-store-policy-review block (app-store surface).

This plugin works standalone. If you also have `personal-coach` installed, IP-triage results are written to `<vault>/Legal/` and app-store-policy reviews are written to `<vault>/AppStore/`; if you don't, they live under `~/.claude/appdev-ip-triage/` and `~/.claude/appdev-app-store-rules/` respectively.

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

## What the app-store-policy surface covers

The IP table above is one half. The other half is store-policy review: pre-submission and reactive rejection-triage across the actual storefronts. The store-policy surface is run by the `appdev-app-store-rules` skill (pre-submission) and the `appdev-app-store-rejection-triage` skill (reactive when a rejection notice arrives), both powered by the `appdev-app-store-reviewer` Opus-pinned subagent.

| Store | What the reviewer checks (anchor list — every cited section is live-verified per run) |
|---|---|
| **Apple App Store** (iOS / iPadOS / watchOS / tvOS / visionOS) | App Store Review Guidelines §1–§5 (Safety, Performance, Business, Design, Legal); App Privacy nutrition label; **Privacy Manifest (`PrivacyInfo.xcprivacy`) and the 5 Required Reason API categories**; App Tracking Transparency (ATT); Sign in with Apple §4.8 (mandatory if any third-party sign-in is offered); In-App Purchase §3.1 with the Reader-app / Music-streaming / External Link Account / DMA Alternative Marketplace carve-outs; subscription disclosure §3.1.2; **in-app Account Deletion §5.1.1(v)** (web-only deletion is not sufficient); Generative AI App requirements (content moderation + age 17+ for unmoderated outputs); TestFlight rules; visionOS HIG; watchOS complications; CarPlay; tvOS focus engine. |
| **Mac App Store** | Same Guidelines + sandboxing entitlements + notarisation. |
| **Google Play Store** | Developer Policy Center; **Data Safety form** (per-SDK / per-data-type matrix — frequent rejection source when SDKs drift); Account Deletion (in-app + accessible web URL); **Sensitive Permissions declarations** (SMS / Call Log / Location-in-background / Accessibility Service / Notification Listener / VPN Service / All-Files-Access / Health Connect); Play Billing §4.1 with Reader-app exemption and User Choice Billing in eligible markets; Android Vitals (ANR + crash rate bad-behaviour thresholds, verified live each run); Families Policy (Designed for Families opt-in, SDK self-certification, third-party-ad ban for child-targeted apps); AI-generated content policy. |
| **Microsoft Store** (Windows) | Microsoft Store Policies; capability declarations (restricted capabilities require business justification at submission); privacy statement URL requirement; AppX/MSIX signing chain; Partner Center certification report parsing. |
| **Amazon Appstore** | Fire OS-specific overlays (no GMS); Amazon in-app billing on Fire OS; restricted-content policy. |
| **Samsung Galaxy Store** | Watch / DeX / S Pen / Knox-specific surfaces; Samsung Seller Office certification feedback parsing. |
| **Huawei AppGallery** | HMS-vs-GMS replacement for the China surface; AppGallery Review Guideline overlays for Chinese content rules. |
| **F-Droid** | FOSS-only Inclusion Policy; **Anti-Features tagging** (NonFreeNet, Tracking, Ads, NonFreeDep, NonFreeAssets, etc. — missing an Anti-Feature that applies is a rejection); reproducible-build requirement (the build server's output must match the developer's locally-signed APK); no Play Services hard-dep. |
| **Flathub** (Linux Flatpak) | Manifest validation; reverse-DNS app ID with owner-controlled domain; sandbox-permission justification (`--filesystem=home` and other broad scopes). |
| **Snap Store** (Ubuntu / Linux) | **Strict-vs-classic confinement** (classic requires manual review with a written engineering justification); snapd interfaces; auto-update tolerance (users don't opt out of snap updates). |
| **Steam** | Steam Direct ($100 fee + content survey); revenue tiers (30% / 25% / 20%); content policy; adult-content gating; Workshop UGC moderation. |
| **Epic Games Store** | 12% revenue share; content rating; Epic Online Services optional integration. |
| **EU-DMA alternative iOS marketplaces** (AltStore PAL, Setapp, Epic Games Store iOS) | Apple **notarisation** (separate from App Store review — a smaller technical-review pass); **Core Technology Fee** structure; DMA-specific entitlements. |
| **Cross-store consistency** | Apple privacy nutrition label vs Google Play Data Safety vs Microsoft Store privacy statement (same SDKs should produce consistent disclosures); Apple §5.1.1(v) vs Google Play Account Deletion (build once with both in mind); Apple §3.1.1 vs Google Play §4.1 (carve-outs differ — don't conflate); age ratings (Apple questionnaire vs IARC); sensitive-permission rationale strings; AI-content disclosure architecture (strictest of Apple Generative AI App rules / Play AI-content / EU AI Act Art. 50 wins). |

**Legal overlays the reviewer flags** but routes to counsel: GDPR Art. 17 (erasure); GDPR Art. 9 (special-category data — biometric / health / face / voice); EU AI Act Art. 50 (in force 2026-08-02); COPPA (16 CFR Part 312); GDPR-K (Art. 8); UK Children's Code (ICO Age Appropriate Design Code); CCPA / CPRA (minor-data rules); EU Accessibility Act (Directive (EU) 2019/882, in force 2025-06-28).

**Out of scope for the app-store-policy surface, same as for the IP surface:** software patents (registered patent attorney); IAP / Play Billing / DMA evasion advice (the carve-outs exist — use them; grey-area schemes are a developer-account-termination risk); App Review pass/fail predictions (reviewer-specific by design); final-form regulated content (final privacy nutrition label, final Data Safety form text, final account-deletion flow copy, final age-rating questionnaire answers, final EULAs — the skill sketches structure, never produces final text).

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

## Two surfaces, two engines

The plugin has **two Opus-pinned subagents**, each pinned because the cost of misreads (App Store rejection, takedown, opposition, a cease-and-desist on launch day, a back-of-queue penalty for resubmits on the same clause) is high enough that the careful tier is worth the spend.

| Engine | Skills it powers | Setup command |
|---|---|---|
| `appdev-ip-analyst` (Opus) | `appdev-ip-triage` | `/appdev-ip-advisor:setup-ip-triage-routine` |
| `appdev-app-store-reviewer` (Opus) | `appdev-app-store-rules`, `appdev-app-store-rejection-triage` | `/appdev-ip-advisor:setup-app-store-review-routine` |

Pick the surface that matches the work:

| Surface | When to use | Privacy posture |
|---|---|---|
| **Cowork IP-triage Routine** — set up via `/appdev-ip-advisor:setup-ip-triage-routine`. Watches an inbound folder (Google Drive / Box / iManage / NetDocuments / Docusign envelope feed) and triages every new third-party asset bundle automatically — UI screenshots, icon packs, font bundles with EULAs, marketing copy drafts, T&Cs drafts, third-party licence packs. | Marketing assets, public-facing UI mocks, third-party licence bundles, generic app-name shortlists. High throughput, low touch. | Asset text and image OCR pass through Anthropic's cloud during Routine execution. Acceptable only for non-sensitive assets. |
| **Cowork app-store-review Routine** — set up via `/appdev-ip-advisor:setup-app-store-review-routine`. Watches a release-candidate folder (Drive / Box / iManage / NetDocuments) for new submission artifacts — `PrivacyInfo.xcprivacy`, `AndroidManifest.xml`, Data Safety form exports, listing copy drafts, screenshot sets, age-rating questionnaire drafts, Account Deletion flow copy, subscription paywall copy, T&Cs / privacy-policy drafts, SDK licence ledgers — and runs pre-submission policy review against each one. | Steady inflow of non-embargoed release-candidate artifacts across multiple stores. High throughput, low touch. | Artifact text and image OCR pass through Anthropic's cloud during Routine execution. Acceptable only for non-sensitive / non-embargoed release candidates. |
| **Interactive IP skill** — invoke `appdev-ip-triage` ("can we name our app this", "review this icon set's licence", "did we just clone Halide's UI", "is this AI splash image safe to ship", "is our GPL-bundled SDK safe in a closed-source app"). | Pre-launch app names under embargo, unannounced UI mocks, M&A asset-portfolio diligence, internal design docs. You decide doc-by-doc. | User controls each call; no standing cloud watch. |
| **Interactive app-store-policy skill** — invoke `appdev-app-store-rules` ("review my app for Apple §3.1.1", "is my privacy manifest complete", "do I need Account Deletion in-app for this build", "kids-category compliance check before we ship", "F-Droid Anti-Features audit", "generative AI app review for the App Store"). | Per-feature or per-release pre-submission review where confidentiality matters. | User controls each call. |
| **Interactive rejection-triage skill** — invoke `appdev-app-store-rejection-triage` ("App Review rejected my build on §5.1.1", "Play Console flagged Families policy", "Microsoft cert failure", "F-Droid build-server reproducibility broke"). | Reactive — any received rejection. Paste / drag the rejection notice in. | User controls each call. The literal rejection text is needed (paraphrase loses the section number). |

The Cowork Routines are the **primary high-volume deployment surface**. The interactive skills are the safety valve for sensitive material and for one-off reactive work (rejection-triage in particular is per-incident — there is no Routine for it).

## Start here

If you have a steady inflow of non-sensitive **third-party asset bundles** (icon packs you're evaluating, font EULAs, sound libraries, draft marketing copy) landing in a Drive folder, run:

```text
/appdev-ip-advisor:setup-ip-triage-routine
```

If you have a steady inflow of non-embargoed **release-candidate artifacts** (privacy manifests, AndroidManifest, Data Safety exports, listing copy, screenshot sets, account-deletion-flow copy) landing in a release-candidate folder, run:

```text
/appdev-ip-advisor:setup-app-store-review-routine
```

Each walks you through its own privacy gate (mandatory), folder pair, working-context confirmation (jurisdiction + asset-type-mix for IP; target-stores + app-category + features for app-store), and the Cowork **Routines** UI. Both Routines write structured issue lists to paired output folders, one file per input, before you ship. The two Routines are independent and can run side-by-side on different folders.

If you don't have that kind of inflow — or the assets are confidential / embargoed — invoke the skills interactively. For IP questions, say "is this app name safe", "review this icon-pack licence", "did we just clone the Halide camera UI", or "is this AI splash image OK to use". For app-store-policy questions, say "review my app for app-store policy across Apple and Google", "is my privacy manifest complete", "do I need in-app Account Deletion for this build", or "kids-category compliance check before we ship". For rejections, say "App Review rejected my build" (then paste the literal notice).

## Vault integration

All three skills are **vault-citizens** — they use the shared `vault-companion` surface from `vault-librarian`. IP-triage outputs land in `<vault>/Legal/`; app-store-policy reviews and rejection-triage outputs land in `<vault>/AppStore/`. Frontmatter `type` distinguishes them (`ip-triage` vs `store-policy-review` vs `store-rejection-triage`). The `Legal/` category is shared with `fintech-legal-advisor`; the `AppStore/` category is dedicated to this plugin's two app-store skills.

For `appdev-ip-triage`:

- **Phase 0.5 — Vault Recall** runs after jurisdiction and asset-type are named. It calls `vault-companion-recall` with topic = `<jurisdiction> + <asset-type> + IP`. If you've previously triaged the same (jurisdiction × asset-type) cell — e.g., another US/icon-pack triage, or another EU/trade-dress review — the prior issue list and decision history surface as context before the new triage walks the checklist. Returns silently if the vault has no matches.
- **Phase 4 — Save** delegates to `vault-companion-append` with `category=Legal`. Output writes to `<vault>/Legal/YYYY-MM-DD-<slug>.md`, with every named statute / treaty / app-store-policy section wrapped in `[[wikilink]]` form so `[[Lanham Act §43(a)]]` / `[[GDPR]]` / `[[AI Act Art. 50]]` / `[[App Store Review Guideline 5.2]]` / `[[GPL-3.0]]` / `[[AGPL-3.0]]` accumulate backlinks across your `Legal/` folder over time. Frontmatter carries `type: ip-triage, jurisdictions, asset_type, distribution_channels, issue_count, regimes`.
- **Fallback.** If you've declined a vault (or `vault-librarian` isn't installed), the triage falls back to `~/.claude/appdev-ip-triage/YYYY-MM-DD-<slug>.md` via plain file write. Same triage quality, no cross-linking.

Install `vault-librarian` alongside this plugin to get the vault-companion surface. It auto-bootstraps a personal-schema vault on first triage save — no separate command needed.

The `Legal/` category is **shared** with `fintech-legal-advisor`. The two plugins' triage logs sit side by side in the same folder; the frontmatter `type` field (`legal-triage` vs. `ip-triage`) and the wikilink surface (regulation names vs. IP-statute / app-store-policy names) distinguish them.

## Skills

| Skill | Purpose |
|---|---|
| `appdev-ip-triage` | IP issue-spotter walked through Phase 0 (jurisdiction + asset type + distribution channels) → Phase 0.5 (vault recall, optional) → Phase 1 (match the (jurisdiction × asset-type × channel) cell) → Phase 2 (walk the issue checklist) → Phase 3 (output structured issue list) → Phase 4 (save via vault-companion-append, `<vault>/Legal/`) → Phase 5 (take-to-counsel block). Covers US (federal + key state right-of-publicity), EU (EUTM + national copyright + AI Act), UK (post-Brexit UKTM + CDPA), Russia (Civil Code Part IV + Rospatent), Singapore (TMA 2005 + Copyright Act 2021), UAE (Federal Decree-Law 36/2021 + DIFC + ADGM). Regimes: trademark, trade dress, copyright (icons/fonts/music/code/photo/AI), open-source licence compliance, right of publicity, app-store IP rules. |
| `appdev-app-store-rules` | **App-store pre-submission policy reviewer.** Phase 0 (stores + app category + features + distribution channels) → Phase 0.5 (vault recall) → Phase 1 (per-store checklist cluster — Apple / Google / Microsoft / Amazon / Samsung / Huawei / F-Droid / Flathub / Snap / Steam / Epic / EU-DMA alt iOS) → Phase 2 (walk the per-store + cross-store checklist) → Phase 3 (structured issue list with the specific artifact + key to change, and cross-store carry-over flags) → Phase 4 (save to `<vault>/AppStore/`) → Phase 5 (take-to-store-policy-review block). Routes IP-rooted findings back to `appdev-ip-triage`, privacy/AI-Act findings to privacy counsel, restricted-category findings (gambling / crypto / health / finance / kids) to subject-matter counsel. |
| `appdev-app-store-rejection-triage` | **Reactive sibling.** Fires when a build has already been rejected. Parses the rejection notice, identifies the cited guideline section(s), categorises the rejection (technical / content / IP / privacy / monetisation / metadata / account-data / restricted-category / AI-generative / sherlocked / build-signing / geographic-export), and produces a structured remediation plan: what to change in which artifact, fix-vs-appeal decision, official appeal route per store, cross-store carry-over flags. Saves to `<vault>/AppStore/`. Routes IP-rooted rejections through `appdev-ip-triage`. |

## Subagents

| Agent | Model | Role |
|---|---|---|
| `appdev-ip-analyst` | **Opus** | The careful IP tier. Pinned to Opus because the cost of IP misreads (App Store rejection, takedown, opposition, a cease-and-desist on the launch-day press release) is high enough that the careful tier is worth the spend. Asks jurisdiction + asset type + distribution channel first, refuses to proceed without them. Outputs an issue list, never a verdict. Refuses to opine on software patents — routes to a patent attorney. Ends every output with a take-to-counsel block. Powers `appdev-ip-triage`. |
| `appdev-app-store-reviewer` | **Opus** | The careful app-store-policy tier. Pinned to Opus because the cost of a wrong call (App Review rejection cascade, takedown of a live listing, certification fail on a Microsoft Store submission, F-Droid build-server reproducibility break) is calendar-days of lost revenue plus a back-of-queue penalty for resubmits on the same clause. Asks stores + app-category + features + distribution-channel mix first, refuses to proceed without all four. Outputs issue lists and remediation plans, never pass/fail predictions. Refuses IAP / DMA / Play Billing evasion advice. Routes IP-rooted findings back to `appdev-ip-analyst` via `appdev-ip-triage`. Ends every output with a take-to-store-policy-review block. Powers `appdev-app-store-rules` and `appdev-app-store-rejection-triage`. |

## Slash commands

| Command | What it does |
|---|---|
| `/appdev-ip-advisor:setup-ip-triage-routine` | Wire `appdev-ip-triage` into a Cowork Routine that watches an inbound asset / spec / licence-bundle source (Google Drive / Box / iManage / NetDocuments / Docusign envelope feed). Includes a mandatory privacy gate (Step 0) that refuses to proceed without explicit non-sensitive-only confirmation. Optionally emits Word tracked-changes output if the Microsoft connector is granted (for draft marketing copy / T&Cs / acknowledgements screens). |
| `/appdev-ip-advisor:setup-app-store-review-routine` | Wire `appdev-app-store-rules` into a Cowork Routine that watches a release-candidate folder (Drive / Box / iManage / NetDocuments) for new submission artifacts — `PrivacyInfo.xcprivacy`, `AndroidManifest.xml`, Data Safety form exports, listing copy drafts, screenshot sets, age-rating questionnaire drafts, Account Deletion flow copy, subscription paywall copy, T&Cs / privacy-policy drafts, SDK licence ledgers — and runs pre-submission policy review against each one. Same mandatory privacy gate (non-sensitive-only release candidates), same optional Word tracked-changes pass when the Microsoft connector is granted. Pairs with — and is independent from — the IP-triage Routine; the two watch different folders. |

## Where things live

| Artifact | Path (vault) | Path (no vault) |
|---|---|---|
| IP-triage outputs (interactive) | `<vault>/Legal/YYYY-MM-DD-<slug>.md` | `~/.claude/appdev-ip-triage/YYYY-MM-DD-<slug>.md` |
| IP-triage outputs (Routine) | Output folder you named at setup (in whichever document system you used as the source — Drive / Box / iManage / NetDocuments) | — |
| App-store-policy review outputs (interactive) | `<vault>/AppStore/YYYY-MM-DD-<slug>.md` | `~/.claude/appdev-app-store-rules/YYYY-MM-DD-<slug>.md` |
| App-store rejection-triage outputs | `<vault>/AppStore/YYYY-MM-DD-<slug>.md` | `~/.claude/appdev-app-store-rejection-triage/YYYY-MM-DD-<slug>.md` |
| App-store-review outputs (Routine) | Output folder you named at setup (Drive / Box / iManage / NetDocuments) | — |
| Tracked-change Word output — IP triage (optional) | Next to source draft, suffixed `-redline-<YYYY-MM-DD>.docx` | — |
| Tracked-change Word output — store policy (optional) | Next to source draft, suffixed `-store-policy-redline-<YYYY-MM-DD>.docx` | — |
| Plugin state | — | `~/.claude/appdev-ip-advisor.local.md` |

## Cowork Routine templates

A copy-paste template for the IP-triage Routine lives at `docs/routines/ip-triage-on-drive-routine.md`. The slash commands (`/appdev-ip-advisor:setup-ip-triage-routine` and `/appdev-ip-advisor:setup-app-store-review-routine`) each build the full Routine prompt inline; if you want to wire either Routine by hand instead of through the command, run the command once, copy the Step-4 prompt block, and configure the Routine yourself. **Read the privacy header before installing either Routine.** Both refuse to wire without explicit non-sensitive-only confirmation.

## Hard limits

- **No "is this safe to ship" yes/no answers and no App Review pass/fail predictions.** The plugin returns issue lists and remediation plans, not verdicts. App Review is reviewer-specific by design.
- **No software-patent analysis.** Patent freedom-to-operate searches, infringement opinions, and patent-clearance work are out of scope. The plugin will refuse and route to a registered patent attorney.
- **No trademark freedom-to-use opinions.** The plugin can flag that a name looks too close to an existing mark; only an IP attorney issues a freedom-to-use opinion.
- **No IAP / Play Billing / DMA evasion advice.** Apple §3.1.1 and Google Play §4.1 are the rules. The genuine carve-outs — Reader-app, Music-streaming, External Link Account, DMA Alternative Marketplace, User Choice Billing — are the only valid framings. Grey-area schemes (hidden payment surface, dark-UX web checkout, region-shifted billing, separate developer accounts for the same product family) get refused and explained as a developer-account-termination risk.
- **No "ship it and see" advice.** A rejection that was technically correct stays technically correct after a resubmit-without-fix; App Review tracks resubmission patterns and back-of-queues frequent re-rejecters on the same clause.
- **No drafting of final-form regulated content** (final App Store metadata, final privacy nutrition label, final Google Play Data Safety form text, final Account Deletion flow copy, final age-rating questionnaire answers, final T&Cs, final EULAs, final Acknowledgements screens, final OSS-attribution text, final AI-content disclosure language). The plugin can sketch a structure or surface a missing clause; final text needs counsel or store-policy lead sign-off.
- **No case-law interpretation.** Citations to specific cases (Sony v. Universal, Authors Guild v. Google, Star Athletica v. Varsity Brands, Andy Warhol Foundation v. Goldsmith, Epic v. Apple, the pending generative-AI cases) are read for *naming* — the analyst surfaces the case name as a flag for counsel, never as a holding.
- **No DMCA-strategy advice** (whether to issue or contest a takedown). That's outside-counsel work.
- **No evasion advice.** "Set up in Country X to avoid GPL contagion / right-of-publicity rules / AI Act disclosure" — refuse and explain. Same for "ship from a separate developer account to dodge App Review history".
- **Take-to-counsel block on every IP output. Take-to-store-policy-review block on every app-store output.** No exceptions. No softening.
- **IP surface — jurisdiction + asset type + distribution channel first, every time.** App-store surface — **stores + app category + features + distribution-channel mix first, every time.** No analysis without all four on the app-store side; the parameters bound the relevant ruleset.

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
- **App-store policy (IP slice)** — Apple App Store Review Guidelines, esp. § 5.2 (Intellectual Property); Google Play Developer Policy Center — Impersonation, Intellectual Property, Deceptive Behavior.
- **App-store policy (broader surface — per-store developer portals)** — Apple: developer.apple.com/app-store/review/guidelines/ + Apple's privacy-manifest / Required Reason API / App Tracking Transparency / Sign in with Apple reference docs + visionOS / watchOS / CarPlay / tvOS HIGs + Apple's DMA developer documentation (Alternative app marketplaces in the EU, Notarization for iOS apps, Core Technology Fee). Google: play.google.com/about/developer-content-policy/ + Play Console help + developer.android.com/topic/performance/vitals + Data Safety form documentation + Families Policy + User Choice Billing programme docs. Microsoft: learn.microsoft.com/en-us/windows/uwp/publish/store-policies + Partner Center docs. Amazon: developer.amazon.com (Appstore Policy Center). Samsung: seller.samsungapps.com. Huawei: developer.huawei.com (AppGallery Connect). F-Droid: f-droid.org/docs/Inclusion_Policy/ + f-droid.org/docs/Anti-Features/ + f-droid.org/docs/Reproducible_Builds/. Flathub: docs.flathub.org/docs/for-app-authors/submission/. Snap: snapcraft.io/docs (confinement + snapd interface reference). Steam: partner.steamgames.com/doc/home + Steam Direct documentation. Epic: dev.epicgames.com/docs/epic-games-store/.
- **App-store policy (legal overlays the reviewer flags but routes)** — GDPR Art. 17 (erasure, behind Account Deletion); GDPR Art. 9 (special-category biometric / health data); EU AI Act Art. 50 (AI-content disclosure, in force 2026-08-02); COPPA (16 CFR Part 312); GDPR-K (GDPR Art. 8); UK Children's Code (ICO Age Appropriate Design Code); CCPA / CPRA minor-data rules; EU Accessibility Act (Directive (EU) 2019/882, in force 2025-06-28).
- **OSS licence canon** — GPL-2.0 / GPL-3.0 / LGPL-2.1 / LGPL-3.0 / AGPL-3.0 (FSF texts); MIT / BSD-2-Clause / BSD-3-Clause / Apache-2.0 / MPL-2.0 / EPL-2.0 (OSI texts); "source-available" non-OSS: Business Source License 1.1, SSPL-1.0, Elastic License 2.0.
- **AI-content policy** — US Copyright Office *Guidance on Works Containing AI-Generated Material* (37 CFR Part 202, 2023, and the 2026 update); EU AI Act Art. 50 (in force 2026-08-02); OpenAI / Anthropic / Stability / Midjourney commercial-use terms (versioned, verify live).
