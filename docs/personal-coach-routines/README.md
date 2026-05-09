# Cowork Routine templates

Copy-paste templates for Anthropic Cowork **Routines** — the cloud-hosted automation tier introduced in April 2026. Routines run on Anthropic's cloud infrastructure even when your laptop is closed, and can fire on schedule, on webhook, or on integration events (Linear ticket changes, GitHub PRs opening, etc.).

These templates are **not** auto-installed. They are documented here for the user to read, decide, and create manually in Cowork.

## When to use a Routine vs a Scheduled Task

| Use Routine when… | Use Scheduled Task when… |
|---|---|
| You want it to run with the laptop closed | Your laptop is normally open at the trigger time |
| The trigger is a webhook or external event (Linear / GitHub / monitoring) | The trigger is a clock |
| The output going through Anthropic's cloud is acceptable | The output is sensitive and should stay on the local machine |
| You want a long-running multi-step automation that may take minutes | You want a quick check |

## Privacy posture for personal-coach

This is the **important** part to read before installing any of these templates.

Routines execute in Anthropic's cloud. That means:

- **Inputs to the Routine pass through Anthropic** — including the prompt, any connector data the Routine reads (Drive files, Gmail content, etc.), and any local-file content the Routine has been granted access to.
- **The plugin's hard limit "nothing leaves the machine" no longer fully holds** when a Routine is the execution surface. This is the explicit tradeoff for cloud-tier capability.

**personal-coach skills not safe to wrap in a Routine:**

- `reflection-session` — reflection content is the most sensitive material in the plugin. Run only as an interactive session; never schedule, never routine.
- `personal-profile` — profile edits should be confirmed in real time, not auto-applied by a Routine.

**personal-coach skills generally safe to wrap in a Routine:**

- `news-digest` — search queries already touch the public web; the digest itself is a synthesis of public information.
- `morning-briefing` — modulo the user's comfort with the briefing scaffold transiting cloud (the briefing reads profile / journal / decisions metadata).
- `business-mentoring` — only for the **decision-grading scan** half of the workflow (which Decisions/ files are past deadline). Actual grading is interactive, not routinable.
- `fintech-legal-triage` — only when the contract source is itself already cloud-stored (Drive / DocuSign). If the contract is local-only, the privacy posture is asymmetric — local privacy wasted for cloud convenience.

## Templates

### `news-digest-routine.md`

Runs the daily news digest. Recommended Routine for users who want the digest to land regardless of whether their laptop is open.

### `morning-briefing-routine.md`

Runs the briefing scaffold (with five fields as prompts, not answers). The user fills in fields when they sit down.

### `decision-grading-routine.md`

Weekly scan for decisions past deadline. Surfaces them; does not grade.

### `legal-triage-on-drive-routine.md`

Event-triggered: when a new file lands in a designated Drive folder, run `fintech-legal-triage` against it and write the issue list back to a paired Drive folder. **Read the privacy header in this template carefully before installing — sensitive contracts should not be triaged via Routine.**

## How to install a Routine in Cowork

1. Open Cowork → **Customize** in sidebar → **Routines** (or **Scheduled** for desktop tasks).
2. **+ New routine**.
3. Paste the prompt from the template.
4. Set trigger: schedule (cron-style) or event (Linear / GitHub / webhook URL).
5. Connect connectors the prompt references (Drive, Gmail, etc.).
6. Save.

The first run is the real test. Inspect the Routine's output before relying on the cadence.
