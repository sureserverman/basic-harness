# Routine: Fintech Legal Triage on document-source folder change

**Trigger:** event, when a new file is created in a designated folder.
**Source options:** Google Drive (default), or any of the Claude for Legal document connectors — **Box**, **iManage**, **NetDocuments**, **Docusign** (envelope received). Pick the one that matches where your inbound contracts actually live.
**Connectors required (source):** the chosen source connector — read on the trigger folder, write on a paired output folder. For Docusign as source, the output destination must be a writable connector (Drive / Box / iManage / NetDocuments).
**Connectors recommended (primary law):** **Westlaw** and/or **Practical Law** via Claude for Legal — read-only. With these granted, the analyst live-verifies every named regulation before it lands in the issue list. Without them, the analyst falls back to `WebFetch` against the regulator's official site, and issues that can't be verified carry `confidence: low pending verification`.
**Connectors optional (output):** **Microsoft** (Word) — if granted, the Routine additionally produces a `.docx` with the issue list as in-line comments and proposed redlines, surfaced as tracked changes for attorney review (never silent-accepted).
**Privacy:** ⚠ **Read this section before installing.**

> Sensitive contracts, NDAs with named parties, M&A documents, regulatory correspondence — none of these should be triaged via Routine. The contract text passes through Anthropic's cloud during execution, gets read by an LLM, and produces output that may be cached. For any document where confidentiality is load-bearing, do **not** install this Routine. Run `fintech-legal-triage` interactively in Cowork (with Drive connector) instead — same triage quality, no event-trigger overhead, and you get to decide doc-by-doc whether the privacy posture is acceptable.

This Routine is appropriate when:

- The Drive folder receives **vendor T&Cs**, **public-facing partnership templates**, or **routine procurement contracts** that are not confidentiality-sensitive.
- The user wants a fast first-pass issue list before any human review.

This Routine is NOT appropriate when:

- The Drive folder receives client contracts under NDA.
- The folder is shared with parties whose terms prohibit AI processing of contract text.
- The user works with regulated data (e.g., contracts that themselves disclose third-party PII or commercially-sensitive financial terms).

## Prompt

```
A new file has landed in the watched <source> folder. Run fintech-legal-triage
against it.

Primary-law verification preference:
- If Claude for Legal connectors are granted (Westlaw / Practical Law),
  verify every named regulation against them before adding it to the issue
  list. Use WebFetch against the official regulator site as fallback.
  Citation-grounding is non-negotiable per the analyst's hard rules.

1. Read the new file from the watched source (<Drive | Box | iManage |
   NetDocuments | Docusign envelope>). If it's not a contract, T&Cs, or
   regulator-facing document, skip and exit cleanly with a one-line
   note in the output folder.
2. Run fintech-legal-triage Phase 0 (jurisdiction). The jurisdictions
   should already be in the personal profile's Business context section
   (or in the setup-recorded answers) — read them from there. If still
   ambiguous, mark the issue list as
   `confidence: low pending jurisdiction confirmation` and proceed.
3. Walk the issue checklist for the matched cell. Live-verify each named
   regulation per the citation-grounding gate.
4. Produce the structured issue list per the skill's Phase 3 format, with
   each issue citing the source it was verified against (Westlaw,
   Practical Law, or the regulator URL via WebFetch).
5. Append the take-to-counsel block — non-negotiable, no exceptions.
6. Write the issue list to the paired output folder as
   "<original-filename>-triage-<YYYY-MM-DD>.md".
7. [If the Microsoft connector is granted and the source is .docx]
   Additionally produce "<original-filename>-redline-<YYYY-MM-DD>.docx"
   next to the source, with the issue list as in-line comments and
   proposed redlines as tracked changes (NOT auto-accepted).
8. Do NOT modify the original file.
9. Stop.
```

## Folder configuration

The user designates two folders, in whichever source they picked:

- **Trigger folder / source:** the watched folder where new contracts land. Example: `/Legal/Inbound/` (Drive / Box / iManage / NetDocuments) or the Docusign envelope queue for the chosen account.
- **Output folder:** where the triage output lands. Example: `/Legal/Triage/`. May live in the same source system or in Drive — whichever the user prefers. (For Docusign as source, the output must be a writable system, since Docusign isn't a writable destination.)

Configure the Routine's event trigger to fire on **file created** (or **envelope received** for Docusign) in the trigger folder. Set connector permissions to **read** on the trigger folder and **write** on the output folder.

If using Claude for Legal primary-law connectors (Westlaw / Practical Law), grant them at the **session** level — read-only — so the Routine can call them during analysis. If using Microsoft Word output, grant the Microsoft connector with write access to the source-system folder where redlines should land.

Always grant connector access at the **named-folder scope**, not the whole account.

## Cadence

Event-driven; no schedule.

## What this Routine will NOT do

- Will not modify the original file.
- Will not produce a verdict ("this contract is fine") — only an issue list.
- Will not omit the take-to-counsel block, even on low-issue triages.
- Will not run against files outside the designated folder.
- Will not store contract text outside the issue-list output file.

## Verification

The first triggered run is the test:

- Drop a sample non-sensitive contract into the trigger folder.
- Wait for the Routine to fire (typically <2 minutes after upload).
- Check the output folder for `<filename>-triage-<date>.md`.
- Verify the issue list has the take-to-counsel block.

If the Routine fires but produces no issues, that's a flag — most contracts have at least one issue worth raising with counsel. Inspect the prompt; the Routine may have skipped a category.

## Hard rules

- **No verdicts.** Issue lists, always.
- **Take-to-counsel block on every output.** No exceptions.
- **Original file is read-only from the Routine.** No edits, no annotations, no rename. (The Word redline output, when enabled, is a separate `.docx` next to the source — never an edit to the source itself, and all redlines are tracked changes, never auto-accepted.)
- **Citation grounding is non-negotiable.** Every named regulation in the issue list either came from a live primary-law source this run (Westlaw / Practical Law / regulator URL) or is flagged `confidence: low pending verification`. No training-data recall citations.
- **Privacy posture is the user's responsibility** — the prompt cannot enforce that the user is using this only for non-sensitive folders. Read the privacy header above before installing.
