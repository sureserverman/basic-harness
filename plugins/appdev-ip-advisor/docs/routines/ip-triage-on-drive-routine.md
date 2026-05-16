# Routine: Appdev IP Triage on document-source folder change

**Trigger:** event, when a new file is created in a designated folder.
**Source options:** Google Drive (default), or any of the Claude for Legal document connectors — **Box**, **iManage**, **NetDocuments**, **Docusign** (envelope received). Pick the one that matches where your inbound asset bundles / EULAs / licence files / drafts actually live.
**Connectors required (source):** the chosen source connector — read on the trigger folder, write on a paired output folder. For Docusign as source, the output destination must be a writable connector (Drive / Box / iManage / NetDocuments).
**Connectors recommended (primary law):** **Westlaw** and/or **Practical Law** via Claude for Legal — read-only. With these granted, the analyst live-verifies every named statute / app-store-guideline section / OSS-licence clause before it lands in the issue list. Without them, the analyst falls back to `WebFetch` against USPTO / EUIPO / WIPO / UKIPO / Rospatent / IPOS / MOEM (trademarks), developer.apple.com / play.google.com (app-store policy), opensource.org / gnu.org (OSS licences), and any issue that can't be verified carries `confidence: low pending verification`.
**Connectors optional (output):** **Microsoft** (Word) — if granted, the Routine additionally produces a `.docx` with the issue list as in-line comments and proposed redlines for `.docx` inputs (typically draft marketing copy / T&Cs / Acknowledgements). Redlines are surfaced as tracked changes for attorney review (never silent-accepted).
**Privacy:** ⚠ **Read this section before installing.**

> Pre-launch UI mocks, unannounced app names, M&A asset-portfolio diligence under NDA, anything trade-secret — none of these should be triaged via Routine. The asset text and any image OCR for screenshots pass through Anthropic's cloud during execution, get read by an LLM, and produce output that may be cached. For any asset where confidentiality is load-bearing, do **not** install this Routine. Run `appdev-ip-triage` interactively in Cowork (with Drive connector) instead — same triage quality, no event-trigger overhead, and you get to decide doc-by-doc whether the privacy posture is acceptable.

This Routine is appropriate when:

- The folder receives **third-party asset bundles you're evaluating** (icon packs, font EULAs, music libraries, sound-effect packs), **OSS-licence ledger exports** from your dependency tree, **draft marketing copy** that hasn't shipped, **draft App Store / Play Store listing artwork**, **draft T&Cs / Acknowledgements / About-screen text**, **public-competitor analysis** material.
- The user wants a fast first-pass IP issue list before any human review.

This Routine is NOT appropriate when:

- The folder receives pre-launch app names under embargo.
- The folder receives unannounced UI mocks or feature designs.
- The folder is shared with parties whose terms prohibit AI processing of asset text.
- The folder contains M&A diligence material under NDA.

## Prompt

```
A new file has landed in the watched <source> folder. Run appdev-ip-triage
against it.

Primary-law verification preference:
- If Claude for Legal connectors are granted (Westlaw / Practical Law),
  verify every named statute / app-store guideline section / OSS-licence
  clause against them before adding it to the issue list. Use WebFetch
  against the official source (USPTO TESS / EUIPO eSearch / WIPO Madrid
  Monitor / UKIPO / Rospatent / IPOS / MOEM for trademarks;
  developer.apple.com and play.google.com for app-store policy;
  opensource.org and gnu.org for OSS licences) as fallback.
  Citation-grounding is non-negotiable per the analyst's hard rules.

1. Read the new file from the watched source (<Drive | Box | iManage |
   NetDocuments | Docusign envelope>). If the file is not a recognisable
   IP-triage input (not an asset bundle / EULA / licence file / OSS-licence
   ledger / marketing-copy or T&Cs draft / UI screenshot / App Store
   listing draft), skip and exit cleanly with a one-line note in the
   output folder. Patent-related uploads — refuse and route to a patent
   attorney without analysis.
2. Run appdev-ip-triage Phase 0 (jurisdiction + asset type + channel).
   The jurisdictions + distribution channels should already be in the
   setup-recorded answers (or the personal profile's App-dev context
   section) — read them from there. Auto-detect the asset type from
   the file (extension + metadata + content sniff). If still ambiguous,
   mark the issue list as
   `confidence: low pending jurisdiction / channel confirmation`
   and proceed.
3. Walk the issue checklist for the matched (jurisdiction × asset-type ×
   channel) cell. Live-verify each named statute / app-store-guideline
   section / OSS-licence clause per the citation-grounding gate.
4. Produce the structured issue list per the skill's Phase 3 format, with
   each issue citing the source it was verified against (Westlaw,
   Practical Law, USPTO / EUIPO / WIPO / UKIPO / Rospatent / IPOS / MOEM
   URL, developer.apple.com / play.google.com URL, opensource.org /
   gnu.org URL, or copyright.gov / EUR-Lex URL).
5. Append the take-to-counsel block — non-negotiable, no exceptions.
6. Write the issue list to the paired output folder as
   "<original-filename>-ip-triage-<YYYY-MM-DD>.md".
7. [If the Microsoft connector is granted and the source is .docx
   containing draft regulated content — marketing copy / T&Cs /
   Acknowledgements] Additionally produce
   "<original-filename>-redline-<YYYY-MM-DD>.docx" next to the source,
   with the issue list as in-line comments and proposed redlines as
   tracked changes (NOT auto-accepted).
8. Do NOT modify the original file.
9. Stop.
```

