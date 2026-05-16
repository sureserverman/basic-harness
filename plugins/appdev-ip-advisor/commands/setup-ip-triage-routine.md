---
description: One-shot setup for the Cowork Routine that watches an asset / spec / licence-bundle source (Google Drive / Box / iManage / NetDocuments / Docusign envelopes) for new app-development inputs and runs appdev-ip-triage on each one. Prefers Claude for Legal MCP connectors when granted; falls back to Drive. Walks the user through the privacy posture, the source/output folder pair, jurisdiction + asset-type + distribution-channel context, optional Word tracked-change output, the Routine prompt, and the Cowork UI. Refuses to proceed without explicit privacy confirmation.
---

# Setup IP-Triage Routine

Wire the `appdev-ip-triage` skill (driven by the `appdev-ip-analyst` subagent) into Cowork's **Routines** so that when a new asset bundle, font EULA, icon-pack licence, OSS-licence ledger export, marketing draft, or T&Cs draft lands in a designated source, the analyst produces a structured IP issue list automatically — before the user ships.

This is the Cowork-first deployment surface for the plugin. The interactive skill still works fine on its own; the Routine is for keeping a steady flow of inbound assets triaged without the user remembering to ask.

This command is **Claude for Legal–aware**: if any of Box / iManage / NetDocuments / Docusign / Microsoft Word connectors have been granted to the user's Cowork session (typically because Claude for Legal is installed), the command offers them as alternatives to Google Drive and offers a Word tracked-change pass alongside the canonical Markdown output. None of those connectors is required; Drive + Markdown is always the fallback.

## Step 0 — Privacy gate (mandatory)

Routines execute in Anthropic's cloud. The asset text and any image OCR pass through the LLM and may be cached. **Do not let the user proceed past this step without confirming the privacy posture.**

Read the privacy header out loud (paraphrasing is fine, removing it is not):

> A Cowork Routine runs in Anthropic's cloud with your laptop closed. The asset text (and any image OCR for UI screenshots / marketing visuals) is read by an LLM during triage; that means it leaves your desktop. This is acceptable for **third-party asset bundles you're evaluating (icon packs, font EULAs, music libraries), draft marketing copy that hasn't shipped yet but isn't trade-secret, draft Acknowledgements / About screens, public competitor analysis**. It is **not appropriate** for pre-launch app names under embargo, unannounced UI mocks, M&A asset-portfolio diligence under NDA, or anything where confidentiality is load-bearing. For those, run the `appdev-ip-triage` skill interactively in Cowork (with the Drive connector granted ad-hoc) instead — same triage quality, you decide doc-by-doc whether the posture is acceptable.
>
> Is the folder you're about to watch limited to non-sensitive assets? (yes / no / not sure)

Branches:

- **yes** → continue to Step 0.5.
- **no** or **not sure** → stop. Tell the user: "Then this Routine is the wrong tool. Use the interactive skill (`appdev-ip-triage` in Cowork with Drive connector) per-asset instead. I won't set up a Routine you'll later regret installing."
- Don't accept partial yes-es ("yes for now"). The Routine fires automatically; "for now" is not a posture.

## Step 0.5 — Claude for Legal install nudge (one-shot)

Before configuring the Routine, **check whether Claude for Legal is installed for this session** (probe for any Westlaw / Practical Law / CoCounsel / CourtListener / Box / iManage / NetDocuments / Docusign / Microsoft Word connector). If none is detected, print exactly once:

> **Strongly recommended for this Routine.** Anthropic's **Claude for Legal** ships the primary-law connectors (Westlaw, Practical Law) and the document-source connectors (Box, iManage, NetDocuments, Docusign) the Routine prefers. Install it now in another tab:
>
> - Cowork → **Customize** → **Browse plugins** → **Legal** → **Install**.
>
> The Routine will still wire on Google Drive without Claude for Legal — but every named statute / app-store-guideline / OSS-licence clause in the issue list will fall back to `WebFetch` (lower confidence), and the source list shrinks to Drive only. **Install Claude for Legal first if you can; otherwise continue and the plugin will downgrade gracefully.**
>
> Continue with this Routine setup? (`continue` / `pause to install Claude for Legal first` / `cancel`)

