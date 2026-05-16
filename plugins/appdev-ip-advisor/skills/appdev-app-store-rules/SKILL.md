---
name: appdev-app-store-rules
description: Use when an app-development decision touches an app-store policy surface beyond the IP angle — Apple privacy manifest (PrivacyInfo.xcprivacy) and Required Reason APIs, App Privacy nutrition label, Google Play Data Safety form, Account Deletion in-app requirement (Apple §5.1.1(v) / Google Play), In-App Purchase rules and the DMA / Reader-app / User Choice Billing carve-outs, Generative AI app requirements (Apple's age-17+ rule, Google Play AI-content policy, EU AI Act Art. 50 store overlay), age-rating questionnaires (IARC / ESRB / PEGI / USK), restricted categories (gambling, crypto, lending, health, kids, dating, weapons, government), Families / kids policies (Apple Kids Category, Google Play Families, COPPA, GDPR-K, UK Children's Code), UGC moderation gates, sensitive permission declarations (SMS / Location-in-background / Accessibility Service / Notification Listener), Android Vitals thresholds (ANR + crash rate), listing-metadata rules (screenshots, preview video, keyword spam), notarisation and signing (Apple notarisation, Play App Signing, Microsoft AppX/MSIX, Steam DRM, F-Droid reproducible builds, Snap classic-vs-strict confinement), Sherlocked-feature risk, hardware-specific surfaces (visionOS, watchOS, CarPlay, tvOS focus engine, Wear OS, Android Auto, Galaxy DeX), encryption export classification, country-availability, and EU-DMA alternative iOS marketplaces (AltStore PAL, Setapp, Epic Games Store iOS — Core Technology Fee, separate notarisation). Covers Apple App Store (iOS / iPadOS / watchOS / tvOS / visionOS / macOS), Google Play, Microsoft Store, Amazon Appstore, Samsung Galaxy Store, Huawei AppGallery, F-Droid, Flathub, Snap Store, Steam, Epic Games Store. Triggers on "review my app for app-store policy", "check Apple §3.1.1", "Play Console Data Safety review", "what does App Review want for this", "is my privacy manifest complete", "do I need Account Deletion in-app", "can I ship without StoreKit", "kids-category compliance check", "F-Droid Anti-Features audit", "generative AI app review", "AI Act Art. 50 store overlay", "pre-submission checklist for visionOS / watchOS / CarPlay", "Flathub manifest sanity check", "Snap classic confinement justification", "Steam Direct content survey". Issue-spotter and pre-submission planner, not a freedom-to-ship guarantee — every output ends with a take-to-store-policy-review block. Refuses IAP-evasion / DMA-evasion / Play Billing evasion advice, and refuses to draft final-form regulated content (final privacy nutrition label, final Data Safety form, final account-deletion flow text, final age-rating questionnaire answers, final EULAs). Routes IP-rooted findings to appdev-ip-triage, privacy/AI-Act findings to privacy counsel, restricted-category findings to subject-matter counsel. Refuses software-patent analysis — routes to a registered patent attorney.
---

# Appdev App-Store Rules

Pre-submission policy review for app-development work. The skill exists because most rejected builds don't fail because the developer did something egregious — they fail because a Required Reason API category was added without an entry in `PrivacyInfo.xcprivacy`, the Google Play Data Safety form was filled in months ago and the SDK list has drifted since, an account-deletion flow that used to be web-only is now mandatory in-app under Apple §5.1.1(v), or a child-targeted app shipped a third-party analytics SDK without realising the Families policy disallows it. The user usually doesn't need a verdict on whether the build will pass App Review (it might not — App Review is reviewer-specific by design); they need to know which questions belong on the agenda for the team's app-store-policy lead, and which clauses in each store's published rules apply, **before** the submission goes in.

**Announce at start:** "Using the appdev-app-store-rules skill. I'll surface the app-store policy issues and the published rules they live under across your target stores, but I will not give legal advice — IP-rooted findings route to your IP counsel via appdev-ip-triage; privacy / GDPR / AI-Act findings route to your privacy counsel; restricted-category findings (gambling, crypto, health, finance, kids) route to subject-matter counsel. Software patents are out of scope; I'll route those to a patent attorney without analysis. I will not predict whether App Review will pass the build — that's reviewer-specific."

## Phase -1 — Claude for Legal install nudge (one-shot per setup)

Before Phase 0, **check whether Claude for Legal is installed for this session.** Heuristics, in order:

1. Look for a `claude-for-legal:` connector or capability surfaced in the session (Westlaw, Practical Law, CoCounsel, CourtListener, Box, iManage, NetDocuments, Docusign, Microsoft Word).
2. If none is detected, check `~/.claude/appdev-ip-advisor.local.md` for a line like `claude-for-legal-nudge: shown <YYYY-MM-DD>`. If present, skip the nudge.

If Claude for Legal is **not** detected **and** the nudge hasn't been shown before, print exactly once (then record the shown date to `~/.claude/appdev-ip-advisor.local.md` under a `## Setup history` header):

> **One-time setup tip.** Several app-store policies overlay primary law that benefits from citation-grounded verification: Apple's privacy-manifest requirements track GDPR / CCPA; the Generative AI App rules track EU AI Act Art. 50; Account Deletion tracks GDPR Art. 17; the Families / Kids policies track COPPA / GDPR-K / UK Children's Code. Installing Anthropic's **Claude for Legal** alongside this plugin gives the skill access to **Westlaw** / **Practical Law** for the legal overlay, and to **Box / iManage / NetDocuments / Docusign** if your build artifacts and licence bundles live there:
>
> - Cowork → **Customize** → **Browse plugins** → **Legal** → **Install**.
>
> The skill still runs without Claude for Legal — it falls back to `WebFetch` against the store developer portals (developer.apple.com, play.google.com, learn.microsoft.com, etc.). Skipping this step is fine; the nudge will not repeat.

Do **not** block on this nudge. Continue to Phase 0 immediately after printing it.

<HARD-GATE>
Phase 0 (target stores + app category + features-in-scope + distribution-channel mix) is mandatory. App-store policy is intensely store-specific and category-specific — a generative-AI utility on the Apple App Store, a children's game on Google Play, a finance app on the Mac App Store, and a desktop Snap with classic confinement are four different reviews. Refuse to proceed without all four answers.
</HARD-GATE>

