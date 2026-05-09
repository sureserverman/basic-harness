# Routine: News Digest

**Trigger:** schedule, daily at user's preferred time.
**Connectors recommended:** Gmail (for newsletter folder), Google Drive (only if curated reading list).
**Privacy:** acceptable — search queries hit the public web; digest is synthesis of public info. See `README.md` in this directory for the full posture.

## Prompt

```
Run the news-digest skill against my saved news preferences.

1. Read preferences from <vault>/Profile/news-preferences.md
   (or ~/.claude/news-preferences.md if no vault).
2. Plan queries from Topics-of-interest weighted as configured.
3. Run queries: WebSearch + WebFetch primary; consult Gmail label
   "Newsletters/Daily" if granted; consult Drive folder "Reading list"
   if listed in Preferred sources.
4. Apply filters strictly — Hard exclusions are absolute, soft exclusions
   per topic, sources-to-avoid drop, recency cap 48 hours unless on watch.
5. Synthesize per the Format-preferences section: length, style, structure,
   per-topic cap, "why this matters" inclusion.
6. Check the open-questions watchlist; surface movement, otherwise
   write "no update" for each.
7. Save the digest to <vault>/News/<YYYY-MM-DD>-digest.md (or
   ~/.claude/news/<YYYY-MM-DD>-digest.md). Frontmatter and body
   structure per the news-digest skill.
8. Do not fabricate items. Empty buckets are fine; fake items are not.
9. Stop. Do not auto-open or notify.
```

## Cadence

Default: daily at the time set in the user's news-preferences `Cadence` section, in the user's IANA timezone.

If preferences specify only weekdays, set the Routine to weekdays. The Routine's schedule and the preferences file should agree.

## What this Routine will NOT do

- Will not modify news-preferences.
- Will not surface excluded topics, even if a major story breaks within them.
- Will not produce items without verifiable links.
- Will not retry indefinitely on rate-limit / unreachable sources — write the empty digest and stop.

## Verification

The first scheduled run is the test. Open `News/<date>-digest.md` and check:

- Are all your interest topics represented (or explicitly empty-bucketed)?
- Is the format matching your preferences?
- Are open questions surfaced or marked "no update"?
- Are any excluded topics present? (If yes — bug; report.)

If anything is off, run the `news-preferences` skill to refine, then let the next morning's run validate.
