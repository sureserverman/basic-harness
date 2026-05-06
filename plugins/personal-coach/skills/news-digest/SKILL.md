---
name: news-digest
description: Use to produce a personalized daily news digest against the user's saved news preferences. Reads news-preferences, runs targeted web searches per topic, filters by sources and exclusions, synthesizes per the user's format prefs, checks the open-questions watchlist, and writes a dated digest to disk. Triggers on "give me my news", "news digest", "what's new on <topic>", "morning news", "what should I know today", or whenever the user asks for a curated daily readout. Refuses to run without preferences — generic headlines are not the point.
---

# News Digest

Produces a personalized news digest against the user's saved preferences. The whole skill is structured around **filtering by what the user has explicitly told us they want** — without that filter, news-digest is just a slower version of any news app.

**Announce at start:** "Using the news-digest skill. I'll run against your saved preferences and produce today's digest."

<HARD-GATE>
Refuse to run if `news-preferences` has not been initialized. Tell the user: "I don't have your news preferences yet — run the `news-preferences` skill first to tell me what you actually want to track. Generic headlines aren't useful and I won't fake them." Hand off to that skill.
</HARD-GATE>

<HARD-GATE>
Hard exclusions in the preferences file are absolute. The digest never includes excluded topics, no matter how "relevant" they seem. Soft-exclusion ("not preferred") is different from hard-exclusion ("never").
</HARD-GATE>

## Where the digest lives

If a personal vault is configured, save to `<vault>/News/YYYY-MM-DD-digest.md`. Otherwise, `~/.claude/news/YYYY-MM-DD-digest.md`. Append-only — old digests are kept so the user can re-read what they noticed before.

## Phase 1 — Read preferences

Read the news preferences file (see `news-preferences` skill for path resolution). Extract:

- Topics of interest with weights.
- Topics excluded.
- Hard exclusions (strict filter).
- Preferred sources.
- Sources to avoid.
- Language settings.
- Format preferences (length, style, structure, per-topic cap, "why this matters" inclusion).
- Open questions on watch.

If the file is malformed (missing required sections), surface the issue to the user and offer to fix via `news-preferences`. Do not improvise.

## Phase 2 — Plan the queries

For each **Topic of interest**, plan 1-3 search queries. Weight scaling:

- **High-weight topics:** up to 3 queries; aim for 3-5 items in the digest.
- **Medium-weight:** 1-2 queries; 1-3 items.
- **Low-weight:** 1 query; 0-2 items (skip if nothing fresh).

For each **Open question on watch**, plan 1 search query phrased as "<question topic> latest <YYYY-MM>". The threshold for inclusion is higher: only surface if there's actually news, never to fill space.

Query construction:

- Include the topic phrase plus a freshness qualifier ("today", "this week", "<current month>").
- For regulation / policy topics, prefer official sources: "site:eur-lex.europa.eu", "site:fca.org.uk", "site:cbr.ru", etc., before secondary coverage.
- Respect language settings: search in the user's primary language; auto-translate from listed source languages.

## Phase 3 — Fetch (Code or Cowork)

The skill uses whichever surfaces are available:

- **`WebSearch`** — primary mechanism; works in both Code and Cowork.
- **`WebFetch`** — when a specific URL is in the preferences (e.g., a named outlet's RSS or a regulator's announcements page), fetch directly.
- **Cowork connectors** when granted:
  - **Gmail** — read newsletters tagged or in a labeled folder the user names (e.g., `Newsletters/Daily`). Treat the newsletter content as a first-class source.
  - **Google Drive** — if the user has a folder of clipped articles or a curated reading list in Drive, optionally consult.
  - **Slack** — generally avoid; team chat is not news.

Fetch up to ~30 candidates total across all queries. More than that and the synthesis quality drops; fewer and the digest gets thin.

## Phase 4 — Filter

Apply filters in this order:

1. **Hard exclusions** — drop any item touching a Hard-excluded topic. No appeals.
2. **Sources to avoid** — drop items from these outlets.
3. **Soft exclusions** — drop items that are about a Topic-excluded subject (less strict than Hard, but still drop unless the user has the topic on their **Open questions** list).
4. **Recency** — drop anything older than 48 hours unless it's surfacing under an open-questions watch (where slow movement matters).
5. **Per-topic cap** — for each topic, keep the top items up to the per-topic cap from preferences.

If after filtering a topic has zero items, **say so explicitly** in the digest: "**<Topic>:** nothing material today." Empty-bucket signal is information; silent omission is not.

