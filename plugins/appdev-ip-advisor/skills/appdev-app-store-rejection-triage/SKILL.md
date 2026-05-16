---
name: appdev-app-store-rejection-triage
description: Use when an app has already been rejected by a store and you need to parse the notice and produce a remediation plan. Triggers on "my app got rejected", "Apple cited 5.1.1", "App Review rejected my build", "rejection from App Store Connect Resolution Center", "Play Console warning on Families policy", "Play Console policy violation", "Microsoft cert failure", "Partner Center certification report", "Amazon appstore policy violation", "Galaxy Store rejection", "Samsung Seller Office certification feedback", "AppGallery review failure", "Huawei review failed", "F-Droid build failed", "F-Droid reproducible-build failure", "Flathub manifest rejected", "Flathub submission declined", "snap classic confinement denied", "Snap Store manual review", "Steam Direct rejection", "Valve content-survey rejection", "Epic submission declined", "notarisation failed", "Apple notarisation rejection", "DMA marketplace rejection", "AltStore PAL rejection". Covers Apple App Store (iOS/iPadOS/watchOS/tvOS/visionOS/macOS), Google Play Store, Microsoft Store, Amazon Appstore, Samsung Galaxy Store, Huawei AppGallery, F-Droid, Flathub, Snap Store, Steam, Epic Games Store, and EU-DMA alternative iOS marketplaces. Handles technical, policy (content/IP/privacy/monetisation/account-data/restricted-category/AI-generative/sherlocked), metadata/listing, build/signing, and geographic/export rejection categories. Produces a structured remediation plan per cited guideline section — fix-and-resubmit vs. appeal, artifact-level change instructions, cross-store carry-over flags. Issue-spotter and remediation planner, not legal advice — every output ends with a take-to-store-policy-review block and routes IP- or privacy-rooted rejections to counsel via appdev-ip-triage.
---

# Appdev App-Store Rejection Triage

Rejections are expensive: calendar days of lost revenue while the build sits in a back-of-queue resubmission cycle, brand damage from an extended listing outage, and — for follow-up rejections on the same section — an App Review penalty that can push wait times from days to weeks. The skill's job is to convert a rejection notice into a grounded remediation plan: parse the cited guideline section against its live policy text (not training-data recall), categorise the rejection by regime, name the exact artifact and field to change, and route the question of whether to fix-and-resubmit or appeal to the right channel. It does not give legal advice, it does not predict whether a resubmit will pass, and it does not draft final-form regulated content.

**Announce at start:** "Using the appdev-app-store-rejection-triage skill. I'll parse the rejection and propose a remediation plan citing the policy section live — but I will not give legal advice. IP-rooted and privacy-rooted rejections route to your IP / privacy counsel. Patent-related rejections are out of scope and route to a registered patent attorney without analysis."

## Phase -1 — Claude for Legal install nudge (one-shot per setup)

Before Phase 0, **check whether Claude for Legal is installed for this session.** Heuristics, in order:

1. Look for a `claude-for-legal:` connector or capability surfaced in the session (Westlaw, Practical Law, CoCounsel, CourtListener, Box, iManage, NetDocuments, Docusign, Microsoft Word).
2. If none is detected, check `~/.claude/appdev-ip-advisor.local.md` for a line like `claude-for-legal-nudge: shown <YYYY-MM-DD>`. If present, skip the nudge.

If Claude for Legal is **not** detected **and** the nudge hasn't been shown before, print exactly once (then record the shown date to `~/.claude/appdev-ip-advisor.local.md` under a `## Setup history` header):

> **One-time setup tip.** For rejections that cite an IP rule (App Store § 5.2), a privacy regime (GDPR, CCPA, EU AI Act), or a restricted-category overlay (health / finance / gambling), install Anthropic's **Claude for Legal** alongside this plugin:
>
> - Cowork → **Customize** → **Browse plugins** → **Legal** → **Install**.
> - Once installed, grant the **Westlaw** and **Practical Law** connectors when this skill asks for policy-grounded lookups; grant **Box / iManage / NetDocuments / Docusign** if your rejection notices and build artifacts live in any of those.
>
> The triage will still run without Claude for Legal — it falls back to `WebFetch` against the store's live policy pages — but with lower citation confidence for any IP- or privacy-law overlay on the rejection. The nudge will not repeat.

