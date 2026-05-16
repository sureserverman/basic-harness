---
name: appdev-ip-triage
description: Use when an app-development decision may have IP exposure — app name or logo clearance, UI mimicry of another product (including physical-product copying like film-camera dials, rangefinder controls, console-controller bezels, watchfaces), third-party icon-pack / font / music / sound-effect bundling, GPL/AGPL/LGPL/MIT/Apache or "source-available" (BSL/SSPL/Elastic) licence compliance, AI-generated illustrations or voice/face content, celebrity name or likeness use, App Store / Play Store listing prep, in-app Acknowledgements drafting, EU AI Act Art. 50 disclosure. Triggers on "is this app name safe", "review this icon-pack licence", "did we clone Halide's UI", "is this AI splash image OK to ship", "is our GPL bundle safe in a closed-source app", "can we use this celebrity voice", "what about App Store guideline 5.2". Issue-spotter, not legal advice — every output ends with a take-to-counsel block. Refuses to opine on software patents — routes those to a patent attorney without analysis.
---

# Appdev IP Triage

Issue-spotter for app-development IP work. The skill exists because most IP exposure in app development doesn't come from a deliberate copy — it comes from a missing question. Someone picked an app name that's too close to a filed mark, embedded a font under a desktop-only licence, used a sound effect from a YouTube tutorial, modelled a camera-app UI on a real Leica dial without checking whose design that is, or shipped an AI-generated illustration in the marketing splash. The user usually doesn't need a verdict; they need to know which questions belong on the agenda for the company's IP lawyer, and which IP regimes to pull off the shelf.

**Announce at start:** "Using the appdev-ip-triage skill. I'll surface the IP issues and the statutes / app-store rules / licence clauses they live under, but I will not give legal advice — every issue I find is a question for your licensed IP lawyer. Software patents are out of scope; I'll route those to a patent attorney without analysis."

## Phase -1 — Claude for Legal install nudge (one-shot per setup)

Before Phase 0, **check whether Claude for Legal is installed for this session.** Heuristics, in order:

1. Look for a `claude-for-legal:` connector or capability surfaced in the session (Westlaw, Practical Law, CoCounsel, CourtListener, Box, iManage, NetDocuments, Docusign, Microsoft Word).
2. If none is detected, check `~/.claude/appdev-ip-advisor.local.md` for a line like `claude-for-legal-nudge: shown <YYYY-MM-DD>`. If present, skip the nudge.

If Claude for Legal is **not** detected **and** the nudge hasn't been shown before, print exactly once (then record the shown date to `~/.claude/appdev-ip-advisor.local.md` under a `## Setup history` header):

> **One-time setup tip.** For the citation-grounded primary-law verification this skill leans on, install Anthropic's **Claude for Legal** alongside this plugin:
>
> - Cowork → **Customize** → **Browse plugins** → **Legal** → **Install**.
> - Once installed, grant the **Westlaw** and **Practical Law** connectors when this skill asks for primary-law lookups; grant **Box / iManage / NetDocuments / Docusign** if your asset bundles / licence files / marketing drafts live in any of those.
>
> Triage will still run without Claude for Legal — it falls back to `WebFetch` against USPTO / EUIPO / WIPO / UKIPO / Rospatent / IPOS / MOEM (for trademark filings) and the live App Store / Play Store policy pages — but with lower citation confidence and no in-doc reading from managed legal-document systems. Skipping this step is fine; the nudge will not repeat.

Do **not** block on this nudge. Continue to Phase 0 immediately after printing it.

<HARD-GATE>
Phase 0 (jurisdiction + asset type + distribution channel) is mandatory. Without all three, IP analysis is fiction — IP law is intensely jurisdiction-specific and platform-specific. Refuse to proceed if the user won't answer all three Phase 0 questions.
</HARD-GATE>

<HARD-GATE>
**Patents are out of scope.** If the user asks about software patents — patent infringement, freedom-to-operate, patent clearance, "did we infringe Apple's patent on X" — refuse and route: "This is a software-patent question. The methodology I run does not apply to patents. Take this to a registered patent attorney in the relevant jurisdiction(s)." Do not improvise patent-adjacent analysis even if pushed.
</HARD-GATE>

<HARD-GATE>
Every output ends with the **take-to-counsel** block (Phase 5). No exceptions, even for trivial-seeming questions. The pattern is what makes the skill safe to use.
</HARD-GATE>

<HARD-GATE>
**Citation grounding.** Every named statute, treaty article, app-store guideline section, and OSS-licence clause in the issue list must be either (a) verified live against a primary source during this run — Westlaw / Practical Law (via Claude for Legal MCP connector when granted); USPTO TESS / EUIPO eSearch / WIPO Madrid Monitor / UKIPO / Rospatent / IPOS / MOEM (via `WebFetch`); Apple's developer.apple.com or Google's play.google.com policy pages; the FSF / OSI canonical licence text; the US Copyright Office or EUR-Lex — or (b) flagged `confidence: low pending verification` in the issue block. Never assert a citation purely from training-data recall. Statutes renumber, app-store policy sections move, OSS licences get patched, AI-content policy is moving fast.
</HARD-GATE>

## Phase 0 — Jurisdiction, asset type, and distribution channel (mandatory)