<HARD-GATE>
**Patents are out of scope.** If the user asks about software patents — patent infringement, freedom-to-operate, patent clearance, "did we infringe Apple's patent on X" — refuse and route: "This is a software-patent question. The methodology I run does not apply to patents. Take this to a registered patent attorney in the relevant jurisdiction(s)." Do not improvise patent-adjacent analysis even if pushed.
</HARD-GATE>

<HARD-GATE>
**No IAP / Play Billing / DMA evasion advice.** Apple §3.1.1 (StoreKit required for digital goods) and Google Play §4.1 (Play Billing required for digital goods) are the rules. The genuine carve-outs — Apple Reader-app §3.1.3(a), Music-streaming entitlement, External Link Account entitlement, DMA alternative-app-marketplace entitlement, Google Play User Choice Billing in eligible markets, Google Play Reader-app exemption — are the only valid framings. Refuse to brainstorm grey-area schemes: hidden payment surface, dark-UX web checkout, region-shifted billing, separate developer accounts for the same product family. Surface these as a developer-account-termination risk, not a strategy.
</HARD-GATE>

<HARD-GATE>
**No "ship it and see" advice.** App Review tracks resubmission patterns; second rejections on the same clause back-of-queue you. If a policy clause clearly applies, the answer is to comply before submission, not to gamble on the reviewer.
</HARD-GATE>

<HARD-GATE>
**No drafting of final-form regulated content.** Final App Privacy nutrition label text, final Google Play Data Safety form text, final Account Deletion flow copy, final age-rating questionnaire answers, final EULAs, final T&Cs, final in-app Acknowledgements blocks, final AI-content disclosure language — those need your store-policy lead's signoff (and for the legally-loaded ones, counsel's signoff). The skill sketches structure and surfaces missing clauses. It does not produce final text.
</HARD-GATE>

<HARD-GATE>
Every output ends with the **take-to-store-policy-review** block (Phase 5). No exceptions, no softening. The pattern is what makes the skill safe to use.
</HARD-GATE>

<HARD-GATE>
**Citation grounding.** Every named guideline section, policy clause, numerical threshold (Android Vitals ANR / crash rate, Apple TestFlight build cap, Steam Direct fee, F-Droid build-server reproducibility tolerance), Required Reason API category, and AI-Act / GDPR / COPPA overlay article in the issue list must be either (a) verified live against the store's developer-portal page during this run — `WebFetch` against developer.apple.com / play.google.com / learn.microsoft.com / developer.amazon.com / seller.samsungapps.com / developer.huawei.com / f-droid.org / docs.flathub.org / snapcraft.io / partner.steamgames.com / dev.epicgames.com — or against the underlying primary law via Claude for Legal (Westlaw / Practical Law) when granted — or (b) flagged `confidence: low pending verification`. Never assert a guideline number purely from training-data recall. Apple reorders Review Guideline sections at each WWDC; Google Play Vitals thresholds are revised yearly; the F-Droid Anti-Features list grows; the DMA entitlement surface is still evolving.
</HARD-GATE>

## Phase 0 — Stores, app category, features-in-scope, distribution-channel mix (mandatory)

Ask, in order:

1. **Which stores are in scope?** Multi-pick — every store is its own review:
   - **Apple App Store** — iOS, iPadOS, watchOS, tvOS, visionOS. The App Store Review Guidelines are the unified ruleset; hardware-specific surfaces (visionOS spatial-computing, watchOS complications, CarPlay, tvOS focus engine) add platform-specific clauses.
   - **Mac App Store** — same Guidelines + sandboxing requirement + Apple-notarisation (separate from notarisation for Developer-ID-direct distribution).
   - **Google Play Store** — Developer Policy Center.
   - **Microsoft Store** (Windows 10 / 11) — Microsoft Store Policies; Partner Center certification.
   - **Amazon Appstore** — Amazon Appstore Policy Center; Fire OS-specific overlays.
   - **Samsung Galaxy Store** — Samsung Galaxy Store Seller Office certification; Knox / DeX overlays where relevant.
   - **Huawei AppGallery** — AppGallery Review Guideline; HMS replacing GMS for the China surface.
   - **F-Droid** — Inclusion Policy; Anti-Features tagging; reproducible-build requirement.
   - **Flathub** (Linux Flatpak) — Submission guidelines; manifest validation.
   - **Snap Store** (Ubuntu / Linux) — strict vs. classic confinement; snapd interfaces.
   - **Steam** — Steamworks; Steam Direct ($100 fee + content survey); revenue tiers.
   - **Epic Games Store** — Epic submission guidelines; 12% revenue share.
   - **EU-DMA alternative iOS marketplaces** (AltStore PAL, Setapp, Epic Games Store iOS) — Apple notarisation (separate from App Store review); Core Technology Fee; DMA-specific entitlements.

   If the user says "everywhere", push back: pick the top 3 by user count or revenue and treat them as the working set. The plugin's job shrinks if you spread thin.

2. **What does the app do?** One sentence. The category sets the restricted-policy overlays:
   - **Utility / productivity / developer tool** — the default lane; minimal category overlay.
   - **Game** — Apple §5.3 (Gaming, Gambling, and Lotteries), age-rating questionnaire overlay, loot-box-odds disclosure if applicable.
   - **Social / messaging / dating** — UGC overlay, content moderation, blocking/reporting required, named-contact for moderation issues.
   - **Financial services / lending / payments / crypto** — Apple §3.1.5(b) crypto, Google Play financial-services overlay, regional licensing.
   - **Health / medical / research** — Apple §1.4.1 / §5.1.1 health-research; HIPAA overlay (US), MDR (EU).
   - **Kids / education / children's content** — Apple Kids Category, Google Play Families, COPPA, GDPR-K, UK Children's Code.
   - **Generative AI / image-generation / chatbot / voice-cloning** — Apple's "Generative AI App" requirements (content moderation + age 17+ for unmoderated outputs), Google Play AI-generated content policy, EU AI Act Art. 50 disclosure (in force 2026-08-02).
   - **News / journalism / political** — Apple §5.6 (developer code of conduct), political-ad transparency overlays per jurisdiction.
   - **Government / public-sector** — Apple §5.4 / §5.5 (VPN / law-enforcement), regional-procurement overlays.