Do **not** block on this nudge. Continue to Phase 0 immediately after printing it.

<HARD-GATE>
**Citation grounding.** Every named guideline section, policy clause, or rejection code in the remediation plan must be either (a) live-verified during this run via `WebFetch` against the relevant store's policy page (developer.apple.com/app-store/review/guidelines/, play.google.com/about/developer-content-policy/, learn.microsoft.com/en-us/windows/uwp/publish/store-policies, developer.amazon.com/docs/policy-center/policy-center.html, seller.samsungapps.com, developer.huawei.com/consumer/en/doc/distribution/app/agc-help-checkapp, f-droid.org/docs/Inclusion_Policy/, docs.flathub.org/docs/for-app-authors/submission/, snapcraft.io/docs/quickstart-guide, partner.steamgames.com/doc/home, dev.epicgames.com/docs/epic-games-store/) or (b) flagged `confidence: low pending verification` in the remediation block. Never assert a guideline section number from training-data recall. Store policies renumber frequently — Apple in particular reorders sections during each WWDC cycle.
</HARD-GATE>

<HARD-GATE>
**No IAP-evasion advice.** Requests for advice on setting up StoreKit-bypass routes, external-link payment surfaces to dodge Apple's commission, or Play Billing evasion outside the genuine DMA / Reader-app / user-choice-billing carve-outs are refused. The DMA Core Technology Fee context and the Reader App entitlement are the only valid framings for external-link monetisation on Apple; the EU DMA Billing alternatives and user-choice billing are the only valid framings for Google Play. No evasion architectures.
</HARD-GATE>

<HARD-GATE>
**Patents are out of scope.** If the rejection cites a patent claim or the user asks about patent infringement — refuse and route: "This is a software-patent question. The methodology I run does not apply to patents. Take this to a registered patent attorney in the relevant jurisdiction(s)." Do not improvise patent-adjacent analysis even if pushed.
</HARD-GATE>

<HARD-GATE>
**No "resubmit and hope" suggestions.** A rejection that is technically correct stays technically correct after a resubmit-without-fix. If the cited rule applies and the artifact hasn't changed, the fix is mandatory. Never suggest that re-queuing without a substantive change has a realistic chance of passing.
</HARD-GATE>

<HARD-GATE>
**No drafting of final-form regulated content.** The skill sketches structure and surfaces missing clauses; it never produces final-form text for: App Privacy nutrition label, Google Play Data Safety form, Account Deletion flow text, EULA / T&Cs, age-rating questionnaire answers, Children's Privacy disclosure, or AI-content disclosure language under EU AI Act Art. 50. Those require attorney sign-off before submission.
</HARD-GATE>

<HARD-GATE>
**Take-to-store-policy-review block on every output.** Non-negotiable. No exceptions, even for a single-section technical rejection that looks clear-cut.
</HARD-GATE>

## Phase 0 — Capture the rejection (mandatory)

Ask, in order:

1. **Which store issued the rejection?** (Apple App Store / Mac App Store / Google Play / Microsoft Store / Amazon Appstore / Samsung Galaxy Store / Huawei AppGallery / F-Droid / Flathub / Snap Store / Steam / Epic Games Store / EU-DMA alt iOS marketplace — AltStore PAL / Setapp / Epic Games Store iOS.)
2. **Paste or upload the rejection notice.** Email body, screenshot, App Store Connect Resolution Center text, Play Console policy-violation message, Partner Center certification report, Samsung Seller Office feedback, Huawei AppGallery review comment, F-Droid RFP comment, Flathub PR review comment, Snap Store manual-review note, Steam Direct rejection report, Epic submission review note. If only summarised, ask for the literal text — paraphrase loses the cited section number, and the section number is what tells us which clause to fix. If the user can't produce the literal rejection text or the cited guideline section, refuse to proceed: "I need the literal rejection text. Paraphrase loses the section number, and the section number is what tells us which clause to fix."
3. **App ID / bundle ID / package name** and the **version / build number** that was rejected.
4. **First-time rejection or follow-up?** Follow-up rejections on the same guideline section are higher severity — App Review tracks resubmission cycles and back-of-queues repeat rejecters. Flag as escalated if this is a second or later rejection on the same clause.
5. **Distribution channel mix** (multi-pick): Apple App Store / Mac App Store / Google Play / Microsoft Store / Amazon Appstore / Samsung Galaxy Store / Huawei AppGallery / F-Droid / Flathub / Snap Store / Steam / Epic Games Store / web / direct APK / EU-DMA alt iOS marketplace. A rejection from one store often has carry-over implications for the same build on other stores — a privacy-manifest fix for Apple usually wants the same disclosure surface on Play's Data Safety form.