## Phase 5 — Synthesize per format preferences

Build the digest body using the user's `Structure` preference:

- **Flat bullets** — single chronological list, each item one line.
- **Sectioned by topic** — H2 per topic, items as bullets underneath. Most users prefer this.
- **Sectioned by source** — H2 per source. Useful if the user is doing a press review.

Each item:

```markdown
- **<concise headline rewritten in plain language>** — <one-sentence summary>. _(<source>, <date>)_  [link](<url>)
- **Why this matters:** <one line, only if "include why this matters" preference is on>
```

Style:

- **Facts only** — no interpretation; just what happened.
- **Analytical** — short framing line allowed; never opinion.
- **Analytical with named opinion** — one per digest, max, and labeled "[opinion]" with attribution.

## Phase 6 — Open-questions section

Separate H2 at the bottom of the digest. For each open question the user is tracking:

- If there was movement, surface it: "**<question>:** <one-line update>. <source>."
- If there was no movement, say so: "**<question>:** no update."

This section is what makes the digest a tracking tool, not just a summary.

## Phase 7 — Save

Write the digest to `<news-path>/YYYY-MM-DD-digest.md`:

```markdown
---
date: <YYYY-MM-DD>
type: news-digest
topic_count: <n>
item_count: <n>
empty_topics: [<list of topics with no items today>]
language: <primary>
generated_at: <ISO-8601 timestamp>
---

# News Digest — <YYYY-MM-DD>

<digest body per format preferences>

## On watch

<open-questions section>

---

_Source coverage: <n> queries, <n> candidate items, <n> after filtering. Run on <Code | Cowork>; connectors used: <Gmail | Drive | none>._
```

## Phase 8 — Hand off

If `morning-briefing` is going to run today, mention the digest at the top: "Today's news digest is at `<path>` — fold it into briefing field 1 if anything relevant."

If running as a Cowork **Scheduled Task** with no interactive user, save the digest and stop. Do not auto-open. The user reads it when they sit down.

## Failure modes

- **No preferences file** — refuse, redirect to `news-preferences`.
- **All queries returned nothing** (offline, blocked, rate-limited) — write a digest entry that says so explicitly, with the timestamp and the queries attempted, so the user can see *why* the digest is thin. Do not fabricate items.
- **Hard-excluded topic surfaced anyway** (e.g., a story about an interest topic that incidentally references an excluded topic) — soft note: include the item, but flag the excluded angle: "_(includes peripheral mention of <excluded topic>)_". Don't filter at the item level if the primary topic is wanted.
- **Source unreachable** — note the source under `empty_topics` for the day; don't retry indefinitely.

## Hard rules

- **No fabricated items, ever.** If sources are unreachable or empty, the digest is empty. A fake "based on general knowledge" item is the failure mode this skill exists to prevent.
- **No commentary the user didn't ask for.** Style preference is binding. "Analytical" doesn't mean "the assistant's opinions"; it means short framing lines.
- **Cite every item.** Every bullet has a source name and a working URL. Items without a verifiable link don't ship.
- **Hard exclusions are not negotiable.** Even if the day's biggest story is on the Hard-excluded list, it stays out. The user has chosen to not read about it.
- **No Cowork-cloud Routine for sensitive topics.** If the user has a Hard exclusion that's identity-related (health, sexuality, substance recovery, etc.) and is also running this skill as a cloud Routine, surface the privacy tradeoff: cloud-routed search queries log the topic, even when the digest excludes it. Recommend desktop Scheduled Task instead.

## Sources and rationale

- **Refuse without preferences** — Cal Newport, *Digital Minimalism* (2019): undirected information consumption is the failure mode of every news tool; preferences are the discipline.
- **Per-topic cap and weighting** — Tetlock & Gardner, *Superforecasting* (2015), Ch. 4: signal density beats coverage; capped lists outperform exhaustive ones.
- **Open-questions watchlist** — Heuer, *Psychology of Intelligence Analysis* (CIA, 1999), Ch. 10 on Indicators and Warnings.
- **Empty-bucket explicit-negative** — same principle as the design-doc rollout-section rule (`brainstorming` skill): explicit "no news" beats silent omission.
- **No fabrication** — basic anti-hallucination discipline; cited every-item rule from research-grounded RAG practice.
- **Hard-exclusion strictness** — Tversky & Kahneman, "Judgment under Uncertainty: Heuristics and Biases" (1974), on anchoring: even a single mention reshapes interpretation; soft filters do not work.