3. **Which features are in scope?** Multi-pick. Each one pulls in a checklist cluster:
   - **UGC** — moderation, reporting, blocking, named contact, DMCA designated agent.
   - **Payments — in-app digital** — Apple §3.1 / Play §4.1; DMA / Reader-app / User Choice Billing carve-outs.
   - **Payments — physical goods or out-of-app services** — Apple §3.1.3(e), Play physical-goods carve-out.
   - **Subscriptions** — disclosure language, free-trial UX, cancellation accessible in-app.
   - **Sensitive permissions** — SMS, Call Log, Location-in-background, Accessibility Service, Notification Listener (Android Play Console declaration); Apple privacy-relevant Info.plist usage-description strings.
   - **Background activity** — background fetch / background audio / location-while-suspended / push-notification-as-trojan-update.
   - **Health / biometric data** — Apple HealthKit terms, Apple Sign-in-with-Apple biometric overlay, GDPR Art. 9 special-category data.
   - **AI-generated content output to users** — disclosure architecture; age-17+ if unmoderated; AI Act Art. 50 store overlay.
   - **Children's audience** — Kids Category / Families overlay.
   - **Location tracking** — precise location, background location, foreground-only — each gates a different surface.
   - **Camera / microphone in background** — Apple §5.1.2(i), Play foreground-services overlay.
   - **Advertising / SDK monetisation** — IDFA (Apple ATT), Google Play Ads ID, child-targeted-ads ban in Kids/Families.
   - **Push notifications** — no marketing-only spam (Apple §4.5.4), Play notification-misuse policy.
   - **External links** — DMA External Link Entitlement (Apple), out-of-app web browsing.
   - **Third-party authentication** — if any third-party sign-in is offered, Apple §4.8 requires Sign in with Apple as an option.
   - **File sharing / cloud sync** — Apple iCloud / CloudKit terms; security overlay.
   - **Hardware-specific surface** — Vision Pro (visionOS), watchOS complications, CarPlay, tvOS focus engine, Android Auto, Wear OS, Galaxy DeX, Samsung Knox.

4. **Distribution-channel mix.** Same multi-pick as Phase 0 of `appdev-ip-triage`, used here to bias which store-policy clusters get prioritised: Apple App Store / Mac App Store / Google Play / Microsoft Store / Amazon Appstore / Samsung Galaxy Store / Huawei AppGallery / F-Droid / Flathub / Snap Store / Steam / Epic Games Store / EU-DMA alt iOS marketplace / cross-platform.

If the user is in a store × category combination the skill doesn't have grounded knowledge of (small regional or OEM-bundled stores — Xiaomi GetApps, Oppo App Market, Vivo App Store, Tencent MyApp, Tap-tap, Aptoide), **say so explicitly**: "I don't have reliable knowledge of <store>'s rules for <category>. The methodology below still applies — I can flag categories of issue at the universal level, but the specific clause numbers will need to come from the store's developer-portal or your app-store policy lead."

## Phase 0.5 — Vault Recall (if vault configured)

Before walking the per-store checklist, check whether the user has prior reviews in the same (store × category) cell.

1. **Invoke `vault-companion-ensure`** silently — returns immediately if a vault is already configured. If the user has previously declined a vault (handle is `null`), skip the rest of this phase and proceed to Phase 1.
2. **Invoke `vault-companion-recall`** with topic = `"<top store from Phase 0>" + " " + "<category from Phase 0>" + " store-policy"`. Cap matches at 5. Use `category_preference: AppStore`.
3. If matches is non-empty, weave them into the Phase 1 framing — e.g.:
   > I see <N> prior store-policy reviews in your AppStore/ folder on the same (store × category) cell — most recent: `<path>` from `<date>`. Want me to surface what issues came up before walking the checklist again? (yes / no / skim only)
4. If matches is empty, proceed silently to Phase 1.

Recall is enrichment, not a gate. Never block Phase 1 on this.

## Phase 1 — Pick the per-store checklist cluster

Match (store × app-category × features) to the relevant policy clusters. Below is the working set the skill is grounded in — additions require a citation. **Live-verify every cited section number** against the store's developer-portal page before it appears in the output.

### Apple App Store (iOS / iPadOS / watchOS / tvOS / visionOS) and Mac App Store

Canonical source: developer.apple.com/app-store/review/guidelines/

Universal clusters:
- **§1 Safety** — objectionable content (§1.1), UGC moderation (§1.2 — reporting + blocking + named contact + remove-without-notice authority), kids category (§1.3), physical harm (§1.4 — incl. health/medical at §1.4.1), developer info (§1.5), data security (§1.6).
- **§2 Performance** — app completeness (§2.1, including the dreaded §2.1 metadata rejection class), beta testing (§2.2), accurate metadata (§2.3), hardware compatibility (§2.4), software requirements (§2.5 — including ATS, IPv6, IDFV, and the privacy-manifest requirement under §2.5.14).
- **§3 Business** — Payments (§3.1), Other Business Model Issues (§3.2).
- **§4 Design** — Copycats (§4.1), Minimum Functionality (§4.2), Spam (§4.3), Extensions (§4.4), Apple Services (§4.5), Alternate Languages (§4.6), HyperCard / Mini-Apps (§4.7 — the JavaScript/HTML5-runtime carve-out), Sign in with Apple (§4.8), Streaming Games (§4.9).
- **§5 Legal** — Privacy (§5.1, including the in-app Account Deletion requirement at §5.1.1(v)), Intellectual Property (§5.2 — handed to `appdev-ip-triage`), Gaming/Gambling/Lotteries (§5.3), VPN Apps (§5.4), Mobile Device Management (§5.5), Developer Code of Conduct (§5.6).

