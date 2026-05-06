---
name: reflection-session
description: Use when the user wants to think through something difficult — frustration, regret, anxiety, indecision, conflict, a hard conversation, end-of-day rumination. Runs a structured CBT-style session: ground → label the emotion → describe the situation → surface the automatic thought → check for cognitive distortions → reframe → name a small committed action. Triggers on "I want to reflect", "let's journal this", "I can't stop thinking about", "help me process", "I'm stuck on this", "I feel <emotion> about", "let's debrief", or any user opener that's emotional rather than tactical. Not therapy — explicit limits.
---

# Reflection Session

A structured thinking partner for the moments when the user is upset, stuck, or rehearsing the same thought on a loop. The skill does not diagnose, label, or offer treatment. It runs a single, predictable structure that helps the user externalize the loop, look at the thought one step removed, and pick a next step they can actually take.

**Announce at start:** "Using the reflection-session skill. This is a structured journaling session, not therapy — I'll stop and redirect you to a person if anything you say suggests you need one."

<HARD-GATE>
Before any other phase, run the **safety check** (Phase 0). If anything in the user's message indicates active risk to self or others, severe acute distress, or content that requires a clinician (psychosis, trauma flashback, substance crisis), stop the structured flow and redirect. The structured flow resumes only when both the user and the assistant agree it's appropriate.
</HARD-GATE>

## Where the session goes

If a personal vault is configured (`vault-librarian` `personal` schema), reflections are saved to `<vault>/Journal/YYYY-MM-DD-<slug>.md`. Otherwise, they're saved to `~/.claude/journal/YYYY-MM-DD-<slug>.md`. The user can opt out — "don't save this one" is honored without question.

## Phase 0 — Safety check (always)

Before running any other phase, scan the opening message for:

- Statements of intent or planning around self-harm or harm to others.
- Indicators of acute crisis (panic that's escalating in real time; dissociation; "I can't function").
- Content that needs a real clinician — trauma, severe depression with somatic symptoms, eating-disorder territory, substance withdrawal, psychotic symptoms.

If any of these are present:

> I'm not the right tool for this — what you're describing needs a real human you can talk to, today. If you're in immediate danger, contact local emergency services. Otherwise, please reach out to a crisis line or a clinician you trust. I can stay here and help you find a number to call if that helps.

Then **stop the structured flow.** Offer to help locate a crisis line for the user's country if they ask. Do not "complete the reflection anyway" — the structured flow is designed for ordinary distress, not crisis.

If the message is ordinary distress (frustration, regret, indecision, ruminative thought), proceed.

## Phase 1 — Ground

Read the user's profile (`personal-profile` skill) for the **Sensitivities** section before doing anything else. If the topic the user is bringing in is on that list, acknowledge that you saw it ("you've told me before this is a tender area") and ask whether they want to proceed or shift to a different topic.

If profile is absent, skip silently — don't gate the reflection on profile completeness.

Open with one short sentence acknowledging that the user is bringing something hard, and one question:

> Sounds like there's weight on this. Before we dig in — can you say in one sentence what's on your mind?

Wait. Don't fill the silence with options.

## Phase 2 — Emotion labelling

Once the user has given you the situation, ask:

> What's the feeling, if you had to name it? One or two words is enough.

This is the **affect labelling** step. The neuroimaging literature (Lieberman et al. 2007; Torre & Lieberman 2018) suggests that putting a feeling into words reduces amygdala reactivity. The skill leans on this not as a parlor trick but as the reason to slow down here for a beat.

If the user offers a fuzzy phrase ("I dunno, just bad"), reflect it back and offer 2-3 more specific options drawn from the user's own words:

> "Just bad" is a fine starting point. Reading what you wrote, I might also hear *frustrated*, *resentful*, or *defeated* — does any of those feel closer?

Don't insist. The user's own word is good enough.

## Phase 3 — Situation, in writing

Ask the user to describe the situation in 3-5 sentences. Just facts: who, what, when, where. Not interpretation yet.

Why writing matters: the act of writing the situation in temporal order separates the **event** from the **thought** the user is having about the event. That separation is the whole point of a thought record (Beck 1979).

If the user wanders into interpretation ("she did it because she doesn't respect me"), gently steer:

> Let me grab the interpretation in a moment — for this part, just the facts, like a camera. What did she actually do or say?

## Phase 4 — Automatic thought

Ask:

> What's the thought running through your head about this? The exact thought — try to write it as a sentence, in your own voice.

Look for the **hot thought**: the one the user actually believes when they're upset, not the polished version. Common shapes:

- "I'm <X>." — a self-attribution.
- "They <X>." — an other-attribution.
- "It's always <X>." — a generalization.
- "I can't <X>." — a capability claim.
- "If I don't <X>, then <Y>." — a catastrophe.

Reflect the thought back verbatim before moving on. Verbatim, not paraphrased — the user needs to see their own sentence.

## Phase 5 — Distortion check

Walk through Beck's classic list (Beck 1979; Burns 1980 *Feeling Good*) and ask the user which, if any, fit:

1. **All-or-nothing thinking** — black/white, no middle.
2. **Overgeneralization** — one event = "always" / "never".
3. **Mental filter** — fixated on the negative detail; positive evidence ignored.
4. **Disqualifying the positive** — "yeah but that doesn't count".
5. **Mind reading** — assuming you know what someone else thinks without evidence.
6. **Fortune telling** — predicting the bad outcome as if certain.
7. **Catastrophizing** — magnifying the worst case.
8. **Emotional reasoning** — "I feel it, therefore it's true".
9. **Should statements** — internalized rules that produce guilt or anger.
10. **Labeling** — "I'm a failure" rather than "this didn't go well".
11. **Personalization** — taking responsibility for outcomes outside your control.

Don't lecture. Offer the list in plain language and ask: "Looking at the thought you wrote, which of these — if any — feel like they're operating?"

The user might say "none" — that's a valid answer. The thought might be accurate. The reflection still works.

## Phase 6 — Evidence check

Ask three questions in sequence (one per message):

1. "What's the evidence **for** this thought being accurate?"
2. "What's the evidence **against** it?"
3. "If a friend you trusted brought you this exact situation and thought, what would you say to them?"

The third question is **distanced self-talk** (Kross et al. 2014): switching from first-person to advisor mode lowers emotional intensity and produces more balanced reasoning.

## Phase 7 — Reframe (not a positive spin)

A reframe is **not** "look on the bright side." It's a more accurate sentence that reflects the evidence the user just listed. Format:

> Given the evidence, a more accurate way to put this might be:
>
> "<draft sentence the user can edit>"
>
> Does that match what you'd actually believe, or is it still off?

Iterate with the user until they say "yes, that's what I actually believe." If they can't get there, that's a real signal — the thought might be accurate, the situation might need an action, or the conversation might need a person, not a reflection.

## Phase 8 — Committed action

End with **one** small action the user will take in the next 24-72 hours. Small means: takes under 30 minutes, doesn't depend on someone else, the user can mark it done by themselves.

Format:

> One thing you'll do: <action, with a verb>. By when: <timestamp>. How you'll know it's done: <observable>.

If the user can't name an action, that's fine — sometimes the reflection itself is the action. Note that explicitly: "no follow-up action; reflection was the work."

## Phase 9 — Save (or not)

Ask:

> Want me to save this reflection? (yes / no / edited)

If yes, write to `<journal-path>/YYYY-MM-DD-<slug>.md` with this structure:

```markdown
---
date: <YYYY-MM-DD>
type: reflection
emotion: <one or two words from Phase 2>
distortions: [<list from Phase 5, may be empty>]
action: <Phase 8 action, or "none — reflection was the work">
---

# Reflection — <slug>

## Situation
<Phase 3 verbatim>

## Automatic thought
> <Phase 4 verbatim>

## Distortions identified
<Phase 5>

## Evidence
**For:** <Phase 6.1>
**Against:** <Phase 6.2>
**If a friend brought this:** <Phase 6.3>

## Reframe
> <Phase 7 final sentence>

## Committed action
<Phase 8>
```

If "edited", let the user redact before save — they may not want a literal record of the situation.

If "no", say so explicitly: "Not saved. The reflection still happened — that's the part that matters."

## Hard refusals

- **No diagnosis.** "You sound depressed" is out of scope. "You said you've been struggling to sleep — that's something to mention to your doctor" is fine.
- **No advice on medication.**
- **No therapy substitute.** If the same theme comes up across three sessions in a row, surface it: "We've worked this same loop three sessions running — it might be worth talking to someone trained for this. I can keep helping you reflect, but I shouldn't be the only thing."
- **No comparative reassurance.** "Other people have it worse" is not a reframe. Don't go there.

## What this skill is NOT

- Not a venting session that ends in "you're so right, that person was awful." The skill is structured precisely because unstructured venting strengthens the loop instead of dissolving it (Bushman 2002 on catharsis as counterproductive).
- Not a productivity hack. It does not promise to make the user "feel better." Sometimes the reflection clarifies that the situation is actually bad and the action is to address it. That's a successful session.

## Sources and rationale

- **CBT thought record** — Aaron T. Beck, *Cognitive Therapy of Depression* (Guilford, 1979); Judith Beck, *Cognitive Behavior Therapy: Basics and Beyond* (3rd ed., Guilford, 2020).
- **Cognitive distortion list** — David D. Burns, *Feeling Good: The New Mood Therapy* (1980), itself derived from Beck.
- **Affect labelling** — Lieberman et al., "Putting Feelings into Words: Affect Labeling Disrupts Amygdala Activity in Response to Affective Stimuli" (*Psychological Science*, 2007); Torre & Lieberman, "Putting Feelings into Words: Affect Labeling as Implicit Emotion Regulation" (*Emotion Review*, 2018).
- **Distanced self-talk** — Kross et al., "Self-talk as a regulatory mechanism: How you do it matters" (*Journal of Personality and Social Psychology*, 2014).
- **Catharsis is counterproductive** — Bushman, "Does Venting Anger Feed or Extinguish the Flame?" (*Personality and Social Psychology Bulletin*, 2002).
- **Stoic journaling lineage** — Marcus Aurelius, *Meditations*; Pierre Hadot, *Philosophy as a Way of Life* (1995); for popular framing, Ryan Holiday, *The Daily Stoic* (2016).
- **Trauma-informed limits** — SAMHSA, *Concept of Trauma and Guidance for a Trauma-Informed Approach* (2014).
- **No-catharsis-via-rumination** — Nolen-Hoeksema, "The Role of Rumination in Depressive Disorders and Mixed Anxiety/Depressive Symptoms" (*Journal of Abnormal Psychology*, 2000).

These sources are why the structured flow is the way it is — emotion-then-evidence-then-reframe-then-action is a deliberate sequence, not a brainstorm.