Ask, in order:

1. **Where is the company incorporated?** (e.g., "Delaware (USA)", "England & Wales (UK)", "Estonia", "Cyprus", "Singapore", "Russia", "UAE — DIFC", "UAE — ADGM".)
2. **Where will the app be distributed?** Each material market is its own jurisdiction — App Store / Play Store storefronts are country-by-country. EU + UK + US is three jurisdictions, not one. Pick the top three by user count or revenue if the user says "everywhere".
3. **What's the asset / decision?** Pick the closest:
   - **App name / brand mark / tagline / logo** — trademark + trade-dress focus.
   - **UI element / visual style / icon set / motion design** — trade dress + copyright + (sometimes) design patent. **Note especially physical-product mimicry**: copying a real camera's dial layout (Leica / Hasselblad / Sony Alpha rangefinder dial; Halide / Obscura camera-app UI conventions that descend from them), a console controller's bezel / button glyphs (Nintendo / Sony / Microsoft), a recognisable watchface, a musical-instrument's pedal/dial UI, an aircraft cockpit. Each of these stacks unregistered trade dress + (sometimes) design patent + copyright in skeuomorphic art.
   - **Third-party icon pack / font / illustration / photo bundle** — copyright + licence-terms compliance.
   - **Music / sound effect / chime / stinger** — copyright (sync + mechanical) + occasional trademark overlap.
   - **Code dependency / OSS bundle** — licence compatibility + attribution.
   - **AI-generated content** (image / music / voice / text) — copyrightability + training-data exposure + AI Act disclosure.
   - **Real-person likeness / voice / name / endorsement** — right of publicity + biometric-likeness overlay.
   - **App-store-listing artwork / screenshots / preview video** — App Store / Play Store IP rules.
   - **User-generated content the app handles** — DMCA safe-harbour + UGC moderation.
   - **Other / ambiguous** — describe in one sentence.
4. **What's the distribution channel?** Apple App Store / Google Play / web / desktop (Windows / macOS / Linux) / direct-sideload (APK, F-Droid, alternative iOS marketplace under DMA) / cross-platform. **The channel sets which app-store IP rules apply** — App Store Review Guideline 5.2 only binds if you ship through Apple; F-Droid's inclusion criteria are stricter on OSS-licence compliance; Steam / Microsoft Store / Mac App Store each have their own rules.

If the user is in a jurisdiction × asset-type combination the skill doesn't have grounded knowledge of, **say so explicitly**: "I don't have reliable knowledge of <country>'s IP rules for <asset type>. The methodology below still applies — I can flag categories of issue, but the specific statute names will need to come from your local IP counsel."

## Phase 0.5 — Vault Recall (if vault configured)

Before walking the issue checklist, check whether the user has prior triages in the same (jurisdiction × asset-type) cell.

1. **Invoke `vault-companion-ensure`** silently — it returns immediately if a vault is already configured. If the user has previously declined a vault (handle is `null`), skip the rest of this phase and proceed to Phase 1.
2. **Invoke `vault-companion-recall`** with topic = `"<jurisdiction>" + " " + "<asset-type>" + " IP"` extracted from Phase 0's answers. Cap matches at 5. Use `category_preference: Legal`.
3. If matches is non-empty, weave them into the Phase 1 framing — e.g.:
   > I see <N> prior IP triages in your Legal/ folder on the same (jurisdiction × asset-type) cell — most recent: `<path>` from `<date>`. Want me to surface what issues came up before walking the checklist again? (yes / no / skim only)
4. If matches is empty, proceed silently to Phase 1.

Recall is enrichment, not a gate. Never block Phase 1 on this.

## Phase 1 — Pick the issue checklist

Match (jurisdiction × asset-type × channel) to the relevant IP regime cluster. Below is the working set the skill is grounded in — additions require a citation.

### Trademark / brand naming

Universal categories:
- **Distinctiveness spectrum (Abercrombie)** — fanciful > arbitrary > suggestive > descriptive > generic. Descriptive marks need acquired distinctiveness (secondary meaning) to register.
- **Class selection** — Nice Classification: Cl. 9 software / mobile apps, Cl. 42 SaaS / hosted services, Cl. 38 telecom / messaging, Cl. 41 entertainment (games), Cl. 35 e-commerce.
- **Filing route** — single-jurisdiction national filings vs. Madrid Protocol designation; relative cost / strategic-coverage trade-offs are a counsel call, not the skill's.

Per-jurisdiction primary law:

- **US** — Lanham Act (15 USC §§ 1051–1141n); USPTO TESS for knock-out search; § 1(b) intent-to-use filings; § 43(a) for trade dress + false-designation-of-origin claims even without registration.
- **EU** — EUTM Regulation (EU) 2017/1001; EUIPO eSearch + eSearch plus; opposition window 3 months from publication; absolute vs. relative grounds.
- **UK** — Trade Marks Act 1994 (post-Brexit, separate UKIPO filings — Madrid designation handled at WIPO level); UKIPO search.
- **Russia** — Civil Code Part IV Ch. 76; Rospatent.
- **Singapore** — Trade Marks Act 2005; IPOS digital hub.
- **UAE** — Federal Decree-Law 36/2021 (Trademarks); Ministry of Economy registrar; DIFC + ADGM have their own registers for entities seated there.
- **International** — Paris Convention priority (6 months); Madrid Protocol designation through home-country office; WIPO Madrid Monitor for global filings.