Branches:

- **continue** → proceed to Step 1.
- **pause to install Claude for Legal first** → stop. Tell the user: "Good call. After installing Legal in Cowork → Customize → Browse plugins → Legal, re-run `/appdev-ip-advisor:setup-ip-triage-routine`. The Westlaw / Practical Law / Box / iManage / NetDocuments / Docusign connector options will appear in Step 1."
- **cancel** → stop, no follow-up.

If a Claude for Legal connector **is** already detected, skip this step silently and continue to Step 1.

## Step 1 — Pick the source system and the folder pair

The Routine needs an inbound source (where new asset bundles / licences / drafts land) and an output destination (where the triage output is written, one file per input).

First ask which source the user wants to watch. Detect which connectors are granted to the session and offer the available ones; the canonical list is below in preference order (most-managed → least-managed):

> Which source should the Routine watch?
>
> 1. **iManage** or **NetDocuments** — if your IP / asset-licence files live in a managed legal-document system. (Claude for Legal connector.)
> 2. **Box** — common for M&A asset-portfolio diligence and large-team asset libraries. (Claude for Legal connector.)
> 3. **Docusign** — fire on new envelope received. Useful for vendor EULAs / talent releases / asset-licence signings still in flight. (Claude for Legal connector.)
> 4. **Google Drive** — the default if none of the above is available. Most common landing place for icon-pack zips, font-EULA PDFs, OSS-licence-ledger exports.
>
> Pick one. (If unsure, Drive is fine.)

Then the folder pair:

1. **Trigger (inbound) folder / source** — where new asset bundles / drafts land. The Routine fires on file-created (or envelope-received, for Docusign) events here. Example: `/AppDev/IP-Inbound/`.
2. **Output folder** — where the triage output is written, one file per input. Example: `/AppDev/IP-Triage/`. The output folder can live in the same source system or in Drive — whichever the user prefers. If the user picked Docusign as source, the output folder must be a writable system (Drive / Box / iManage / NetDocuments) — Docusign envelopes are not a writable destination.

Ask the user:

> 1. What's the **inbound** folder path in `<chosen source>`? (e.g., `/AppDev/IP-Inbound/`.)
> 2. What's the **output** folder path? (System + path — same system as inbound, or Drive.)
> 3. Will the inbound folder ever receive pre-launch confidential UI mocks or unannounced app names by mistake? If yes, set up a separate inbound for non-sensitive only — don't mix.

Confirm system + both paths back to the user before continuing.

## Step 2 — Confirm jurisdiction + asset-type-mix + distribution-channel context

The Routine needs the user's working context baked in, because Phase 0 of `appdev-ip-triage` (jurisdiction + asset type + channel) cannot be answered interactively in a cloud Routine.

Check whether the user has a personal profile (via `personal-coach`) with an "App-dev context" section listing incorporation jurisdiction(s), target distribution markets, and primary distribution channels. If yes, read those and confirm with the user.

If no profile is present (or the section is empty), ask:

1. "Where is the company incorporated?"
2. "Where will the app be distributed?" (Top three by user count or revenue if the answer is 'everywhere'.)
3. "What's the distribution channel mix?" (Apple App Store / Google Play / web / desktop / direct-sideload / cross-platform. Multi-pick.)
4. "What asset-type mix will land in the inbound folder?" (Multi-pick: app-name / brand-mark drafts; UI mocks / screenshots; icon-pack or font EULAs; music or sound-effect libraries; OSS-licence ledger exports; AI-generated content; right-of-publicity / talent releases; marketing drafts; App Store listing drafts; T&Cs / Acknowledgements drafts.)

Hold the answers — they go into the Routine prompt in Step 4.

If the user can't answer 1, 2, and 3, stop. An IP triage without jurisdiction + channel is fiction. (Question 4 the Routine can detect per file from extension / metadata; it's there to bias the Routine's check selection, not as a hard gate.)

## Step 3 — Offer Word tracked-change output (if Microsoft connector granted)

