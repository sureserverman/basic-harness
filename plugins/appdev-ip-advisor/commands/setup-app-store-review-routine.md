---
description: One-shot setup for the Cowork Routine that watches a release-candidate folder (Google Drive / Box / iManage / NetDocuments) for new app-store submission artifacts — build metadata, PrivacyInfo.xcprivacy, AndroidManifest.xml, Data Safety form exports, listing copy drafts, screenshot sets, age-rating questionnaire answers, account-deletion flow copy — and runs appdev-app-store-rules pre-submission against each one. Prefers Claude for Legal MCP connectors when granted; falls back to Drive. Walks the user through the privacy posture, the source/output folder pair, target-stores + app-category + features context, optional Word tracked-change output, the Routine prompt, and the Cowork UI. Refuses to proceed without explicit privacy confirmation.
---

# Setup App-Store-Review Routine

Wire the `appdev-app-store-rules` skill (driven by the `appdev-app-store-reviewer` subagent) into Cowork's **Routines** so that when a new release-candidate artifact lands in a designated source — a freshly-exported `PrivacyInfo.xcprivacy`, an updated `AndroidManifest.xml`, a Play Console Data Safety form export (`.json` / `.csv`), a draft listing copy `.docx`, a new screenshot set, a draft age-rating questionnaire, a draft Account Deletion flow copy, an SDK licence pack — the reviewer produces a structured pre-submission policy issue list automatically, **before** the build is submitted to App Review / Play Console / Partner Center / Seller Office / etc.

This is the Cowork-first deployment surface for the app-store-rules skill. The interactive skill still works fine on its own; the Routine is for keeping a steady flow of release-candidate artifacts reviewed without the user remembering to ask before every submission.

This command is **Claude for Legal–aware**: if any of Box / iManage / NetDocuments / Microsoft Word connectors have been granted to the user's Cowork session (typically because Claude for Legal is installed), the command offers them as alternatives to Google Drive and offers a Word tracked-change pass alongside the canonical Markdown output. None of those connectors is required; Drive + Markdown is always the fallback.

This command is **paired with `/appdev-ip-advisor:setup-ip-triage-routine`** but distinct from it. The IP-triage Routine watches incoming third-party asset bundles (icon packs, font EULAs, OSS-licence ledgers); this Routine watches the user's own release-candidate artifacts. The two can run side-by-side on different folders without overlap.

## Step 0 — Privacy gate (mandatory)

Routines execute in Anthropic's cloud. The artifact text and any image OCR pass through the LLM and may be cached. Release-candidate artifacts often contain unannounced features, embargoed launch dates, and competitive-sensitive listing copy. **Do not let the user proceed past this step without confirming the privacy posture.**

Read the privacy header out loud (paraphrasing is fine, removing it is not):

> A Cowork Routine runs in Anthropic's cloud with your laptop closed. The artifact text (and any image OCR on screenshots / preview-video stills) is read by an LLM during review; that means it leaves your desktop. This is acceptable for **release-candidate artifacts whose contents are not pre-launch confidential** — generic privacy-manifest entries, generic Data Safety form exports for a feature you have already announced publicly, draft listing copy for a product that has already shipped a previous version on the same store, SDK-licence ledgers for dependencies that are public knowledge. It is **not appropriate** for pre-launch feature flags in metadata, unannounced product names, embargoed launch screenshots, M&A-sensitive build artifacts, or anything where confidentiality is load-bearing. For those, run the `appdev-app-store-rules` skill interactively in Cowork (with the Drive / Box / iManage / NetDocuments connector granted ad-hoc, artifact by artifact) instead — same review quality, you decide doc-by-doc whether the posture is acceptable.
>
> Is the folder you're about to watch limited to non-sensitive release-candidate artifacts? (yes / no / not sure)

Branches:

- **yes** → continue to Step 0.5.
- **no** or **not sure** → stop. Tell the user: "Then this Routine is the wrong tool. Use the interactive skill (`appdev-app-store-rules` in Cowork with the relevant document connector) per-artifact instead. I won't set up a Routine you'll later regret installing — embargoed-launch screenshots are exactly the wrong thing to push through a cloud watch."
- Don't accept partial yes-es ("yes except during launch week"). The Routine fires automatically; "except during launch week" is not a posture.

