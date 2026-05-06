---
description: Two-minute calm read-only map of the basic-harness marketplace. Explains what basic-harness is, the four personas it's built for, the install flow, and points to the two specialized tours (/personal-coach:tour and /process-skills:tour). Runs the marketplace-tour skill — same flow as when the user says "show me what you do" or equivalent in any language.
---

# Tour

Run the **marketplace-tour** skill (in this same plugin, `welcome`).

Both this slash command and natural-language questions like "show me what you do", "что ты умеешь", "qué haces", "was kannst du", "qu'est-ce que tu fais", "你能做什么" — in any language — are entry points to the same skill. The skill body lives at `skills/marketplace-tour/SKILL.md` and contains the canonical tour: language detection, marketplace one-paragraph framing, persona question, optional install-flow explanation on request, closing pointers to the specialized tours.

This command is a thin wrapper. Do not duplicate the skill's logic here — invoke the skill and let it drive.

## What this command will NOT do

- Will not run a different version of the tour from the skill.
- Will not bypass the skill's hard-gates (no auto-chain into a specialized tour, two disclosure levels max, read-only).
- Will not be the only entry point — natural-language queries trigger the skill directly.