If the **Microsoft** connector is granted (typically because Claude for Legal is installed), offer the Word tracked-change pass as an addition to the canonical Markdown output:

> Optional: if the source file is a `.docx` (typically: draft marketing copy, draft T&Cs, draft in-app Acknowledgements screen text), I can also produce a Word file with the IP issue list surfaced as in-line comments and proposed redlines, saved next to the source as `<original>-redline-<YYYY-MM-DD>.docx`. All edits are **tracked changes** for attorney review before acceptance — never silent accepts, never auto-applied. (For asset zips / PDF EULAs / image bundles, the Word output is skipped automatically — Markdown only.)
>
> Add the Word output for `.docx` inputs? (yes / no)

If the Microsoft connector is **not** granted, skip this step — Markdown is the only output. Don't ask the user to grant the connector mid-setup; the value is real but the privacy-and-permissions decision should be made deliberately, not buried in a routine wizard.

## Step 4 — Present the Routine prompt

Show the user this prompt (pre-filled with their answers from Steps 1–3):

```text
A new file has landed in the watched <source> folder. Run appdev-ip-triage
against it.

Working context (from setup):
- Company incorporated in: <answer>
- Distribution markets: <answer>
- Distribution channels: <Apple App Store | Google Play | web | desktop | direct-sideload | cross-platform — multi-pick>
- Expected asset-type mix: <answer — used to bias check selection>

Primary-law verification preference:
- If Claude for Legal connectors are granted (Westlaw / Practical Law),
  verify every named statute / app-store guideline section / OSS-licence
  clause against them before adding it to the issue list. Use WebFetch
  against the official source (USPTO TESS / EUIPO eSearch / WIPO Madrid
  Monitor / UKIPO / Rospatent / IPOS / MOEM for trademarks;
  developer.apple.com and play.google.com for app-store policy;
  opensource.org and gnu.org for OSS licences) as fallback.
  Citation-grounding is non-negotiable per the analyst's hard rules.

1. Read the new file from the watched source. If the file is not a
   recognisable IP-triage input (not an asset bundle / EULA / licence file /
   OSS-licence ledger / marketing-copy or T&Cs draft / UI screenshot /
   App Store listing draft), skip and exit cleanly with a one-line note
   in the output folder. Patent-related uploads — refuse and route to a
   patent attorney without analysis (Phase -1 hard rule).
2. Run appdev-ip-triage Phase 0 (jurisdiction + asset type + channel)
   using the values above. Auto-detect the asset type from the file
   (extension + metadata + content sniff). If the file's apparent
   distribution target falls outside the working channel set, mark the
   issue list as `confidence: low pending channel confirmation` and proceed.
3. Match (jurisdiction × asset-type × channel) to the canonical issue
   checklist in the skill. Walk it. Live-verify each named statute /
   guideline / licence clause per the skill's citation-grounding gate.
4. Produce the structured issue list per the skill's Phase 3 format —
   one Issue block per item, naming the regime (with the source it was
   verified against), the question for counsel, and the confidence value.
5. Append the take-to-counsel block — non-negotiable, no exceptions,
   no softening.
6. Write the issue list to the output folder as
   "<original-filename>-ip-triage-<YYYY-MM-DD>.md".
7. [If Word output enabled] When the input is a `.docx` and contains
   draft regulated content (marketing copy / T&Cs / Acknowledgements),
   additionally produce "<original-filename>-redline-<YYYY-MM-DD>.docx"
   next to the source, with the issue list as in-line comments and
   proposed redlines as tracked changes (NOT auto-accepted).
8. Do NOT modify the original file. Do NOT email anyone. Do NOT
   summarize the asset content outside the issue-list output.
9. Stop.
```

Ask: "Use this prompt? (yes / edit / no)". On `edit`, accept changes and re-show. On `no`, stop and explain that the prompt is the contract between the user and the Routine — without it, the Routine can drift.

## Step 5 — Walk the user through Cowork's UI