## Step 0.5 — Claude for Legal install nudge (one-shot)

Before configuring the Routine, **check whether Claude for Legal is installed for this session** (probe for any Westlaw / Practical Law / CoCounsel / CourtListener / Box / iManage / NetDocuments / Docusign / Microsoft Word connector). If none is detected, print exactly once:

> **Strongly recommended for this Routine.** Several store policies overlay primary law that benefits from citation-grounded verification — Apple's privacy-manifest enforcement tracks GDPR / CCPA; Generative AI App rules track EU AI Act Art. 50; Account Deletion tracks GDPR Art. 17; the Families / Kids policies track COPPA / GDPR-K / UK Children's Code. Anthropic's **Claude for Legal** ships the primary-law connectors (Westlaw, Practical Law) and the document-source connectors (Box, iManage, NetDocuments) the Routine prefers. Install it now in another tab:
>
> - Cowork → **Customize** → **Browse plugins** → **Legal** → **Install**.
>
> The Routine will still wire on Google Drive without Claude for Legal — but the legal overlay on each store-policy issue will fall back to `WebFetch` (lower confidence), and the source list shrinks to Drive only. **Install Claude for Legal first if you can; otherwise continue and the plugin will downgrade gracefully.**
>
> Continue with this Routine setup? (`continue` / `pause to install Claude for Legal first` / `cancel`)

Branches:

- **continue** → proceed to Step 1.
- **pause to install Claude for Legal first** → stop. Tell the user: "Good call. After installing Legal in Cowork → Customize → Browse plugins → Legal, re-run `/appdev-ip-advisor:setup-app-store-review-routine`. The Westlaw / Practical Law / Box / iManage / NetDocuments connector options will appear in Step 1."
- **cancel** → stop, no follow-up.

If a Claude for Legal connector **is** already detected, skip this step silently and continue to Step 1.

## Step 1 — Pick the source system and the folder pair

The Routine needs an inbound source (where new release-candidate artifacts land) and an output destination (where the review output is written, one file per input).

First ask which source the user wants to watch. Detect which connectors are granted to the session and offer the available ones; the canonical list is below in preference order (most-managed → least-managed):