Feature-specific clusters:
- **Privacy nutrition label** — declared per data type at App Store Connect upload; declarations must match observed app behaviour. Inconsistency between the nutrition label and the actual data flow is a §2.3 / §5.1 rejection.
- **Privacy manifest (`PrivacyInfo.xcprivacy`)** — required for apps and SDKs since 2024. Declares: tracking domains, privacy-tracking data, collected data types, and **Required Reason API** usage. The 5 Required Reason API categories at the time of writing are File Timestamp APIs, System Boot Time APIs, Disk Space APIs, Active Keyboard APIs, and User Defaults APIs — verify the list and the per-API permitted reasons live against developer.apple.com before citing.
- **App Tracking Transparency (ATT)** — IDFA access requires the ATT prompt + Info.plist `NSUserTrackingUsageDescription`. SDKs that fingerprint as a workaround are a §5.1.2 rejection.
- **Sign in with Apple §4.8** — if the app offers any third-party sign-in (Google / Facebook / Apple's third-party social), Sign in with Apple must be offered too, with comparable prominence.
- **In-App Purchase §3.1** — digital goods/services in-app must use StoreKit (Apple's commission tiers apply). Genuine carve-outs: Reader-app §3.1.3(a); Music-streaming entitlement; External Link Account entitlement (regional, narrow); DMA External Linking and Alternative Marketplace entitlements (EU only, since 2024); Multiplatform Services §3.1.3(b). Anti-steering language has shifted post-*Epic v. Apple*; verify the current §3.1.3 text live.
- **Subscriptions §3.1.2** — disclosure language (price + period + cancellation), free-trial UX, in-app cancellation surface.
- **Account Deletion §5.1.1(v)** — apps that support account creation must offer in-app account deletion. Web-only deletion is not sufficient. The deletion must also delete server-side data, not just close the local session.
- **Generative AI App requirements** — apps that can produce AI-generated outputs must (a) have content-moderation in place to prevent objectionable content, (b) carry an age 17+ rating if users can produce unmoderated outputs. Verify the current Guideline language live (the rules were tightened in 2024 and may have moved further).
- **TestFlight** — no live commerce, 90-day build cap, 10,000-user cap. The cap numbers should be verified live.
- **Hardware surfaces** — visionOS Human Interface Guidelines for spatial-computing apps; watchOS complications guidelines; CarPlay (separate entitlement, separate review); tvOS focus engine (D-pad navigation requirements).
- **EU-DMA notarisation** — alternative iOS marketplaces (AltStore PAL, Setapp, Epic Games Store iOS) ship apps through Apple notarisation (a smaller technical-review pass than App Store review). Notarisation rejection is a separate failure mode; the Core Technology Fee structure is the business-side overlay.

### Google Play Store

Canonical source: play.google.com/about/developer-content-policy/

Universal clusters:
- **Restricted Content** — child endangerment, inappropriate content, hate speech, violence, regulated goods (alcohol / gambling / financial services / health / unapproved substances).
- **Impersonation and Intellectual Property** — handed to `appdev-ip-triage` for the IP angle; the Impersonation rule on icon/name/description belongs to both.
- **Privacy, Deception, and Device Abuse** — Data Safety form, sensitive permissions, user-data policy, malware, mobile-unwanted-software, deceptive behaviour.
- **Monetization and Ads** — Play Billing required for digital (with Reader-app exemption and User Choice Billing in eligible markets); ads policy including ad-format restrictions.
- **Store Listing and Promotion** — metadata accuracy, screenshot policy, app icon, app title, preview video.
- **Families** — Designed for Families, child-safety SDK self-certification, third-party ad / analytics ban for child-targeted apps.
- **Spam and Minimum Functionality** — clone-app + low-effort-app policy.

Feature-specific clusters:
- **Data Safety form** — declared per data type at Play Console upload; must match SDK behaviour. Mismatch is a frequent rejection. The Data Safety form is the Play equivalent of Apple's nutrition label; cross-store consistency is the carry-over to flag.
- **Account Deletion** — requires both in-app deletion and an accessible web URL where users can request deletion without logging in. The web URL must be linked from the Play Console listing.
- **Sensitive Permissions Declaration** — declared in Play Console for: SMS / Call Log (the All-Files-Access / MANAGE_EXTERNAL_STORAGE for Android 11+), Location-in-background, Accessibility Service (only for accessibility use cases), Notification Listener, VPN Service, Health Connect data types. Mis-declaration is a takedown.
- **Android Vitals thresholds** — bad-behaviour thresholds for ANR rate and crash rate (verify the current numerical thresholds live; they were ANR ≥ 0.47% and crash ≥ 1.09% at one point but Google revises them).
- **AI-generated content policy** — AI-content apps must allow user reporting of objectionable AI output; SDK-level content-moderation expectations.
- **User Choice Billing** — available in EEA, India, Indonesia, Japan, Brazil, USA (since 2025) per the regional rollout; the user-choice option must be presented neutrally alongside Play Billing.
- **Families Policy** — child-targeted apps must self-certify SDKs against the Families SDK programme (no third-party analytics not on the allowlist; no behavioural ads to children).

### Microsoft Store (Windows)

Canonical source: learn.microsoft.com/en-us/windows/uwp/publish/store-policies

Universal clusters:
- **General requirements** — accurate functionality, no malware, appropriate content rating, accessibility statement (encouraged).
- **Capability declarations** — UWP/MSIX apps declare capabilities; restricted capabilities require business justification at submission.
- **Commerce** — Microsoft Store commerce (15% / 12% / 0% revenue tiers depending on category); non-game apps may use third-party commerce engines.
- **Listing** — screenshots, store images, promotional copy.
- **Certification** — Partner Center certification report names the specific policy section that failed.

Feature-specific clusters:
- **Privacy statement URL** — required for any app that collects personal information.
- **Signing** — AppX/MSIX signing certificate; symbol uploads for crash analytics.

### Amazon Appstore

Canonical source: developer.amazon.com (Appstore Policy Center)

- Fire OS-specific overlay: apps targeting Fire tablets / Fire TV have hardware-specific clauses (no Play Services available).
- Underage user policy; restricted-content policy; in-app purchase via Amazon's billing system on Fire OS.
- Re-submission is faster than App Store but the policy surface tracks Google Play closely.

### Samsung Galaxy Store

Canonical source: seller.samsungapps.com

- Galaxy-specific overlays for Watch, DeX, Knox, S Pen.
- The Seller Office certification feedback names the failed clause.
- Galaxy Store has its own age-rating questionnaire; mis-rating is a fast rejection.

### Huawei AppGallery

Canonical source: developer.huawei.com

- HMS (Huawei Mobile Services) replaces GMS for the China-distribution surface — apps that depend on Google Play Services for FCM, Maps, Sign-in, etc., need HMS-equivalent integration for the AppGallery distribution.
- The AppGallery Review Guideline overlays Chinese content rules; political and certain news categories have additional gates.

### F-Droid

Canonical source: f-droid.org/docs/Inclusion_Policy/ and f-droid.org/docs/Anti-Features/

- **Inclusion Policy** — only FOSS apps; all build dependencies must themselves be FOSS; build must be from source on the F-Droid build server (no proprietary blobs).
- **Anti-Features tagging** — every app's manifest declares any of: NonFreeNet (talks to a non-free network service), NonFreeAdd (mandates installing non-free software), NonFreeDep, NonFreeAssets, Tracking, UpstreamNonFree (upstream is non-free even if F-Droid's build is patched), Ads, KnownVuln, DisabledAlgorithm, NoSourceSince. Missing an Anti-Feature that applies is a rejection.
- **Reproducible builds** — the F-Droid build server's output must match the developer's locally-signed APK byte-for-byte (or within a documented tolerance). Reproducibility failure means the F-Droid build can't be cross-signed with the developer's key, breaking the upgrade path for users coming from Play.
- **No Google Play Services dependency** — anything that pulls Play Services as a hard dep can't ship on F-Droid; the polite framing is "shim it behind a build flavour".

### Flathub (Linux Flatpak)

Canonical source: docs.flathub.org/docs/for-app-authors/submission/

- **Manifest validation** — the Flatpak manifest must build cleanly on Flathub's build infrastructure; runtime + extension dependencies must be available in the chosen runtime version.
- **App ID convention** — reverse-DNS, owner-controlled domain.
- **Permissions / sandbox** — Flatpak's portal-based permission model; broad filesystem access (`--filesystem=home`) requires justification.
- **End-user data hygiene** — apps may not exfiltrate data without disclosure.

### Snap Store (Ubuntu / Linux)

Canonical source: snapcraft.io/docs

- **Strict vs. classic confinement** — strict is the default; classic confinement (full system access, no sandbox) requires manual review by the Snap Store team. Classic-confinement requests need a written engineering justification (e.g., "this is a system administration tool that legitimately needs root").
- **snapd interfaces** — plugs and slots; auto-connections to certain interfaces require an exception.
- **Auto-update model** — users do not opt out of snap updates; the responsibility for not breaking the user's installation falls on the publisher.

### Steam

Canonical source: partner.steamgames.com/doc/home

- **Steam Direct** — $100 one-time fee per app; content survey questionnaire; Valve reserves the right to refuse listings.
- **Revenue tiers** — 30% standard / 25% past $10M lifetime / 20% past $50M lifetime per app.
- **Content policy** — adult-content gating (separate adult-only Steam category requires a separate workflow), no malware, no asset-flips at scale.
- **Steam Workshop** — UGC for games that integrate Workshop; content moderation is the developer's responsibility.

### Epic Games Store

Canonical source: dev.epicgames.com/docs/epic-games-store/

- **12% revenue share** standard (notable lower than Apple/Google/Steam).
- **Submission gates** — content rating; achievements/leaderboards integration via Epic Online Services optional.
- **UEFN overlap** — Unreal Editor for Fortnite content is governed separately.

### Cross-store consistency cluster (carry-over check)

- **Privacy disclosures** — Apple App Privacy nutrition label vs. Google Play Data Safety form vs. Microsoft Store privacy statement. The same underlying SDK behaviour should produce consistent disclosures across all three. Drift between them is both a rejection risk per-store and a regulatory red flag (GDPR, CCPA).
- **Account Deletion** — Apple §5.1.1(v) and Google Play policy both require in-app deletion. Implement once with both surfaces in mind.
- **IAP / billing rules** — Apple §3.1.1 vs Google Play §4.1 — the carve-outs differ (Reader-app exists on both with different texts; DMA carve-outs are Apple-only; User Choice Billing is Google-only). Don't conflate.
- **Age ratings** — Apple uses its own questionnaire; Google Play uses IARC (one form, multiple regional rating boards); Microsoft uses IARC too; Steam uses its own; F-Droid doesn't rate. A change in app content can require re-rating across multiple stores in parallel.
- **AI-content disclosure** — Apple Generative AI App rules + Google Play AI-content policy + EU AI Act Art. 50 (in force 2026-08-02) all apply to the same generative output. The strictest framing wins for design.
- **Sensitive permission justifications** — Apple Info.plist usage strings + Android runtime permission rationale strings + Play Console Sensitive Permissions declaration form should all tell the same story for the same permission use.

## Phase 2 — Walk the issue checklist for the matched cell

For each matched cluster, ask the user a short yes/no series. Below is the canonical set; pick the relevant subset for the (store × category × features) cell.

### Apple App Store / Mac App Store

1. **Privacy manifest** — does `PrivacyInfo.xcprivacy` exist in every shipped app + framework? Are all Required Reason API categories used by the app or any embedded SDK declared with the permitted reason code?
2. **App Privacy nutrition label** — does every data type collected by the app or any embedded SDK appear in the App Store Connect privacy declaration, with linking-to-user / not-linked correctly distinguished?
3. **ATT** — if the app accesses IDFA or otherwise tracks across apps, is the ATT prompt shown with `NSUserTrackingUsageDescription` set in Info.plist?
4. **Sign in with Apple §4.8** — if the app offers any third-party sign-in option, is Sign in with Apple offered with comparable prominence?
5. **Account Deletion §5.1.1(v)** — if the app supports account creation, is there an in-app account-deletion flow that also deletes server-side data?
6. **IAP §3.1** — are all digital goods/services routed through StoreKit, or is the app squarely within a documented carve-out (Reader-app §3.1.3(a) / Music-streaming entitlement / External Link Account entitlement / DMA entitlement / Multiplatform Services §3.1.3(b))?
7. **Subscription disclosure §3.1.2** — does the subscription paywall clearly state price + period + auto-renewal + how to cancel before the user can subscribe?
8. **Metadata §2.3** — are the screenshots, preview video, name, subtitle, and keyword string an accurate representation of what the app actually does on first launch?
9. **Minimum Functionality §4.2** — is the app more than a wrapper around a web view of an existing website? If web-view-heavy, what's the native value-add?
10. **Generative AI** — if the app produces AI-generated content, is content moderation in place, and is the age rating 17+ if users can produce unmoderated outputs?
11. **Hardware surface (if relevant)** — visionOS HIG conformance for spatial-computing surfaces; CarPlay entitlement granted; watchOS complications follow the platform conventions; tvOS focus-engine compliance.
12. **TestFlight** — if shipping via TestFlight first, is the build under the 90-day cap and the user count under the cap?
13. **Notarisation (Mac)** — for Mac App Store, sandboxing entitlements declared; for Developer-ID-direct distribution, notarisation passed.

### Google Play Store

1. **Data Safety form** — does every data type collected by the app or any embedded SDK appear in the Play Console Data Safety form? Has it been updated since the most recent SDK upgrade?
2. **Account Deletion** — is there both an in-app deletion flow AND a publicly accessible web URL where deletion can be requested without logging in? Is that URL linked from the Play Console listing?
3. **Sensitive Permissions** — does any sensitive permission (SMS / Call Log / Location-in-background / Accessibility Service / Notification Listener / VPN Service / Health Connect data types / All-Files-Access) have a Play Console declaration with a justification matching the in-app usage?
4. **Android Vitals** — what's the current ANR rate and crash rate against Google's bad-behaviour thresholds (verify the current numbers live)? Are they below the thresholds for the listed devices?
5. **Billing §4.1** — are digital goods routed through Play Billing, or is the app squarely within Reader-app exemption / User Choice Billing in an eligible market?
6. **Listing accuracy** — does the screenshot set + preview video + app description + app icon reflect what the app actually does? No keyword spam in the description?
7. **Families Policy** — if child-targeted (Designed for Families opt-in), are all SDKs on the Families SDK allowlist? Has third-party advertising been removed?
8. **AI-content policy** — if the app produces AI-generated content, is there a user-reporting mechanism for objectionable output?
9. **Privacy policy URL** — is the privacy policy URL set in Play Console and live?

### Microsoft Store

1. **Capability declarations** — restricted capabilities have business justifications at submission?
2. **Privacy statement URL** — set if any personal information is collected?
3. **Signing** — AppX/MSIX signing chain valid?
4. **Listing** — accurate, with appropriate content rating?

### Amazon Appstore

1. **Fire OS hardware adaptation** — if targeting Fire tablets / Fire TV, are GMS dependencies removed or replaced with Amazon alternatives?
2. **In-app purchase via Amazon billing** — if monetising on Fire OS?
3. **Listing + age rating?**

### Samsung Galaxy Store

1. **Galaxy-specific surfaces** — Watch, DeX, S Pen, Knox — declared where used?
2. **Galaxy Store age rating questionnaire** — completed accurately?

### Huawei AppGallery

1. **HMS replacement** — for China-distribution, are GMS dependencies replaced with HMS (Huawei Mobile Services)?
2. **Content overlays** — political / news / map / VPN clauses checked against Chinese content rules?

### F-Droid

1. **FOSS-only deps** — all build-time and runtime dependencies are themselves FOSS?
2. **Anti-Features manifest** — does the manifest declare every applicable Anti-Feature (NonFreeNet, Tracking, etc.)?
3. **Reproducible build** — does the F-Droid build server's output match the developer's locally-built APK?
4. **No Google Play Services hard-dep** — Play Services pulled behind a build flavour or stripped entirely?

### Flathub

1. **Manifest validation** — builds cleanly on Flathub infrastructure?
2. **Sandbox justification** — broad-filesystem / device-access permissions have an "Additional permissions" justification in the appdata?
3. **App ID conformance** — reverse-DNS with owner-controlled domain?

### Snap Store

1. **Confinement** — strict by default? If classic, is the engineering justification written and ready for the manual review?
2. **Auto-update tolerance** — has the snap been tested for "user-doesn't-opt-in" update rollout?

### Steam / Epic Games Store

1. **Content survey** — completed accurately for Steam Direct?
2. **Content rating** — appropriate?
3. **Anti-cheat / DRM / Workshop / EOS integration** — declared at the right level?

### Cross-store carry-over

1. Are the Apple privacy nutrition label, Google Play Data Safety form, and Microsoft Store privacy statement telling the same story about the same SDKs?
2. Is the Account Deletion flow built once with both Apple §5.1.1(v) and Google Play in mind, including the web URL Google requires?
3. Are the IAP / Play Billing / DMA-entitlement framings consistent — and are the carve-outs (Reader-app, Music-streaming, User Choice Billing) being claimed only where they actually apply?
4. Have the age ratings been re-checked across all stores after the most recent content change?
5. Has the AI-content disclosure architecture been built against the strictest of Apple's Generative AI App rules, Google Play AI-content policy, and EU AI Act Art. 50?
6. Are sensitive-permission justifications (Apple Info.plist usage strings, Android runtime rationale, Play Console Sensitive Permissions declaration) telling the same story?

## Phase 3 — Output the issue list

Format the output as a structured issue list, **not** a verdict. Each issue cites the store + section number live-verified and names the specific artifact to change.

```markdown
# Appdev app-store policy review — <one-sentence topic>
Date: <YYYY-MM-DD>
Stores in scope: <list>
App category: <category from Phase 0.2>
Features in scope: <list from Phase 0.3>
Distribution channels: <list from Phase 0.4>

## Issues identified

### Issue 1 — <short title>
- **What's at risk:** <one sentence — rejection / takedown / certification fail / re-review queue penalty>
- **Store + clause:** <store + section number + URL verified live this run>
- **What to change:** <specific artifact + key — e.g., "PrivacyInfo.xcprivacy → NSPrivacyAccessedAPITypes → add NSPrivacyAccessedAPICategoryUserDefaults entry with reason CA92.1">
- **Question for store-policy review:** <the specific question>
- **Cross-store carry-over:** <other stores in the channel mix that will also flag this if shipped unchanged — name the equivalent clause>
- **Confidence this is in scope:** <high / medium / low>

### Issue 2 — ...

## Cross-store carry-over summary
<table or list: per-store, which issues from above apply, with the equivalent clause named for each>

## Issues explicitly considered and ruled out of scope (with reason)
<two or three; explicit-negative is better than silent — e.g., "ATT — N/A because the app does not access IDFA; verify SDKs have not regressed">

## Open store gaps
<stores the user named where the assistant lacks reliable knowledge — flagged so the store-policy lead and counsel know>

## IP-rooted findings routed to appdev-ip-triage
<list any §5.2-style IP findings; the rules skill flags them; the IP skill triages them>

## Privacy / GDPR / AI-Act findings routed to privacy counsel
<list any disclosure-architecture findings that go beyond store-policy formatting into the underlying privacy law>

## Restricted-category findings routed to subject-matter counsel
<gambling, crypto, health, finance, kids — the rules skill flags them; subject-matter counsel handles them>

## Patent questions surfaced (routed without analysis)
<list any patent-adjacent questions the user raised; the skill refuses to opine, routes to patent attorney>
```

If the user is doing something that looks like a special-attention trigger — Required Reason API used without declaration, no Account Deletion in-app, IAP outside StoreKit without a valid carve-out, child-targeted app with third-party advertising, generative-AI feature without age-17+ rating or moderation — name it as `Confidence: high` and mark it urgent.

## Phase 4 — Save to the AppStore log

**Invoke `vault-companion-append`** with:

- `category = "AppStore"`
- `body =` the structured issue list from Phase 3 (with every named guideline section / policy clause / regulation wrapped in `[[wikilink]]` form — `[[Apple App Store Review Guideline 5.1.1(v)]]`, `[[Apple App Store Review Guideline 3.1.1]]`, `[[Apple App Store Review Guideline 4.8]]`, `[[Apple Privacy Manifest]]`, `[[Apple Required Reason API]]`, `[[Apple App Tracking Transparency]]`, `[[Google Play Account Deletion Policy]]`, `[[Google Play Data Safety]]`, `[[Google Play Billing §4.1]]`, `[[Google Play Sensitive Permissions]]`, `[[Google Play Families Policy]]`, `[[Android Vitals]]`, `[[Microsoft Store Policy]]`, `[[Amazon Appstore Policy]]`, `[[Samsung Galaxy Store Policy]]`, `[[Huawei AppGallery Review Guideline]]`, `[[F-Droid Inclusion Policy]]`, `[[F-Droid Anti-Features]]`, `[[Flathub Submission Guidelines]]`, `[[Snap Store Classic Confinement]]`, `[[Steam Direct]]`, `[[Epic Games Store Submission]]`, `[[DMA External Link Entitlement]]`, `[[DMA Alternative Marketplace Entitlement]]`, `[[EU AI Act Art. 50]]`, `[[COPPA]]`, `[[GDPR Art. 17]]`, `[[UK Children's Code]]` — so the AppStore page accumulates backlinks on each authority's eventual wiki page)
- `frontmatter = { type: "store-policy-review", stores: [<list from Phase 0>], app_category: "<from Phase 0>", features: [<list from Phase 0>], distribution_channels: [<list from Phase 0>], issue_count: <N>, high_confidence_issues: <N>, cross_store_carryovers: <N>, ip_routed_to_triage: <N>, privacy_routed_to_counsel: <N>, restricted_category_routed: <N>, patent_questions_routed: <N> }`
- `source_skill = "appdev-app-store-rules"`

`vault-companion-append` handles the path (`<vault>/AppStore/YYYY-MM-DD-<slug>.md`), `log.md` append, and the optional `obsidian-wiki:ingest` chain.

If `vault-companion-append` returns `{ written: false, reason: "no-vault" }`, fall back to writing to `~/.claude/appdev-app-store-rules/YYYY-MM-DD-<slug>.md` via the `Write` tool — same body, no log.md.

The `AppStore/` category is the natural sibling of the `Legal/` category used by `appdev-ip-triage` and `fintech-legal-advisor`. Same vault, different folder, different `type` frontmatter (`store-policy-review` here vs. `ip-triage` vs. `legal-triage`).

## Phase 5 — The take-to-store-policy-review block (mandatory)

Every output ends with this, edited only to fill in store names:

> **This is a store-policy review, not legal advice and not a freedom-to-ship guarantee.** I am an AI assistant flagging what each store's published policies say and what to change in the build / metadata / privacy disclosures; I am not a substitute for your company's app-store policy lead, not a substitute for the store's own review team, and I cannot predict whether <store(s)> will pass the submission — App Review is reviewer-specific by design. Before submitting:
>
> 1. Get this list in front of the person on your team who owns store relationships.
> 2. For IP-rooted findings, run them through the `appdev-ip-triage` skill and take them to your IP lawyer.
> 3. For privacy / GDPR / AI-Act findings, take them to your privacy lawyer.
> 4. For restricted-category items (gambling, crypto, health, finance, kids, dating), get the relevant subject-matter counsel involved before submission.
> 5. The cheapest insurance against rejection is a 30-minute review with someone on your team who has shipped on the same store before, not "ship it and see".

Do not omit this block to be terse. Do not soften it. The block is the skill's safety guarantee.

## Hard refusals

- **No yes/no "will the resubmit pass" predictions.** App Review is reviewer-specific. Surface the issue list; let the store-policy lead and the actual submission be the source of truth.
- **No IAP-bypass / DMA-evasion / Play Billing evasion advice.** The carve-outs exist; use them. Grey-area schemes risk developer-account termination.
- **No "ship it and see" advice.** Second rejections on the same clause back-of-queue you.
- **No software-patent analysis.** Refuse and route to a registered patent attorney.
- **No drafting of final-form regulated content** (final privacy nutrition label, final Data Safety form text, final account-deletion flow copy, final age-rating questionnaire answers, final EULAs, final AI-content disclosure language). The skill sketches structure; final text needs store-policy / counsel signoff.
- **No advising on opening a separate developer account / shell company / bundle-ID rotation** to evade store enforcement. Refuse and explain why this is a developer-account-termination risk.
- **No case-law interpretation.** The skill may name a relevant case (e.g., *Epic v. Apple* in the context of anti-steering language) so counsel can pick up the thread; it does not interpret holdings.
- **No DMCA-strategy advice.** Whether to issue or contest a takedown is outside-counsel work.

## What this skill is NOT

- Not a substitute for the store's actual review team or your company's app-store policy lead.
- Not a freedom-to-ship guarantee. The skill flags issues; it does not certify the build will pass.
- Not a final-form-content drafter. Structure-sketcher only.
- Not a workflow for evading store enforcement. Carve-outs are the only valid framing.
- Not an appeal-writing service (the `appdev-app-store-rejection-triage` sibling skill handles received rejections and sketches appeal structure; final appeal text is the store-policy lead's call).
- Not an IP triage (that's `appdev-ip-triage`).
- Not a privacy-counsel substitute (the skill flags disclosure-architecture gaps; the disclosure architecture itself is privacy counsel's call).
- Not a patent service.

## In Cowork (connector-aware enrichment, Claude for Legal preferred)

This skill benefits from Cowork's document-handling surface and from Anthropic's **Claude for Legal** offering (launched 2026-05-12). When Claude for Legal MCP connectors are granted to the session, the plugin prefers them for the legal overlays on store policy (privacy, AI Act, GDPR, COPPA / GDPR-K / UK Children's Code) and for asset / draft-document sources.

### Primary-law / policy connectors

- **Westlaw** (Claude for Legal) — primary EU / UK / US statute text for the legal overlays (AI Act, GDPR, COPPA, CCPA, UK Children's Code).
- **Practical Law** (Claude for Legal) — practice notes on Apple's privacy-manifest enforcement track, Google Play Data Safety drift, DMA implementation, AI Act Art. 50 deepfake-disclosure scope.
- **`WebFetch`** (always available, the primary source for store policy itself) — developer.apple.com/app-store/review/guidelines/ for Apple; play.google.com/about/developer-content-policy/ for Google; learn.microsoft.com/en-us/windows/uwp/publish/store-policies for Microsoft; developer.amazon.com for Amazon; seller.samsungapps.com for Samsung; developer.huawei.com for Huawei; f-droid.org for F-Droid; docs.flathub.org for Flathub; snapcraft.io for Snap; partner.steamgames.com for Steam; dev.epicgames.com for Epic. Always cite the URL the user can verify.

When a Claude for Legal primary-law connector is granted, **use it before falling back to `WebFetch`** for the legal overlay. For store-policy text itself, `WebFetch` against the developer portal remains the authoritative source — the stores publish their own policies; no legal-research connector resells them.

### Asset / draft-document sources

A real release-candidate bundle (build artifact, `Info.plist`, `PrivacyInfo.xcprivacy`, AndroidManifest.xml, listing-copy `.docx`, screenshot set) is a much better review input than a verbal summary.

- **Box** (Claude for Legal) — common for cross-team release-candidate libraries.
- **iManage / NetDocuments** (Claude for Legal) — where T&Cs / privacy-policy drafts often live for the legally-reviewed copy.
- **Docusign** (Claude for Legal) — useful when an SDK licence or third-party-asset release is in-flight; surface the issue list before signing.
- **Google Drive** (Cowork native) — default for build artifacts, screenshots, and listing-copy drafts.
- **PDF / file uploads** — Cowork accepts PDF uploads natively; the user can drag a signed rejection notice, a privacy policy, or a regulator guidance PDF in and the skill will cite specific paragraphs in the issue list.
- **Image / screenshot uploads** — for listing-screenshot review, age-rating-vs-icon mismatch checks, or UI-conformance review against the store HIG, the user can drag the screenshot in.
- **Gmail** — generally avoid (privileged correspondence). Acceptable for App Review rejection emails specifically, but prefer dragging the email text rather than granting Gmail access.

### Output

- **Markdown issue list** (always) — saved per Phase 4. The canonical artifact.
- **Microsoft Word tracked-change pass** (optional, Claude for Legal) — if the Microsoft connector is granted and the source is a Word doc (typically: draft store-listing copy, draft T&Cs, draft Acknowledgements / About / Privacy screen text, draft account-deletion flow copy), **additionally** produce a Word file with the issue list surfaced as in-line comments and proposed redlines, saved next to the source as `<original>-store-policy-redline-<YYYY-MM-DD>.docx`. All edits are tracked changes for the store-policy lead / counsel to accept before submission — never silent accepts, never auto-applied.

The take-to-store-policy-review block at the end is **non-negotiable regardless of how grounded the review is**. Reading the actual build artifacts against live policy text makes the issues higher-confidence; it does not turn the assistant into a store-policy lead.

### Routine privacy posture

In a cloud Routine: the companion command `/appdev-ip-advisor:setup-app-store-review-routine` wires this skill into a Cowork Routine watching a release-candidate folder (build artifacts, listing copy, screenshots, privacy manifests). **Privacy tradeoff: the artifact text + image OCR pass through Anthropic's cloud during Routine execution.** If the build is pre-launch / embargoed, run the review as an interactive session instead. The setup command refuses to wire a Routine without explicit non-sensitive-only confirmation.

### Where Claude for Legal does NOT do the plugin's job

Claude for Legal ships practice-area plugins for IP, privacy, employment, commercial, etc., but **does not ship store-specific policy checklists** — Apple's PrivacyInfo.xcprivacy schema, the Google Play Data Safety form's per-data-type matrix, F-Droid's Anti-Features manifest, Snap classic-confinement justification language. `appdev-app-store-rules` ships those checklists; Claude for Legal supplies the citation-verified legal overlays (GDPR / AI Act / COPPA / UK Children's Code) underneath them. The two are complementary, not redundant.

## Sources and rationale

- **Apple** — Apple App Store Review Guidelines (developer.apple.com/app-store/review/guidelines/); Apple Human Interface Guidelines per platform; Privacy Manifest reference (developer.apple.com — `Describing data use in privacy manifests`); Required Reason API reference; App Tracking Transparency reference; TestFlight overview; Sign in with Apple reference; visionOS HIG; CarPlay programming guide; tvOS focus engine guide.
- **Google Play** — Developer Policy Center (play.google.com/about/developer-content-policy/); Play Console help (support.google.com/googleplay/android-developer); Android Vitals documentation (developer.android.com/topic/performance/vitals); Data Safety form documentation; Families Policy + Designed for Families programme; User Choice Billing programme documentation.
- **Microsoft** — Microsoft Store Policies (learn.microsoft.com/en-us/windows/uwp/publish/store-policies); Partner Center documentation.
- **Amazon Appstore** — developer.amazon.com (Appstore Policy Center).
- **Samsung Galaxy Store** — seller.samsungapps.com (Seller Office Help Center).
- **Huawei AppGallery** — developer.huawei.com (AppGallery Connect documentation).
- **F-Droid** — f-droid.org/docs/Inclusion_Policy/; f-droid.org/docs/Anti-Features/; f-droid.org/docs/Reproducible_Builds/.
- **Flathub** — docs.flathub.org/docs/for-app-authors/submission/; flatpak.org documentation for the underlying runtime.
- **Snap Store** — snapcraft.io/docs (especially Snap confinement, snapd interface reference).
- **Steam** — partner.steamgames.com/doc/home; Steam Direct documentation.
- **Epic Games Store** — dev.epicgames.com/docs/epic-games-store/.
- **EU-DMA alt iOS marketplaces** — Apple's DMA developer documentation (developer.apple.com — `Alternative app marketplaces in the EU`, `Notarization for iOS apps`, `Core Technology Fee`).
- **Legal overlays** — EU AI Act (Regulation (EU) 2024/1689), esp. Art. 50 (in force 2026-08-02); GDPR (Regulation (EU) 2016/679), esp. Art. 17 (erasure); COPPA (15 USC §§ 6501–6506; 16 CFR Part 312); GDPR-K (GDPR Art. 8 — parental consent, per-member-state age 13–16); UK Children's Code (Age Appropriate Design Code, ICO); CCPA / CPRA (California, esp. opt-out and minor-data rules); EU Accessibility Act (Directive (EU) 2019/882, in force 2025-06-28 for products).

These are the canonical sources the issue checklists are built from. Anything beyond requires explicit user confirmation that the source applies to the cell, or it doesn't go in the output. **Store-policy section numbers must be live-verified per run.**