> Cowork → **Routines** → **+ New routine** →
>
> - **Name:** "IP triage on `<source>`"
> - **Trigger:** Event → `<chosen source — Drive / Box / iManage / NetDocuments / Docusign>` → File created (or envelope received, for Docusign) in folder → pick `<inbound folder>`.
> - **Connectors:** the source connector (read on inbound, write on output). Also add **Westlaw / Practical Law** (or whichever Claude for Legal primary-law connectors are available) — read-only — so the analyst can verify regulation citations live. Optionally the **Microsoft** connector if Word output was enabled in Step 3.
> - Grant **scoped** permissions, not the connector's full access: only the named folders, not the entire account.
> - **Prompt:** paste the block from Step 4.
> - **Save.**
>
> The Routine fires automatically when a new file appears in the inbound folder; output lands in the paired folder as `<filename>-ip-triage-<date>.md` (and optionally `<filename>-redline-<date>.docx`).

## Step 6 — Save the choice

Append to `~/.claude/appdev-ip-advisor.local.md` (create the file with an `## Setup history` header if it doesn't exist):

```markdown
- <YYYY-MM-DD> setup-ip-triage-routine —
  source: <Drive | Box | iManage | NetDocuments | Docusign>,
  inbound: <path>, output: <system + path>,
  jurisdictions: <list>, distribution channels: <list>,
  asset-type mix: <list>,
  primary-law connectors: <Westlaw / Practical Law / WebFetch-only>,
  word-redline-output: <yes | no>,
  privacy: non-sensitive-only (confirmed).
```

## Step 7 — Verification (first triggered run)

> The first triggered run is the test:
>
> - Drop a sample non-sensitive asset (e.g., a public icon pack's licence PDF, or a draft marketing-copy `.docx`) into the inbound folder.
> - Wait for the Routine to fire (typically under 2 minutes after upload).
> - Open the output folder; check the triage file exists.
> - Verify the issue list has the take-to-counsel block at the bottom.
> - If the asset type is one the Routine should know about (font EULA, OSS-licence ledger, app-name draft, AI-generated illustration) and the issue list is empty, that's a flag — most assets have at least one IP question worth raising with counsel. Inspect the prompt or re-run interactively.

## Step 8 — Hand off

> Setup recorded. The Routine will run automatically on every new file in `<inbound folder>` until you disable it in Cowork → Routines. The interactive `appdev-ip-triage` skill remains available for sensitive assets (run it in Cowork with the relevant document connector granted ad-hoc, doc by doc).
>
> If you installed Claude for Legal after this setup, the next Routine run will pick up Westlaw / Practical Law automatically — no need to re-run the command. If you switch source systems (e.g., move from Drive to iManage), re-run this command to rewire. If you change distribution channels (e.g., add F-Droid as a sideload target), re-run this command to rewire the channel context.

## What this command will NOT do

- Will not create the source folders — the user owns the asset / document layout.
- Will not grant the Routine broader connector scopes than the named folder pair (plus read-only Westlaw / Practical Law if those connectors are granted at the user's session level).
- Will not bundle in confidential-asset handling. The hard rule is non-sensitive-only.
- Will not omit the take-to-counsel block from the Routine prompt.
- Will not opine on software patents — the prompt's Phase -1 hard rule is preserved.
- Will not auto-accept tracked changes in the Word output. Every redline stays as a tracked change for attorney review.
- Will not configure email notifications. Output is file-only — the user reads it when they look at the output folder.
- Will not block on the Step 0.5 Claude for Legal install nudge. The nudge is a one-shot recommendation; the plugin works without it (Drive + `WebFetch` fallback) and the user can decline and continue.

## Hard rules

- **No setup without the Step 0 privacy confirmation.** "Yes for now" is not a posture.
- **No "test the Routine now" auto-fire.** The first real run is the test.
- **No verdicts in the prompt.** Issue lists, always.
- **No patent analysis in the prompt.** Refuse-and-route is preserved.
- **Original files are read-only** from the Routine. No edits, no rename, no annotations.
- **Privacy posture is the user's responsibility.** The Routine prompt cannot enforce that the user keeps only non-sensitive assets in the watched folder. Step 0 is the only line of defence; do not skip it.