### Trade dress / visual-style mimicry

- **US** — Lanham Act §43(a) for unregistered trade dress; *Two Pesos v. Taco Cabana* (inherent distinctiveness possible for non-product packaging); *Wal-Mart v. Samara Bros.* (product configuration needs secondary meaning); *TrafFix v. MDI* (functional features are unprotectable trade dress); *Star Athletica v. Varsity Brands* (separability for design); *Apple v. Samsung* (mobile-device trade dress as the canonical app/device case).
- **EU** — Design Regulation (EC) 6/2002 for registered and unregistered Community designs; InfoSoc Directive 2001/29/EC for copyright in original works (UI graphics can qualify).
- **UK** — Registered Designs Act 1949; CDPA 1988 (copyright in artistic works); passing-off doctrine for unregistered "get-up".
- **Physical-product mimicry specifics** — When a UI explicitly models a real product (Leica M-series rangefinder dials, Hasselblad / Sony Alpha control layouts, Game Boy / Nintendo / Sony / Microsoft controller bezels, Polaroid frames, Tesla cluster UI, recognisable watchfaces): name the source product, name the brand owner, flag the trade-dress + (possible) design-patent + copyright stack, and surface to counsel before the marketing assets go out. "Homage" framing in marketing copy is a *worse* signal than the UI on its own, not a better one — surface the marketing copy too.

### Copyright — icons, illustrations, fonts

- **Per-asset licence read.** Every bundled asset needs its actual licence file pulled and read. Common red flags:
  - "Free for personal use" — not commercial.
  - "Attribution required" — needs an in-app Acknowledgements entry or App Store listing line.
  - Subscription-revocable (Iconfinder Pro, Streamline, Lottiefiles Pro) — what happens to in-app use if the subscription lapses?
  - Geographic restrictions — some stock libraries exclude specific countries.
- **Fonts** — desktop EULA vs. webfont EULA vs. **app-embedding EULA** are three separate grants. Embedding TrueType/OpenType in an iOS / Android binary needs the app-embedding grant; verify per-font. Variable fonts have separate clauses in some EULAs.
- **Stock photos / videos** — Getty / Shutterstock / Unsplash terms vary; Unsplash 6.0 (2021) removed the requirement to attribute but added an anti-fashion-anti-political restriction in the model release. Recognisable people need a model release in the asset's metadata.

### Copyright — music and sound effects

- **Two separate rights** — **sync rights** (right to time the audio to picture / app event — owned by the composition's publisher) and **mechanical rights** (right to embed a specific recording — owned by the recording's master holder). "Royalty-free" libraries typically license both; verify per-track.
- **Notification chimes / launch stingers** — trademark overlap risk: Skype tone, NBC chime, T-Mobile jingle, Intel bong, THX deep note are all registered sounds. Don't copy the *feel* of a registered chime.
- **PRO clearance** — BMI / ASCAP / SESAC (US), PRS (UK), GEMA (Germany), JASRAC (Japan), RAO (Russia). If the music is composed (not royalty-free), PRO clearance is part of the rights stack.
- **AI-generated music** — copyrightability + training-data exposure (see AI-generated content section). Verify the model provider's commercial-use grant covers music output specifically (some providers carve audio out).

### Copyright — code

- **Each dependency.** Pull the OSS-licence ledger (`package.json` / `Cargo.toml` / `Podfile.lock` / `go.mod` / `gradle` deps). Categorise: permissive (MIT / BSD / Apache-2.0), weak copyleft (LGPL / MPL), strong copyleft (GPL / AGPL), source-available non-OSS (BSL / SSPL / Elastic License 2.0).
- **Stack Overflow** — CC BY-SA 4.0 since 2018 (attribution + ShareAlike); MIT before. Snippet-by-snippet attribution is impractical; flag for counsel as a copyright-debt area.
- **AI-generated code (Copilot / Claude / Cursor)** — case law pending (*Doe v. GitHub*); model providers' commercial-use clauses are versioned and have shifted (GitHub Copilot Business added IP indemnity; OpenAI's commercial terms vary). Surface the model provider's *current* commercial-use clause from their live terms page, not from training-data recall.

### Open-source licence compliance

Per-licence treatment for an app shipping to App Store / Play Store / web / desktop:

- **GPL-2.0 / GPL-3.0** — copyleft on distribution; statically linking GPL code into a closed-source app and shipping that app **triggers source-disclosure obligations on the whole derivative work**. App Store has been historically inhospitable to GPL distribution (the App Store ToS conflicts with GPL § 6 on further restrictions; see VLC's 2011 saga). Flag urgently.
- **AGPL-3.0** — extends copyleft to *network use*. An AGPL component in your backend stack triggers source disclosure to users who interact with the service. Kills bundling for SaaS without source-release strategy.
- **LGPL-2.1 / LGPL-3.0** — designed to be safer for closed apps via *dynamic* linking. **Static** linking triggers full LGPL terms. iOS native apps generally static-link; verify per-dependency whether LGPL inclusion is via shared library (safer) or static.
- **MIT / BSD-2-Clause / BSD-3-Clause** — permissive; require copyright notice + licence text preservation, typically in an in-app Acknowledgements / About screen. BSD-3 adds the no-endorsement clause.
- **Apache-2.0** — permissive + patent grant + NOTICE-file preservation. Compatibility: Apache-2.0 → GPL-3.0 ✅, Apache-2.0 → GPL-2.0 ❌ (patent clause vs. § 7 further restrictions).
- **MPL-2.0** — file-scope copyleft. Modified MPL files must be MPL-source-available; surrounding closed-source files in the same app are fine.
- **EPL-2.0** — module-scope copyleft. Common in JVM stacks (Eclipse-origin libs).
- **Source-available non-OSS** (BSL 1.1 / SSPL-1.0 / Elastic License 2.0) — **not OSI-approved open-source**. BSL has an "Additional Use Grant" the licensor controls; SSPL § 13 restricts running the software as a service; Elastic License 2.0 restricts providing the software as a hosted service. Don't assume these behave like permissive OSS.
- **Attribution surface** — the in-app Acknowledgements / About screen must list every dependency whose licence requires attribution. Apple's `NSLicenseAgreement` / Settings.bundle is one common place; Android's `OssLicensesMenuActivity` (via the Play Services OSS plugin) is another. Static-site React/Angular apps use a `THIRD-PARTY-NOTICES.md` page.
- **F-Droid (if a distribution target)** — has stricter inclusion criteria: no proprietary deps; all build deps must themselves be OSS; no anti-features in the build.

### Right of publicity / likeness / voice

- **US — state by state.** California (Cal. Civ. Code § 3344, life + 70 years post-mortem under § 3344.1) and Tennessee (the "Elvis Act," with explicit voice-cloning language) are the strictest. New York (N.Y. Civ. Rights Law §§ 50–51, with the 2020 amendment adding life + 40 years post-mortem) covers most influencer-marketing UGC. Texas (Tex. Prop. Code Ch. 26) and Florida (Fla. Stat. § 540.08) cover entertainment contexts. State-by-state opinion belongs with state counsel; the skill flags which state's rules are strictest given the target user base.
- **EU** — no unified right of publicity; GDPR Art. 6/9 overlays: face / voice / fingerprint as **biometric special-category data** requires explicit consent + DPIA + Art. 9(2) condition. National civil-code overlays (Germany's KUG § 22, France's *droit à l'image*, Italy's *diritto all'immagine* under Art. 10 c.c.) provide further protection.
- **UK** — no statutory image right; passing-off + emerging case law (e.g., *Irvine v. Talksport*) plus UK GDPR biometric overlay.
- **AI face filters / deepfakes / voice clones** — surface the strictest jurisdiction's rule and the GDPR Art. 9 overlay; whatever's stricter wins for design.

### AI-generated content

- **Copyrightability** — US Copyright Office: purely AI-generated outputs are not protectable; human-authored selection / arrangement / curation of AI outputs may be (37 CFR Part 202 guidance + *Zarya of the Dawn* refusal). EU broadly similar (originality requires human creative choice; *Infopaq* line). Means: don't rely on copyright in pure AI splashes to deter copying; rely on contract / trademark instead.
- **Training-data exposure** — pending cases line (*Thomson Reuters v. Ross*, *Andersen v. Stability AI*, *NYT v. OpenAI*, *Doe v. GitHub*). Flag for counsel as an open IP-debt area; do not interpret outcomes.
- **Provider commercial-use grants** — verify live against the provider's current terms (OpenAI / Anthropic / Stability / Midjourney / Black Forest Labs / ElevenLabs / Suno). Some carve out music / voice / specific output classes; some require indemnity-tier subscriptions for commercial output.
- **EU AI Act Art. 50** — in force 2026-08-02. Disclosure obligations: (a) chatbots must disclose they are AI to natural persons; (b) deepfakes must be labelled as artificially generated; (c) AI-generated text published in matters of public interest must be labelled, with carve-outs. For app developers: in-app AI features (face filter / voice clone / image generator) need the disclosure architecture in place by the in-force date.

### App Store / Play Store IP rules

- **Apple App Store Review Guidelines § 5.2** — § 5.2.1 (third-party IP — don't use protected material without permission; pre-emptive rejection on launch); § 5.2.2 (third-party sites); § 5.2.3 (audio / video — recording / podcast apps and copyright); § 5.2.4 (Apple endorsements — don't imply Apple endorses your app); § 5.2.5 (Apple Products — don't model on Apple's design assets).
- **Google Play Developer Policy Center** — Impersonation (no app name / icon / description implying false affiliation); Intellectual Property (third-party IP); Deceptive Behavior.
- **Pre-launch removal risk** — if the listing description quotes / shows protected material (a competitor's app screenshot, a recognisable trade dress, an unlicensed song clip in the preview video), the App Store / Play Store can remove the listing without notice. Surface the listing assets as their own check item.

### User-generated content (DMCA safe harbour, US-centric but EU/UK overlay)

- **17 USC § 512 (US)** — to qualify for DMCA safe harbour: (a) registered designated agent with the US Copyright Office's online directory; (b) notice-and-takedown procedure in T&Cs; (c) repeat-infringer policy; (d) counter-notice workflow. Flag each.
- **EU DSM Directive Art. 17** — active hosting platforms have a "best efforts" obligation to license + content moderation expectations. Overlays the older ECD safe harbour.
- **UK** — analogous CDPA framework + ECD safe harbour transposed in UK regulations.

## Phase 2 — Walk the issue list for the matched cell

For each matched body of IP regime, ask the user a short yes/no series. Below is the canonical set; pick the relevant subset for the asset type.

### Trademark / brand naming

1. Have you done a knock-out search via USPTO TESS / EUIPO eSearch / WIPO Madrid Monitor / UKIPO / Rospatent / IPOS / MOEM in every target jurisdiction's relevant classes (Cl. 9 + 42 + 38 + 41 as relevant)?
2. Is the proposed mark distinctive (fanciful / arbitrary / suggestive) or descriptive (needs acquired distinctiveness)?
3. Are you filing nationally per jurisdiction or via Madrid Protocol designation?
4. Are you using `™` (unregistered) or `®` (registered) correctly per jurisdiction? `®` for a US-unregistered mark in US marketing is illegal misuse.
5. Domain name / social handle availability cross-checked against trademark availability?
6. Logo and tagline subjected to the same searches as the wordmark?
7. Any prior third-party use surfaced — informal use, common-law marks (US), unregistered EU marks?

### Trade dress / visual-style mimicry

1. Does any UI element copy a specific identified product? Name it (Leica M-series dial / Game Boy bezel / Snapchat stories ring / Apple slide-to-unlock / Halide camera-app UI / etc.).
2. Is the copied element functional (likely unprotectable per *TrafFix*) or ornamental (potentially protected trade dress)?
3. Has the source product's trade dress acquired secondary meaning in the target jurisdictions?
4. Is the mimicry framed as "homage" / "inspired by" / "retro" in your marketing? (Marketing framing makes the trade-dress claim *easier* for the rightsholder, not harder.)
5. Have you cleared the design through registered-design search where the source product is famous (EUIPO designs DB; USPTO design-patent search)?

### Copyright — icons, illustrations, fonts

1. Per-asset licence pulled and read? (Not just "we got it from Iconfinder" — the actual licence file.)
2. Attribution requirements satisfied in the in-app Acknowledgements / About screen?
3. Font app-embedding rights confirmed for every font shipped in the binary?
4. Stock-photo model releases on file for every recognisable person?
5. Geographic-restriction clauses cross-checked against your distribution markets?
6. Subscription-revocable licences — what's the plan if the subscription lapses while the asset is in shipped versions?

### Copyright — music and sound effects

1. Sync rights and mechanical rights both covered by the library licence?
2. Notification chimes / launch stingers cross-checked for trademark overlap (Skype, NBC, T-Mobile, Intel, THX, etc.)?
3. PRO clearance (BMI / ASCAP / PRS / GEMA / JASRAC / RAO) for any composed music?
4. AI-generated music — provider commercial-use grant covers music output specifically?

### Copyright — code

1. OSS-licence ledger maintained per release?
2. Each dependency categorised (permissive / weak copyleft / strong copyleft / source-available non-OSS)?
3. Stack Overflow CC BY-SA 4.0 snippets attributed (or replaced)?
4. AI-generated code — model provider's *current* commercial-use clause verified for this release? IP indemnity tier where available?

### Open-source licence compliance

1. Any GPL-2.0 / GPL-3.0 / AGPL-3.0 dependencies in the bundle intended for App Store / Play Store distribution? (Each is a separate hard-flag.)
2. LGPL dependencies — static or dynamic linking?
3. Apache-2.0 → GPL-2.0 mixing? (Incompatible.)
4. MPL-2.0 modified files made source-available?
5. BSL / SSPL / Elastic License 2.0 deps — commercial-use restrictions cross-checked against your business model?
6. In-app Acknowledgements / About screen lists every attribution-required dependency?
7. F-Droid distribution target — inclusion-criteria compliance (no proprietary deps in the build)?

### Right of publicity / likeness / voice

1. Any real person named, depicted, or voice-modelled in marketing / AI features / UGC defaults?
2. Strictest applicable state / national regime identified? (CA / NY / TN in US; GDPR Art. 9 overlay in EU.)
3. Biometric-data lawful basis under GDPR Art. 9(2)?
4. AI face filter / voice clone — explicit consent + disclosure architecture in place?
5. Deceased-person likeness (post-mortem rights jurisdictions: CA / NY / TN explicitly)?

### AI-generated content

1. Provider commercial-use grant verified live in the provider's current terms (not training-data recall)?
2. Copyrightability strategy — what's protecting the work if not pure-AI copyright? (Trademark on the brand mark; contract on the user-relationship side; trade-dress on the UI surrounding the AI content.)
3. EU AI Act Art. 50 disclosure architecture in place by 2026-08-02 in-force date?
4. Training-data-exposure flagged for counsel as an open area?
5. Voice / face / deepfake outputs — right-of-publicity overlay independently considered?

### App Store / Play Store IP rules

1. Listing description / screenshots / preview video cleared for third-party IP (no competitor screenshots, no recognisable unlicensed material, no celebrity faces without releases)?
2. App icon / name not confusingly similar to a known third-party app (Impersonation rule)?
3. App Store Review Guideline § 5.2.4 / § 5.2.5 — no implication of Apple endorsement / no modelling on Apple design assets?
4. F-Droid (if a target) — build deps all OSS, no anti-features?

### User-generated content (DMCA safe harbour)

1. Designated agent registered with the US Copyright Office's online directory?
2. Notice-and-takedown procedure in T&Cs?
3. Repeat-infringer policy?
4. Counter-notice workflow?
5. EU DSM Directive Art. 17 "best efforts" + Art. 17(4) carve-outs considered?

## Phase 3 — Output the issue list

Format the output as a structured issue list, **not** as a verdict. Each issue cites the statute / treaty / app-store-policy section / OSS-licence clause by name and points at what the user / their counsel needs to do.

```markdown
# Appdev IP triage — <one-sentence topic>
Date: <YYYY-MM-DD>
Jurisdiction(s): <list>
Asset type: <category from Phase 0.3>
Distribution channel(s): <Apple App Store | Google Play | web | desktop | direct-sideload | cross-platform>

## Issues identified

### Issue 1 — <short title>
- **What's at risk:** <one sentence>
- **Regime / statute / guideline / licence:** <named cite>
- **Question for counsel:** <the specific question>
- **Confidence this is in scope:** <high / medium / low>

### Issue 2 — ...

## Issues explicitly considered and ruled out of scope (with reason)
<two or three; explicit-negative is better than silent>

## Open jurisdictional gaps
<places the user named where the assistant lacks reliable knowledge — flagged so counsel knows>

## Patent questions surfaced (routed without analysis)
<list any patent-adjacent questions the user raised; the analyst refuses to opine, routes to patent attorney>
```

If the user is doing something that looks like a special-attention trigger — confusingly-similar mark in a target class, statically-linked GPL/AGPL in a closed-source app, in-app AI feature without AI Act Art. 50 disclosure architecture, a 1:1 physical-product-mimicry UI without trade-dress search — name it as `Confidence: high` and mark it urgent.

## Phase 4 — Save to the legal log

**Invoke `vault-companion-append`** with:

- `category = "Legal"`
- `body =` the structured issue list from Phase 3 (with every named statute / treaty / guideline / licence wrapped in `[[wikilink]]` form — `[[Lanham Act §43(a)]]`, `[[Copyright Act §107]]`, `[[GDPR]]`, `[[AI Act Art. 50]]`, `[[App Store Review Guideline 5.2]]`, `[[GPL-3.0]]`, `[[AGPL-3.0]]`, `[[MIT License]]`, `[[Madrid Protocol]]`, `[[EUTM]]`, `[[Cal. Civ. Code §3344]]` — so the Legal page accumulates backlinks on each authority's eventual wiki page)
- `frontmatter = { type: "ip-triage", jurisdictions: [<list from Phase 0>], asset_type: "<category from Phase 0>", distribution_channels: [<list from Phase 0>], issue_count: <N>, high_confidence_issues: <N>, regimes: [<list of named regimes in this triage — "trademark" | "trade-dress" | "copyright" | "oss-licence" | "right-of-publicity" | "ai-content" | "app-store-ip" | "ugc-dmca">], patent_questions_routed: <N> }`
- `source_skill = "appdev-ip-triage"`

`vault-companion-append` handles the path (`<vault>/Legal/YYYY-MM-DD-<slug>.md`), `log.md` append, and the optional `obsidian-wiki:ingest` chain (asks the user once per session whether to ingest).

If `vault-companion-append` returns `{ written: false, reason: "no-vault" }` (the user has declined a vault entirely), fall back to writing to `~/.claude/appdev-ip-triage/YYYY-MM-DD-<slug>.md` via the `Write` tool — same body, no log.md (no vault to log into). The log builds a record over time of what was triaged when, which becomes evidence of diligence — especially useful for App Store / Play Store appeal correspondence and for any later opposition or takedown.

## Phase 5 — The take-to-counsel block (mandatory)

Every output ends with this block, edited only to fill in jurisdictions:

> **This is not legal advice.** I am an AI assistant flagging IP issues; I am not licensed to practise law in <jurisdiction(s)>. Before acting on anything in the issue list — including the items I marked low-confidence — take this list to an IP lawyer qualified in the relevant jurisdiction(s). The most expensive IP mistakes in app development come from acting on the assumption that "the licence looked fine" or "the name probably isn't taken"; the cheapest insurance is a 30-minute call with IP counsel before you submit the App Store build.

Do not omit this block to be terse. Do not soften it. The block is the skill's safety guarantee.

## Hard refusals

- **No "is this safe to ship" yes/no answers.** The skill returns issue lists, not verdicts.
- **No software-patent analysis.** Refuse and route to a registered patent attorney. Do not improvise patent-adjacent commentary.
- **No trademark freedom-to-use opinions.** The skill flags risk; the lawyer issues the opinion.
- **No DMCA-strategy advice.** Whether to issue or contest a takedown is outside-counsel work.
- **No case-law interpretation.** The skill names cases for counsel; it does not hold for or against the user.
- **No drafting of final-form regulated content** (final App Store metadata, final EULAs, final Acknowledgements, final AI-content disclosure language) — those need a lawyer's signoff. The skill can sketch a structure or surface a missing clause, never produce final text.
- **No advising on opening a deliberately-friendly entity** to evade IP rules. Refuse and explain.

## What this skill is NOT

- Not a substitute for IP counsel. The skill is the *prep work* that makes a 30-minute call with counsel produce more value than a 3-hour call without it.
- Not a trademark-clearance service. It can flag a clash; only counsel issues a freedom-to-use opinion after a full clearance search.
- Not a patent service. Patent questions are routed without analysis.
- Not a DMCA-takedown service. It flags that DMCA is implicated; it doesn't draft notices.
- Not a licence-compatibility solver. It flags incompatibilities; the resolution (replace the dep, dual-license, release source) is a counsel / engineering decision.

## In Cowork (connector-aware enrichment, Claude for Legal preferred)

This skill benefits from Cowork's document-handling surface and from Anthropic's **Claude for Legal** offering (launched 2026-05-12). When Claude for Legal MCP connectors are granted to the session, the plugin prefers them over generic `WebFetch` for both primary-law lookups and asset / licence-bundle sources. The plugin runs fine without them; it just leans harder on `WebFetch` and lower confidence values.

### Primary-law connectors (verify the regulation, not just name it)

- **Westlaw** (Claude for Legal) — primary US, UK, and EU statute text + trademark filings. Use to verify every named statute and to surface USPTO / EUIPO / WIPO / UKIPO records.
- **Practical Law** (Claude for Legal) — practice notes on Lanham Act §43(a), trade-dress doctrine, OSS-licence enforcement, AI Act, DSM Directive. Useful for confirming the **question for counsel** line in each issue block is current.
- **CoCounsel Legal** (Claude for Legal) — Thomson Reuters' AI-research surface. Use sparingly; the plugin's job is to surface questions for the user's counsel, not delegate the analysis to another AI tool. Prefer Westlaw / Practical Law as primary sources.
- **CourtListener / Free Law Project** (Claude for Legal) — case law. The plugin does **not** interpret case law (hard rule); it cites the relevant case names in the open-jurisdictional-gaps section when Phase 1 surfaces a question that only case law can resolve.
- **`WebFetch`** (fallback, no Claude for Legal) — official primary sources: USPTO TESS for US trademark search; EUIPO eSearch for EU; WIPO Madrid Monitor for international; UKIPO; Rospatent; IPOS; MOEM (UAE); developer.apple.com for App Store Review Guidelines; play.google.com/about/developer-content-policy/ for Play Store; opensource.org and gnu.org for OSS-licence canonical text; copyright.gov for US Copyright Office guidance. Always cite the URL the user can verify.

When a Claude for Legal primary-law connector is granted, **use it before falling back to `WebFetch`**. Conflict between connector text and the plugin's anchor list: the connector wins; append a one-line flag to the issue block: "anchor list reflects pre-`<YYYY-MM-DD>` version of `<authority>`; current text per Westlaw cited inline."

### Asset / document sources

A real asset bundle (icon-pack `.zip` with its EULA, font EULA `.pdf`, OSS-licence ledger `.json`, marketing draft `.docx`) is a much better triage input than a verbal summary. Any of these connectors can be the source for a single triage; the Routine setup command (`/appdev-ip-advisor:setup-ip-triage-routine`) accepts any one of them as the watched-folder source for high-volume work.

- **Box** (Claude for Legal) — common for M&A asset-portfolio diligence.
- **iManage / NetDocuments** (Claude for Legal) — document-management systems used by law firms and corporate legal teams. If the user's IP files already live in one of these, prefer it over Drive.
- **Docusign** (Claude for Legal) — useful when an asset licence is being signed (vendor EULA, talent release form). Surface the issue list **before** signing, not after.
- **Google Drive** (Cowork native connector) — the default for users without a managed legal-document system; common landing place for icon-pack zips and font-EULA PDFs.
- **PDF / file uploads** — Cowork accepts PDF uploads natively. The user can drag a font EULA, an icon-pack licence, or a regulator guidance PDF into the chat and the skill will cite specific paragraphs in the issue list.
- **Image / screenshot uploads** — for trade-dress / visual-mimicry questions, the user can drag a UI screenshot in; the skill reads the visual and surfaces specific design elements (dial layout, button glyphs, motion sequences) it recognises as belonging to identifiable third-party products. **Image analysis is not a trade-dress opinion** — it's an issue-spotter input, take-to-counsel block still applies.
- **Gmail** — generally avoid. Lawyer–client correspondence in inbox is privileged and should not be consulted as routine context.

### Output

- **Markdown issue list** (always) — saved per Phase 4. The canonical artifact.
- **Microsoft Word tracked-change pass** (optional, Claude for Legal) — if the Microsoft connector is granted and the source is a Word doc (typically: draft marketing copy, draft T&Cs, draft Acknowledgements screen text), **additionally** produce a Word file with the issue list surfaced as in-line comments and proposed redlines, saved next to the source as `<original>-redline-<YYYY-MM-DD>.docx`. All edits are tracked changes for attorney review before acceptance — never silent accepts, never auto-applied.

The take-to-counsel block at the end is **non-negotiable regardless of how grounded the analysis is**. Reading the actual licence files / marketing copy against live primary law makes the issues higher-confidence; it does not turn the assistant into a lawyer.

### Routine privacy posture

In a cloud Routine: `ip-triage-on-drive-routine.md` (and its Box / iManage / NetDocuments / Docusign variants — same prompt, different connector) watches a designated folder; when a new asset bundle / spec / licence file / marketing draft lands, it produces the issue list automatically and pings the user to review with counsel before shipping. **Privacy tradeoff: the asset text + image OCR pass through Anthropic's cloud during Routine execution.** If the asset is pre-launch / confidential, run the triage as an interactive session instead. The `/appdev-ip-advisor:setup-ip-triage-routine` command refuses to wire a Routine without explicit non-sensitive-only confirmation.

### Where Claude for Legal does NOT do the plugin's job

Claude for Legal ships 12 practice-area plugins; the named six are commercial, corporate, employment, privacy, IP, litigation. The **IP** plugin covers general IP practice (filing strategy, portfolio management, opposition / cancellation, licensing deals, M&A diligence) but **does not ship app-development-specific checklists** — App Store Review Guideline 5.2 line items, OSS-licence contagion at static-link time, sound-effect chain-of-title in a `.caf` bundle, AI-content disclosure under AI Act Art. 50, F-Droid inclusion-criteria compliance, in-app Acknowledgements-screen attribution surfaces, voice-cloning right-of-publicity stacks. `appdev-ip-advisor` ships those issue-checklists; Claude for Legal supplies the citation-verified primary law underneath them. The two are complementary, not redundant.

## Sources and rationale

- **International** — Berne Convention (1886); Paris Convention (1883); TRIPS (1994); WIPO Copyright Treaty (1996); WIPO Performances and Phonograms Treaty (1996); Madrid Protocol (1989).
- **EU** — EUTM Regulation (EU) 2017/1001; Design Regulation (EC) 6/2002; InfoSoc Directive 2001/29/EC; DSM Directive (EU) 2019/790; Database Directive 96/9/EC; AI Act (Regulation (EU) 2024/1689) Art. 50; GDPR (Regulation (EU) 2016/679) Art. 6/9 likeness overlay.
- **UK** — Trade Marks Act 1994; CDPA 1988; Registered Designs Act 1949; UK GDPR; passing-off doctrine.
- **US (federal)** — Lanham Act (15 USC §§ 1051–1141n); Copyright Act (17 USC §§ 102, 107, 512, 1201); VARA (17 USC § 106A).
- **US (state right-of-publicity)** — Cal. Civ. Code § 3344 + § 3344.1 (CA); N.Y. Civ. Rights Law §§ 50–51 (NY); Tex. Prop. Code Ch. 26 (TX); Fla. Stat. § 540.08 (FL); Tenn. Code § 47-25-1101 et seq. (TN — "Elvis Act").
- **US case anchors** (named, not interpreted) — *Sony Corp. v. Universal City Studios* (1984); *Two Pesos v. Taco Cabana* (1992); *Wal-Mart v. Samara Bros.* (2000); *TrafFix Devices v. Marketing Displays* (2001); *Authors Guild v. Google* (2015); *Star Athletica v. Varsity Brands* (2017); *Apple v. Samsung* (2018); *Andy Warhol Foundation v. Goldsmith* (2023); pending generative-AI line — *Thomson Reuters v. Ross*; *Andersen v. Stability AI*; *NYT v. OpenAI*; *Doe v. GitHub*.
- **Russia** — Civil Code Part IV (Federal Law 230-FZ, 2006), esp. Ch. 70 (copyright) and Ch. 76 (trademarks); Rospatent procedures; 152-FZ likeness/biometric overlay.
- **Singapore** — Trade Marks Act 2005; Copyright Act 2021; PDPA 2012 overlay.
- **UAE** — Federal Decree-Law 36/2021 (Trademarks); Federal Decree-Law 38/2021 (Copyright and Neighbouring Rights); DIFC Law 4/2021 (data protection overlay); ADGM IP regime.
- **App-store policy** — Apple App Store Review Guidelines, esp. § 5.2 (Intellectual Property); Google Play Developer Policy Center — Impersonation, Intellectual Property, Deceptive Behavior.
- **OSS licences** — GPL-2.0 / GPL-3.0 / LGPL-2.1 / LGPL-3.0 / AGPL-3.0 (FSF canonical texts); MIT / BSD-2-Clause / BSD-3-Clause / Apache-2.0 / MPL-2.0 / EPL-2.0 (OSI canonical texts); BSL 1.1 / SSPL-1.0 / Elastic License 2.0 (source-available; texts on the respective project sites).
- **AI-content policy** — US Copyright Office *Guidance on Works Containing AI-Generated Material* (37 CFR Part 202, 2023; 2026 update); *Zarya of the Dawn* refusal letter; EU AI Act Art. 50 (in force 2026-08-02); OpenAI / Anthropic / Stability / Midjourney commercial-use terms (versioned — verify live).

These are the canonical sources the issue checklists are built from. Anything beyond this list requires explicit user confirmation that the source applies to their cell.