## Folder configuration

The user designates two folders, in whichever source they picked:

- **Trigger folder / source:** the watched folder where new asset bundles / EULAs / drafts land. Example: `/AppDev/IP-Inbound/` (Drive / Box / iManage / NetDocuments) or the Docusign envelope queue for the chosen account.
- **Output folder:** where the triage output lands. Example: `/AppDev/IP-Triage/`. May live in the same source system or in Drive — whichever the user prefers. (For Docusign as source, the output must be a writable system, since Docusign isn't a writable destination.)

Configure the Routine's event trigger to fire on **file created** (or **envelope received** for Docusign) in the trigger folder. Set connector permissions to **read** on the trigger folder and **write** on the output folder.

If using Claude for Legal primary-law connectors (Westlaw / Practical Law), grant them at the **session** level — read-only — so the Routine can call them during analysis. If using Microsoft Word output, grant the Microsoft connector with write access to the source-system folder where redlines should land.

Always grant connector access at the **named-folder scope**, not the whole account.

## Cadence

Event-driven; no schedule.

## What this Routine will NOT do

- Will not modify the original file.
- Will not produce a verdict ("this name is fine" / "this licence is safe") — only an issue list.
- Will not opine on software patents — refuses and routes without analysis.
- Will not omit the take-to-counsel block, even on low-issue triages.
- Will not run against files outside the designated folder.
- Will not store asset text outside the issue-list output file.

## Verification

The first triggered run is the test:

- Drop a sample non-sensitive asset (e.g., a public icon-pack licence PDF, or a draft marketing-copy `.docx`) into the trigger folder.
- Wait for the Routine to fire (typically <2 minutes after upload).
- Check the output folder for `<filename>-ip-triage-<date>.md`.
- Verify the issue list has the take-to-counsel block.

If the Routine fires but produces no issues on an asset that should have surfaced something (e.g., a font EULA with no app-embedding clause; a marketing draft that names a celebrity; a dependency ledger that includes a GPL package), that's a flag — inspect the prompt; the Routine may have skipped a category.

## Hard rules

- **No verdicts.** Issue lists, always.
- **No patent analysis.** Refuse-and-route is preserved at Phase -1.
- **Take-to-counsel block on every output.** No exceptions.
- **Original file is read-only from the Routine.** No edits, no annotations, no rename. (The Word redline output, when enabled, is a separate `.docx` next to the source — never an edit to the source itself, and all redlines are tracked changes, never auto-accepted.)
- **Citation grounding is non-negotiable.** Every named statute / app-store-guideline / OSS-licence clause in the issue list either came from a live primary-law source this run (Westlaw / Practical Law / USPTO-EUIPO-WIPO-UKIPO-Rospatent-IPOS-MOEM URL / developer.apple.com / play.google.com / opensource.org / gnu.org / copyright.gov / EUR-Lex) or is flagged `confidence: low pending verification`. No training-data recall citations.
- **Privacy posture is the user's responsibility** — the prompt cannot enforce that the user is using this only for non-sensitive folders. Read the privacy header above before installing.
