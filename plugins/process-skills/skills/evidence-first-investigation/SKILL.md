---
name: evidence-first-investigation
description: Use when something looks wrong and you're tempted to propose a fix, an explanation, or a conclusion before you understand the cause. Trigger on "why is X happening", "what went wrong with Y", "diagnose this", "explain this discrepancy", "the numbers don't match", "this doesn't add up", "investigate", "audit this", or any analytical request where the user expects a grounded answer rather than a guess. Useful for journalists, analysts, auditors, researchers, and anyone debugging anything (technical or otherwise).
---

# Evidence-First Investigation

The anti-pattern: **propose explanations or fixes before understanding the cause**, then move on when the symptom disappears or a plausible-sounding story sticks.

This skill applies whether you are debugging a system, auditing a discrepancy, investigating a story, reviewing why a project missed its target, or asking why a metric moved.

## The rule

Do NOT propose conclusions, fixes, or explanations until you can answer:

1. **What** is the observed behavior or discrepancy (specific evidence, not "it seems off")
2. **Why** it is happening (root cause supported by evidence, not just a plausible story)
3. **How** the proposed conclusion or fix follows from that root cause

If you cannot answer all three, you need more evidence, not more guesses.

## What "act before understanding" looks like

- "Maybe the source had a typo" — without checking the original document
- "Try restarting it / try this other approach" — without confirming the first one failed for a specific reason
- "The trend is probably driven by X" — without separating X from confounders
- "It must be because of the new policy" — without checking what changed and when
- Proposing 3 possible explanations and asking which one to investigate
- A conclusion "fits" but you can't show what evidence ruled out the alternatives

## What evidence-first investigation looks like

1. **Observe** — gather the actual evidence: source documents, raw data, logs, configs, prior interviews, exact quotes. Do not summarize before you've read.
2. **Hypothesize** — form one specific hypothesis based on the evidence. Not "something with the data," but "the discrepancy comes from Source A reporting in calendar year and Source B in fiscal year."
3. **Test** — run a single check that would confirm or refute the hypothesis. Cross-reference one document against another. Pull one specific data slice. Ask one specific source.
4. **Repeat or conclude** — if refuted, form a new hypothesis from what you now know. If confirmed, write the conclusion grounded in the specific evidence that confirms it.

## When something "resolves itself"

If a discrepancy disappears, a story changes, or a problem stops happening without a deliberate intervention, that is NOT resolved. Something changed — find what. Common culprits:

- A retry / republish / re-run picked up corrected data the second time
- An upstream system reloaded its config
- Someone in the chain quietly fixed it without telling you
- A timezone, calendar, or rounding difference disguised the issue
- You changed something else that had a side effect
- The thing you thought you measured was actually a different thing the second time

"It works now" or "the numbers match now" without understanding why is a future surprise waiting to happen — and in journalism or audit work, it's a retraction waiting to happen.

## The one-fix / one-claim discipline

When acting on a confirmed root cause:

- **One adjustment at a time.** If you change three things at once, you don't know which mattered.
- **One claim per assertion.** If your conclusion bundles three claims, separate them — each gets its own evidence trail.
- **State what was ruled out, not just what was confirmed.** "X is the cause" is weaker than "X is the cause; we ruled out Y because <evidence>, and Z because <evidence>."

## What to record

For any investigation worth more than five minutes, keep a short evidence log:

- Hypothesis under test (one line)
- Check performed (one line)
- Result (what the check actually returned, verbatim where possible)
- Status: confirmed / refuted / inconclusive

A four-column log makes it easy to revisit when a stakeholder, editor, or reviewer asks "how do you know?" — which they will.

## When to stop investigating

Stop when:

- You have evidence that makes one explanation strongly more likely than the alternatives, AND
- The remaining alternatives have been checked or have a documented reason for being ruled out, AND
- Going further would not change the action or conclusion

Stop also when the cost of more investigation exceeds the value of the additional certainty — but say so explicitly ("we are concluding here because the marginal source is unreachable; the conclusion stands on the evidence we have").

## Sources and rationale

- **Hypothesis-driven debugging** — Brian Kernighan and Rob Pike, *The Practice of Programming*, Ch. 5; Andreas Zeller, *Why Programs Fail*
- **Differential diagnosis** — medical-investigative tradition; rule out alternatives, don't just confirm a favorite
- **Five Whys** — Sakichi Toyoda / Toyota Production System; keep asking "why" past the first plausible answer
- **Bayesian reasoning under uncertainty** — Daniel Kahneman, *Thinking, Fast and Slow*; the strongest evidence is what changes your prior, not what confirms it
- **Investigative-journalism standards** — *Verification Handbook* (European Journalism Centre, 2014); confirm independently, attribute precisely
- **Audit evidence sufficiency** — IIA *International Standards for the Professional Practice of Internal Auditing*, Standard 2310 (Identifying Information): "sufficient, reliable, relevant, and useful information"