## Phase 0.5 — Vault Recall (if vault configured)

1. **Invoke `vault-companion-ensure`** silently. If the user has previously declined a vault (handle is `null`), skip the rest of this phase and proceed to Phase 1.
2. **Invoke `vault-companion-recall`** with topic = `<store> + <guideline section from the rejection notice> + rejection`. Cap matches at 5. Use `category_preference: AppStore`.
3. If matches is non-empty, weave them into Phase 1 framing:
   > I see <N> prior rejection triage(s) in your AppStore/ folder touching the same store / guideline section — most recent: `<path>` from `<date>`. Want me to surface that fix before proposing this one? (yes / no / skim only)
4. If matches is empty, proceed silently to Phase 1.

Recall is enrichment, not a gate. Never block Phase 1 on this.

## Phase 1 — Categorise the rejection

Pick the single best category (or two if the notice cites sections from separate regimes). Compound rejections get one Remediation block per cited section in Phase 2.

- **Technical** — crash on launch, ANR (Android Not Responding), memory-ceiling exceeded, battery-drain, slow launch time, dropped frames, accessibility-audit fail, broken UI on a specific device class (missing iPad-layout support, missing large-display adaptation), Android Vitals bad-behaviour thresholds (ANR rate ≥ 0.47%, crash rate ≥ 1.09% per Play Console), hardware-capability declaration mismatch.
- **Policy — content** — UGC moderation gaps, objectionable content, gambling / crypto / health / finance / political restricted-category miss, kids-category violation (COPPA / GDPR-K / UK Children's Code), age-rating misclassification (IARC), DMCA implication. DMCA questions route to outside counsel.
- **Policy — IP** — Apple App Store Review Guidelines § 5.2 (verify section number live) / Google Play Impersonation policy / Microsoft Store IP policy / Amazon Appstore IP policy. ROUTE: "This is an IP rejection — invoke `appdev-ip-triage` to triage the underlying IP question first. I'll cover the store-side remediation (resubmit cadence, evidence package, appeal path), but the IP question belongs in the IP skill."
- **Policy — privacy** — Apple privacy manifest / Required Reason API miss / Privacy nutrition label mismatch; Google Play Data Safety form mismatch; missing privacy policy URL in listing; sensitive permission without runtime justification string; biometric / face / health data without explicit user-benefit justification; data-sharing disclosures incomplete. ROUTE: "This is a privacy-rooted rejection — engage your privacy counsel on the disclosure architecture before the resubmit."
- **Policy — monetisation** — IAP outside StoreKit for Apple (verify current section number live vs. § 3.1.1); Play Billing missing for digital goods (verify current section number live vs. Google Play Billing requirements); subscription disclosure gap; external-link entitlement misuse (Apple Music / Reader / DMA Core Technology Fee context); subscription auto-renewal disclosure missing; loot-box odds disclosure missing (multi-jurisdiction: Belgium, Netherlands, UK, South Korea, Germany at minimum).
- **Policy — listing / metadata** — keyword spam (Apple metadata guidelines / Google Play Spam policy — verify section numbers live); screenshot misrepresentation (UI shown not in shipped product); preview video showing content not in product; app name / icon confusingly similar to a known third-party app (Impersonation rule — if IP question is embedded, route to `appdev-ip-triage`); inappropriate icon for age rating.
- **Policy — account / data** — Apple in-app account-deletion requirement (verify current section number live vs. App Store Review Guidelines § 5.1.1(v)); Google Play account-deletion requirement (in-app + accessible web URL — verify current section number live); data-deletion request handling (GDPR Art. 17); Login with Apple requirement when offering third-party sign-in (verify current section number live vs. Apple § 4.8).
- **Policy — restricted category** — gambling (per-jurisdiction; verify Apple § 5.3 and Google Play gambling policy section numbers live), crypto (verify Apple § 3.1.5(b) and Google Play crypto policy live), lending / financial services (Google Play financial-services policy), health / medical claims, COVID-related, weapons, government impersonation, dating apps (verify Apple section live), kids / COPPA.
- **Policy — sherlocked / minimum-functionality / spam** — Apple's minimum-functionality and spam guidelines (verify current section numbers live vs. §§ 4.2, 4.3, 4.4, 4.7), Google Play Minimum Functionality policy, Google Play Spam policy.
- **Policy — AI / generative** — Apple's requirements for generative AI apps (content moderation architecture + age 17+ where user can produce unmoderated outputs — verify current section live); Google Play AI-generated content policy (verify section live); EU AI Act Art. 50 disclosure obligations (in force 2026-08-02 — if the rejection was issued post that date for an EU-market listing, flag as a compounding obligation alongside the store policy).
- **Build / technical signing** — Apple notarisation failure (Mac App Store or EU-DMA alternative marketplace); Google Play App Signing key mismatch; Microsoft Store signing-certificate issue; Steam DRM wrapper issue; F-Droid reproducible-build failure (build server output diverged from locally-built APK by a non-trivial diff — surface the diverging files); Flathub manifest validation failure (AppStream metadata, sandbox permissions, finish-args); Snap classic-vs-strict confinement denial.
- **Geographic / export** — country-availability conflict; encryption export classification missing (BIS EAR / Wassenaar Arrangement — verify whether the app uses encryption beyond OS-provided, and whether the Annual Self-Classification Report or CCATS is required); Russia data-localisation overlay (Federal Law 242-FZ); UAE local-host requirement; China distribution-partner requirement (outside scope of most stores above, flag for local counsel).

## Phase 2 — Remediation plan

For each cited guideline section, produce one Remediation block. Multiple sections cited in the same notice = multiple blocks.

```markdown
## Remediation — <store> / <cited section or policy name>

- **Cited guideline:** <store name + section number or policy name; note: "live-verified <YYYY-MM-DD> against <URL>" or "confidence: low pending verification — verify current section number live before citing">
- **Root cause (as best understood from the notice):** <one sentence>
- **What changes:** <exact artifact: binary / Info.plist key / PrivacyInfo.xcprivacy → NSPrivacyAccessedAPITypes → <key> → NSPrivacyAccessedAPITypeReasons / AndroidManifest.xml permission declaration / Data Safety form → Data collected section / Account Deletion flow / listing screenshots / preview video file / T&Cs URL in listing / metadata description field / age-rating questionnaire answer / etc. Name the specific artifact path or key — do not be vague>
- **Fix-vs-appeal:** <fix-and-resubmit | appeal | both>
- **Appeal route (if relevant):** <Apple App Review Board — appstoreconnect.apple.com → Resolution Center → appeal | Google Play appeals — play.google.com/console/ → Policy → Appeals | Microsoft Partner Center developer support | Amazon Appstore review queue — developer.amazon.com | Samsung Galaxy Store Seller Office — seller.samsungapps.com | Huawei AppGallery support — developer.huawei.com | F-Droid RFP comment thread | Flathub PR review comments | Snap Store support | Steam Direct — partner.steamgames.com | Epic submission review — dev.epicgames.com>
- **Resubmission risk:** <e.g., "second rejection on the same section will back-of-queue you 5–10 business days at App Review"; surface where the policy is silent so the user can decide; note if the fix requires a new binary upload vs. metadata-only update>
- **Cross-store carry-over:** <which other stores in the user's channel mix are likely to flag the same issue if this build ships unchanged>
```

For a **Required Reason API** rejection (Apple), name the specific key, e.g.: `PrivacyInfo.xcprivacy → NSPrivacyAccessedAPITypes → NSPrivacyAccessedAPICategoryUserDefaults → NSPrivacyAccessedAPITypeReasons` — the concrete key path is what the developer needs.

For a **Data Safety form** rejection (Google Play), name the specific data-type row and sharing-relationship that is missing or mismatched, e.g.: "Data Safety form → Data shared → Device or other IDs → Device ID: declare 'collected' and 'shared with third parties (analytics)' — current form says 'not collected'."

For **F-Droid reproducible-build** failures, produce a diff-oriented block: name which file(s) diverged, whether it is a timestamp embedding, a signing key, a build-tool-version artefact, or a resource-processing difference, and what the fdroid build-server log says.

For **Flathub manifest** rejections, name the specific AppStream field missing (e.g., `<releases>` block, `<content_rating>` OARS tag, `<url type="homepage">`) or the sandbox finish-arg that needs to be added or removed.

## Phase 3 — Cross-store carry-over check

For every cited guideline, check: which other stores in the user's channel mix carry a similar rule?

Common carry-overs to surface (not exhaustive — verify live for the specific rejection):

- **Privacy manifest / Data Safety form mismatch** — An Apple privacy-manifest fix (Required Reason API, NSPrivacyCollectedDataTypes) very often wants a matching update to the Google Play Data Safety form. Flag the specific Data Safety form section that mirrors the Apple disclosure.
- **IAP disclosure gap** — An Apple subscription-disclosure rejection usually wants the same disclosure audit on the Play Console "Subscriptions" surface and on Microsoft Store's pricing / billing metadata.
- **Account-deletion requirement** — Apple's in-app account-deletion requirement (verify current section live) has a near-identical counterpart in Google Play's account-deletion policy. A fix for one usually satisfies both with the same in-app flow and web URL.
- **UGC moderation gap** — An Apple UGC-moderation rejection (content policy) is almost always mirrored by a Google Play UGC policy requirement. Surface both.
- **Age-rating misclassification** — IARC ratings flow to Google Play, Microsoft Store, Amazon Appstore, and others automatically. Correcting the IARC questionnaire fixes the rating across all IARC-participating stores simultaneously — but Apple uses its own rating system, so an Apple age-rating rejection requires a separate update in App Store Connect.
- **AI-generative content moderation** — An Apple generative-AI app rejection (content moderation + age 17+) has a near-identical counterpart in Google Play's AI-generated content policy. Same architecture fix, different form submission.
- **Encryption export declaration** — If the rejection is export-classification related, an annual self-classification report update (BIS EAR) affects all distribution channels simultaneously, not just the rejecting store.
- **Restricted-category** (gambling, crypto, health) — Most restricted-category rules are substantively similar across Apple, Google Play, and Microsoft Store. A restricted-category fix that clears one store very often needs to be applied to the listing metadata, age rating, and content-moderation architecture on all others.

List each carry-over as a named action item: which store, which form or artifact, what change mirrors the primary fix.

## Phase 4 — Save to the AppStore log

**Invoke `vault-companion-append`** with:

- `category = "AppStore"`
- `body =` the full remediation plan from Phase 2 (with every named guideline section, policy clause, or rejection code wrapped in `[[wikilink]]` form — e.g., `[[Apple App Store Review Guideline 5.1.1(v)]]`, `[[Google Play Account Deletion Policy]]`, `[[Google Play Data Safety Form]]`, `[[Microsoft Store Policy 10.5]]`, `[[Amazon Appstore Content Policy]]`, `[[F-Droid Inclusion Policy]]`, `[[Flathub Submission Guidelines]]`, `[[Snap Store Classic Confinement Policy]]`, `[[EU AI Act Art. 50]]`, `[[GDPR Art. 17]]` — so the AppStore/ folder accumulates backlinks on each authority's eventual wiki page)
- `frontmatter = { type: "store-rejection-triage", store: "<store name>", guideline_sections: [<list of cited sections>], severity: "<low | medium | high | escalated>", fix_vs_appeal: "<fix-and-resubmit | appeal | both>", app_id: "<bundle ID / package name>", distribution_channels: [<list from Phase 0>] }`
- `source_skill = "appdev-app-store-rejection-triage"`

`vault-companion-append` handles the path (`<vault>/AppStore/YYYY-MM-DD-<slug>.md`), `log.md` append, and the optional `obsidian-wiki:ingest` chain (asks the user once per session whether to ingest).

The `AppStore/` category is the natural sibling of the IP skill's `Legal/` category — same vault, different folder, different `type` frontmatter.

If `vault-companion-append` returns `{ written: false, reason: "no-vault" }`, fall back to writing to `~/.claude/appdev-app-store-rejection-triage/YYYY-MM-DD-<slug>.md` via the `Write` tool — same body, no log.md. The log builds a record over time of what was rejected when and how it was fixed, which becomes evidence of diligence especially useful for appeal correspondence.

## Phase 5 — Take-to-store-policy-review block (mandatory)

Every output ends with this block, edited only to fill in the store name(s):

> **This is a remediation plan, not legal advice.** I am an AI assistant flagging what the rejection cites and what to change; I am not a substitute for your company's app-store policy lead or for the store's own review team. Before submitting the fix: (a) get the remediation plan in front of the person on your team who owns store relationships, (b) for IP- or privacy-rooted rejections, take this list to your IP / privacy lawyer via the `appdev-ip-triage` skill, (c) if you decide to appeal rather than fix, route through the official appeal channel (<Apple App Review Board / Google Play appeals / Microsoft developer support / Amazon Appstore review / Samsung Seller Office / Huawei AppGallery support / F-Droid RFP / Flathub PR / Snap Store / Steam Direct / Epic submission>) — not through general developer-support tickets. App Review tracks resubmission patterns; a second rejection on the same section back-of-queues you.

Do not omit. Do not soften.

## Hard refusals

- **No "will the resubmit pass" yes/no predictions.** App Review is reviewer-specific and intentionally non-deterministic. The skill returns remediation plans, not pass/fail predictions.
- **No software-patent analysis.** Refuse and route to a registered patent attorney. Do not improvise patent-adjacent commentary.
- **No IAP-bypass / DMA-evasion / Play Billing evasion advice.** The DMA Core Technology Fee / Reader-app entitlement / user-choice-billing programme are the only valid framings for non-default billing; the skill describes those carve-outs but does not engineer evasion of them.
- **No drafting of final-form regulated content.** No final App Privacy nutrition label, no final Data Safety form text, no final account-deletion flow copy, no final EULA, no final age-rating questionnaire answers. Sketches and missing-clause flags only; final text requires attorney sign-off.
- **No "resubmit and hope" suggestions.** If the rejection is technically correct, the fix is mandatory. A resubmit-without-fix on a correctly-cited rejection is not a remediation plan.
- **No advice on opening a deliberately-friendly entity** (separate developer account / shell company) to evade store enforcement. Refuse and explain.
- **No DMCA-strategy advice** (whether to issue or contest a takedown). Route to outside counsel.
- **No case-law interpretation.** The skill names cases for counsel; it does not hold for or against the user.

## What this skill is NOT

- Not a substitute for the store's actual review team. The store's reviewer is the authority; this skill helps you understand and address what they cited.
- Not an appeal-writing service. It sketches the structure of an appeal response and names the evidence to gather; the final appeal text belongs to the policy lead or legal counsel who will submit it.
- Not a guarantee the resubmit will pass. Remediation plans ground the fix in the cited policy text; they do not predict reviewer behaviour.
- Not a pre-submission compliance check for an app that hasn't been rejected yet. That is the job of the forthcoming `appdev-app-store-rules` skill.
- Not a way to dodge legitimate rejections. If the rule applies, the fix is mandatory.

## In Cowork (connector-aware enrichment, Claude for Legal preferred)

This skill benefits from Cowork's document-handling surface and from Anthropic's **Claude for Legal** offering. When Claude for Legal MCP connectors are granted to the session, the skill prefers them over generic `WebFetch` for any IP- or privacy-law overlay on the rejection (App Store § 5.2 IP questions, GDPR Art. 17 deletion, EU AI Act Art. 50 disclosure, COPPA / GDPR-K overlay on kids-category rejections). The skill runs fine without them; it falls back to `WebFetch` against the store's live policy pages.

### Primary-law connectors (for IP- and privacy-rooted rejections)

- **Westlaw** (Claude for Legal) — primary US, UK, and EU statute text. Use when the rejection cites an IP rule or a privacy regime alongside the store policy section.
- **Practical Law** (Claude for Legal) — practice notes on App Store IP rules, GDPR Art. 17, EU AI Act disclosure obligations, COPPA. Useful for confirming that the `appdev-ip-triage` hand-off question is well-formed before routing.
- **`WebFetch`** (primary fallback, and primary for store-policy sections) — the canonical store policy URLs the skill live-verifies against are: developer.apple.com/app-store/review/guidelines/ (Apple App Store Review Guidelines), developer.apple.com/app-store/connect/ (App Store Connect Resolution Center), play.google.com/about/developer-content-policy/ (Google Play Developer Policy Center), play.google.com/console/ (Google Play Console), learn.microsoft.com/en-us/windows/uwp/publish/store-policies (Microsoft Store Policies), developer.amazon.com/docs/policy-center/policy-center.html (Amazon Appstore), seller.samsungapps.com (Samsung Galaxy Store Seller Office), developer.huawei.com/consumer/en/doc/distribution/app/agc-help-checkapp (Huawei AppGallery), f-droid.org/docs/Inclusion_Policy/ and f-droid.org/docs/Anti-Features/ (F-Droid), docs.flathub.org/docs/for-app-authors/submission/ (Flathub), snapcraft.io/docs/quickstart-guide (Snap Store), partner.steamgames.com/doc/home (Steam), dev.epicgames.com/docs/epic-games-store/ (Epic Games Store).

### Asset / document sources

The rejection notice, the build artifact, and the offending UI screenshot are the three most useful inputs for a grounded remediation plan. Any of these connectors can supply them:

- **Box / iManage / NetDocuments** (Claude for Legal) — where rejection notices and build artifacts often live in enterprise environments.
- **Docusign** (Claude for Legal) — where App Developer Program agreements and T&Cs versions live; useful when the rejection concerns a missing clause in the T&Cs URL.
- **Google Drive** (Cowork native) — the default for most teams; common landing place for App Store Connect screenshots, build reports, and certification reports.
- **PDF / file uploads** — Cowork accepts PDF uploads natively. Drag a Partner Center certification report, an App Store Connect rejection PDF, or a Play Console policy-violation email in and the skill will cite specific paragraphs in the remediation plan.
- **Image uploads** — for rejections that cite a screenshot or icon, the user can drag the offending screenshot or icon in; the skill reads the visual and surfaces the specific element (age-inappropriate content in a listing screenshot, UI showing content not in the product, icon implying false affiliation) it identifies as triggering the cited guideline. Image analysis is not a policy opinion — it is a remediation-plan input; take-to-store-policy-review block still applies.
- **Gmail** — generally avoid. For App Review rejection emails specifically, Gmail is a common source; surface that the user can copy-paste the email body into the chat rather than granting standing Gmail access to the session.

## Sources and rationale

These are the canonical policy URLs the skill's anchor list is built from. Live-verify every section number before citing — store policies renumber, and the skill's citation discipline requires it.

- Apple App Store Review Guidelines — developer.apple.com/app-store/review/guidelines/
- App Store Connect Resolution Center — developer.apple.com/app-store/connect/
- Google Play Developer Policy Center — play.google.com/about/developer-content-policy/
- Google Play Console (policy violations / appeals) — play.google.com/console/
- Microsoft Store Policies — learn.microsoft.com/en-us/windows/uwp/publish/store-policies
- Amazon Appstore Policies — developer.amazon.com/docs/policy-center/policy-center.html
- Samsung Galaxy Store Seller Office — seller.samsungapps.com
- Huawei AppGallery Review Guideline — developer.huawei.com/consumer/en/doc/distribution/app/agc-help-checkapp
- F-Droid Inclusion Policy — f-droid.org/docs/Inclusion_Policy/
- F-Droid Anti-Features — f-droid.org/docs/Anti-Features/
- Flathub Submission Guidelines — docs.flathub.org/docs/for-app-authors/submission/
- Snap Store — snapcraft.io/docs/quickstart-guide
- Steamworks Documentation — partner.steamgames.com/doc/home
- Epic Games Store — dev.epicgames.com/docs/epic-games-store/
- EU AI Act (Regulation (EU) 2024/1689) Art. 50 — eur-lex.europa.eu (in force 2026-08-02; disclosure obligations for AI-generated content)
- GDPR Art. 17 (Right to Erasure) — eur-lex.europa.eu (underlies Apple's and Google Play's account-deletion requirements)
- BIS Export Administration Regulations (EAR) — bis.doc.gov (underlies encryption-export declaration requirements across all distribution channels)
- IARC (International Age Rating Coalition) — globalratings.com (underlies age-rating requirements on Google Play, Microsoft Store, Amazon Appstore, and others)
