# Routine: Fintech Legal Triage on Drive folder change

**Trigger:** event, when a new file is created in a designated Google Drive folder.
**Connectors required:** Google Drive (read on the trigger folder; write on a paired output folder).
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
A new file has landed in the watched Drive folder. Run fintech-legal-triage
against it.

1. Read the new file from Drive. If it's not a contract, T&Cs, or
   regulator-facing document, skip and exit cleanly with a one-line
   note in the output folder.
2. Run fintech-legal-triage Phase 0 (jurisdiction). The jurisdictions
   should already be in the personal profile's Business context section
   — read them from there. If still ambiguous, mark the issue list as
   `confidence: low pending jurisdiction confirmation` and proceed.
3. Walk the issue checklist for the matched cell.
4. Produce the structured issue list per the skill's Phase 3 format.
5. Append the take-to-counsel block — non-negotiable, no exceptions.
6. Write the issue list to the paired output Drive folder as
   "<original-filename>-triage-<YYYY-MM-DD>.md".
7. Do NOT modify the original file.
8. Stop.
```

## Folder configuration

The user designates two folders in Drive:

- **Trigger folder:** the watched folder where new contracts land. Example: `/Legal/Inbound/`.
- **Output folder:** where the triage output lands. Example: `/Legal/Triage/`.

Configure the Routine's event trigger to fire on **file created** in the trigger folder. Set the connector permissions to **read** on the trigger folder and **write** on the output folder.

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
- **Original file is read-only from the Routine.** No edits, no annotations, no rename.
- **Privacy posture is the user's responsibility** — the prompt cannot enforce that the user is using this only for non-sensitive folders. Read the privacy header above before installing.