> Which source should the Routine watch?
>
> 1. **iManage** or **NetDocuments** — if your legally-reviewed copy (T&Cs, privacy policy, Account Deletion flow copy, EULAs) lives in a managed legal-document system, and you want the same Routine to cover the legal-overlay drafts as well. (Claude for Legal connector.)
> 2. **Box** — common for cross-team release-candidate libraries (engineering + design + marketing + legal all dropping into the same release folder). (Claude for Legal connector.)
> 3. **Google Drive** — the default. Most common landing place for build artifacts (`PrivacyInfo.xcprivacy`, `AndroidManifest.xml`, Data Safety form exports, screenshot sets, listing copy drafts).
>
> Pick one. (If unsure, Drive is fine. Note: Docusign is intentionally not offered here — release-candidate artifacts don't land in Docusign envelopes. Use the IP-triage Routine for SDK-licence signings.)

Then the folder pair:

1. **Trigger (inbound) folder / source** — where new release-candidate artifacts land. The Routine fires on file-created events here. Example: `/AppDev/Release-Candidates/`.
2. **Output folder** — where the review output is written, one file per input. Example: `/AppDev/Store-Policy-Reviews/`. Can live in the same source system or in Drive — whichever the user prefers.

Ask the user:

> 1. What's the **inbound** folder path in `<chosen source>`? (e.g., `/AppDev/Release-Candidates/`.)
> 2. What's the **output** folder path? (System + path — same system as inbound, or Drive.)
> 3. Will the inbound folder ever receive pre-launch confidential artifacts (embargoed screenshots, unannounced feature metadata) by mistake? If yes, set up a separate inbound for non-sensitive only — don't mix.

Confirm system + both paths back to the user before continuing.

## Step 2 — Confirm target-stores + app-category + features + distribution-channel context

The Routine needs the user's working context baked in, because Phase 0 of `appdev-app-store-rules` (stores + app-category + features-in-scope + distribution-channels) cannot be answered interactively in a cloud Routine.

Check whether the user has a personal profile (via `personal-coach`) with an "App-dev context" section listing target stores, app category, and feature set. If yes, read those and confirm with the user.

If no profile is present (or the section is empty), ask:

1. **Which stores are in scope?** (Multi-pick — Apple App Store / Mac App Store / Google Play / Microsoft Store / Amazon Appstore / Samsung Galaxy Store / Huawei AppGallery / F-Droid / Flathub / Snap Store / Steam / Epic Games Store / EU-DMA alt iOS marketplace. Top 3 by user count or revenue if 'everywhere'.)
2. **What does the app do?** (One sentence. Category sets the restricted-policy overlays: utility / game / social / dating / financial / health / kids / generative-AI / news / government.)
3. **Which features are in scope?** (Multi-pick — UGC, in-app payments, subscriptions, sensitive permissions, background activity, health/biometric data, AI-generated content output, children's audience, location tracking, camera/microphone background, advertising/SDK monetisation, push notifications, external links, third-party authentication, file sharing, hardware-specific surface like Vision Pro / watchOS / CarPlay / tvOS / Wear OS / Android Auto / Galaxy DeX.)
4. **What's the distribution-channel mix?** (Multi-pick — same set as the IP-triage routine. Used by the reviewer to bias cross-store carry-over flags.)
5. **What artifact-type mix will land in the inbound folder?** (Multi-pick — `PrivacyInfo.xcprivacy`; AndroidManifest.xml; Data Safety form exports; listing copy drafts (`.docx` / `.md`); screenshot sets / preview videos; age-rating questionnaire drafts; Account Deletion flow copy; subscription paywall copy; T&Cs / EULAs / privacy-policy drafts; SDK licence ledger exports; build metadata for notarisation / signing.)

Hold the answers — they go into the Routine prompt in Step 4.

If the user can't answer questions 1, 2, and 3, stop. A store-policy review without stores + category + features is fiction. (Question 4 the Routine can detect per file from extension / metadata; question 5 is there to bias the Routine's check selection, not as a hard gate.)

## Step 3 — Offer Word tracked-change output (if Microsoft connector granted)

If the **Microsoft** connector is granted (typically because Claude for Legal is installed), offer the Word tracked-change pass as an addition to the canonical Markdown output:

> Optional: if the source file is a `.docx` (typically: draft listing copy, draft T&Cs, draft Privacy Policy, draft Account Deletion flow copy, draft AI-content disclosure text, draft subscription paywall copy), I can also produce a Word file with the store-policy issue list surfaced as in-line comments and proposed redlines, saved next to the source as `<original>-store-policy-redline-<YYYY-MM-DD>.docx`. All edits are **tracked changes** for your store-policy lead / counsel to accept before submission — never silent accepts, never auto-applied. (For build artifacts, screenshots, JSON / XML / plist files, the Word output is skipped automatically — Markdown only.)
>
> Add the Word output for `.docx` inputs? (yes / no)

If the Microsoft connector is **not** granted, skip this step — Markdown is the only output.

## Step 4 — Present the Routine prompt

Show the user this prompt (pre-filled with their answers from Steps 1–3):

```text
A new file has landed in the watched <source> folder. Run appdev-app-store-rules
against it.

Working context (from setup):
- Target stores: <answer>
- App category: <answer>
- Features in scope: <answer>
- Distribution channels: <answer>
- Expected artifact-type mix: <answer — used to bias check selection>

Citation-grounding preference:
- For store policy itself, WebFetch the live developer-portal page
  (developer.apple.com/app-store/review/guidelines/, play.google.com/about/
  developer-content-policy/, learn.microsoft.com/en-us/windows/uwp/publish/
  store-policies, developer.amazon.com, seller.samsungapps.com,
  developer.huawei.com, f-droid.org/docs/Inclusion_Policy/, docs.flathub.org,
  snapcraft.io/docs, partner.steamgames.com/doc/home, dev.epicgames.com/docs/
  epic-games-store/) and pull the current section text before citing.
- For the legal overlay (GDPR / AI Act Art. 50 / COPPA / GDPR-K / UK Children's
  Code / CCPA / EAA): if Claude for Legal connectors are granted (Westlaw /
  Practical Law), verify against them before adding to the issue list.
- If neither resolves a citation, mark the issue
  confidence: low pending verification.
- Citation-grounding is non-negotiable per the reviewer's hard rules.

1. Read the new file from the watched source. If the file is not a recognisable
   pre-submission review input (not a build artifact / PrivacyInfo.xcprivacy /
   AndroidManifest.xml / Data Safety export / listing copy / screenshot set /
   age-rating questionnaire / Account Deletion flow copy / subscription paywall
   copy / T&Cs / EULA / privacy policy / SDK licence ledger), skip and exit
   cleanly with a one-line note in the output folder.
2. Run appdev-app-store-rules Phase 0 (stores + app-category + features + channel)
   using the values above. Auto-detect the artifact type from the file (extension
   + metadata + content sniff).
3. Match (store x category x features) to the canonical checklist clusters in the
   skill. Walk the relevant subset. Live-verify each named guideline section /
   policy clause per the skill's citation-grounding gate.
4. Produce the structured issue list per the skill's Phase 3 format — one Issue
   block per item, naming store + clause (with the URL it was verified against),
   the specific artifact + key to change, the question for store-policy review,
   the cross-store carry-over, and the confidence value.
5. Route IP-rooted findings to the appdev-ip-triage skill (separate output);
   route privacy / GDPR / AI-Act findings as a flagged section in the same output
   for privacy counsel; route restricted-category findings (gambling / crypto /
   health / finance / kids / dating) as a flagged section for subject-matter
   counsel.
6. Append the take-to-store-policy-review block — non-negotiable, no exceptions,
   no softening.
7. Write the issue list to the output folder as
   "<original-filename>-store-policy-<YYYY-MM-DD>.md".
8. [If Word output enabled] When the input is a `.docx` and contains draft
   regulated content (listing copy / T&Cs / privacy policy / Account Deletion
   copy / AI-content disclosure / subscription paywall copy), additionally
   produce "<original-filename>-store-policy-redline-<YYYY-MM-DD>.docx" next to
   the source, with the issue list as in-line comments and proposed redlines as
   tracked changes (NOT auto-accepted).
9. Do NOT modify the original file. Do NOT email anyone. Do NOT summarize the
   artifact content outside the issue-list output. Do NOT predict whether the
   build will pass App Review — surface the issue list and let the
   store-policy lead make the submission call.
10. Patent-related uploads — refuse and route to a patent attorney without
    analysis. IAP-bypass / DMA-evasion / Play-Billing-evasion uploads — refuse
    and explain.
11. Stop.
```

Ask: "Use this prompt? (yes / edit / no)". On `edit`, accept changes and re-show. On `no`, stop and explain that the prompt is the contract between the user and the Routine — without it, the Routine can drift.

## Step 5 — Walk the user through Cowork's UI

> Cowork → **Routines** → **+ New routine** →
>
> - **Name:** "App-store policy review on `<source>`"
> - **Trigger:** Event → `<chosen source — Drive / Box / iManage / NetDocuments>` → File created in folder → pick `<inbound folder>`.
> - **Connectors:** the source connector (read on inbound, write on output). Also add **Westlaw / Practical Law** (or whichever Claude for Legal primary-law connectors are available) — read-only — so the reviewer can verify the legal overlay live. Optionally the **Microsoft** connector if Word output was enabled in Step 3.
> - Grant **scoped** permissions, not the connector's full access: only the named folders, not the entire account.
> - **Prompt:** paste the block from Step 4.
> - **Save.**
>
> The Routine fires automatically when a new file appears in the inbound folder; output lands in the paired folder as `<filename>-store-policy-<date>.md` (and optionally `<filename>-store-policy-redline-<date>.docx`).

## Step 6 — Save the choice

Append to `~/.claude/appdev-ip-advisor.local.md` (create the file with an `## Setup history` header if it doesn't exist):

```markdown
- <YYYY-MM-DD> setup-app-store-review-routine —
  source: <Drive | Box | iManage | NetDocuments>,
  inbound: <path>, output: <system + path>,
  target stores: <list>, app category: <category>,
  features: <list>, distribution channels: <list>,
  artifact-type mix: <list>,
  primary-law connectors: <Westlaw / Practical Law / WebFetch-only>,
  word-redline-output: <yes | no>,
  privacy: non-sensitive-only (confirmed).
```

## Step 7 — Verification (first triggered run)

> The first triggered run is the test:
>
> - Drop a sample non-sensitive artifact (e.g., a public privacy-policy PDF, or a draft listing copy `.docx` for a feature you've already announced) into the inbound folder.
> - Wait for the Routine to fire (typically under 2 minutes after upload).
> - Open the output folder; check the review file exists.
> - Verify the issue list has the take-to-store-policy-review block at the bottom.
> - Verify each cited guideline section has the live URL it was verified against in this run.
> - If the artifact type is one the Routine should know about (a privacy manifest, a Data Safety export, a draft Account Deletion flow) and the issue list is empty, that's a flag — most release-candidate artifacts have at least one issue worth surfacing pre-submission. Inspect the prompt or re-run interactively.

## Step 8 — Hand off

> Setup recorded. The Routine will run automatically on every new file in `<inbound folder>` until you disable it in Cowork → Routines. The interactive `appdev-app-store-rules` skill remains available for sensitive / embargoed artifacts (run it in Cowork with the relevant document connector granted ad-hoc, artifact by artifact).
>
> If you installed Claude for Legal after this setup, the next Routine run will pick up Westlaw / Practical Law automatically — no need to re-run the command. If you switch source systems, add a new target store (e.g., add F-Droid as a distribution target), or your feature set changes substantially, re-run this command to rewire the working context.
>
> The companion `/appdev-ip-advisor:setup-ip-triage-routine` covers third-party asset bundles (icon packs, font EULAs, OSS-licence ledgers) on a separate folder; the two Routines are independent and can run side-by-side. The `appdev-app-store-rejection-triage` skill remains the reactive sibling — invoke it interactively when an actual rejection notice arrives (the Routine surface only does pre-submission review; rejection triage is a per-incident workflow).

## What this command will NOT do

- Will not create the source folders — the user owns the release-candidate folder layout.
- Will not grant the Routine broader connector scopes than the named folder pair (plus read-only Westlaw / Practical Law if those connectors are granted at the user's session level).
- Will not bundle in confidential-artifact handling. The hard rule is non-sensitive-only.
- Will not omit the take-to-store-policy-review block from the Routine prompt.
- Will not opine on software patents — the prompt's refusal is preserved.
- Will not auto-accept tracked changes in the Word output. Every redline stays as a tracked change for store-policy lead / counsel review.
- Will not configure email notifications. Output is file-only — the user reads it when they look at the output folder.
- Will not predict App Review outcomes in the Routine prompt. The prompt explicitly refuses pass/fail predictions.
- Will not duplicate the IP-triage Routine. The two Routines watch different folders; running both on the same folder is a configuration error.

## Hard rules

- **No setup without the Step 0 privacy confirmation.** "Yes except during launch week" is not a posture.
- **No "test the Routine now" auto-fire.** The first real run is the test.
- **No pass/fail predictions in the prompt.** Issue lists, always.
- **No IAP-bypass / DMA-evasion / Play Billing evasion advice in the prompt.** The carve-outs (Reader-app, Music-streaming, External Link Account, DMA, User Choice Billing) are the only valid framings.
- **No patent analysis in the prompt.** Refuse-and-route is preserved.
- **Original files are read-only** from the Routine. No edits, no rename, no annotations.
- **Privacy posture is the user's responsibility.** The Routine prompt cannot enforce that the user keeps only non-sensitive artifacts in the watched folder. Step 0 is the only line of defence; do not skip it.
