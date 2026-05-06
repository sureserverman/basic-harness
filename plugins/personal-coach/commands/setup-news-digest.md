---
description: One-shot setup for a recurring news-digest scheduled task in Claude Cowork — checks that news-preferences exists (and runs that skill if not), then walks the user through scheduling a daily digest at their preferred time. In Claude Code (no scheduler), prints a shell-cron equivalent.
---

# Setup News Digest

Wire the `news-digest` skill into Cowork's Scheduled Tasks so a personalized digest lands every morning before the user opens any other app.

## Step 0 — Preferences gate

Before scheduling anything, check whether the user has news preferences saved. Look for:

- `<vault>/Profile/news-preferences.md` (vault-bootstrapped path), OR
- `~/.claude/news-preferences.md` (default path).

If neither exists, tell the user:

> You don't have news preferences saved yet. The digest needs them — without preferences it's just generic headlines, which isn't the point. Let's run the `news-preferences` skill first to capture topics, sources, exclusions, and cadence; then come back here.

Hand off to `news-preferences`. Do not proceed with scheduling.

If preferences exist, read them — especially the **Cadence** section. The user may have already specified time + days there, in which case Step 2 just confirms.

## Step 1 — Detect host

Cowork → `/schedule`. Code → shell cron. Same logic as `setup-morning-briefing`.

## Step 2 — Confirm cadence

Read the **Cadence** section of news-preferences. If `Default firing time` and `Days` are set, ask:

> Your news preferences say `<HH:MM>` `<days>` `<TZ>`. Use that for the scheduled digest? (yes / change / different)

If no cadence is set, ask the two questions:

1. "Time? (24-hour)"
2. "Days? (`weekdays` / `daily` / custom)"

If the answer differs from preferences, ask whether to update the preferences file too — the user may want one cadence here and a different one as the default.

## Step 3a — Cowork path

Present the scheduled-task prompt:

```text
Run my news digest for today.

1. Read my news preferences from <vault>/Profile/news-preferences.md
   (or ~/.claude/news-preferences.md).
2. Run the news-digest skill end to end:
   - Plan queries from Topics-of-interest weighted as configured.
   - Run the queries (WebSearch + WebFetch; Gmail/Drive connectors when granted).
   - Apply filters: hard exclusions strict, soft exclusions per topic, sources-to-avoid,
     recency cap.
   - Synthesize per format preferences (length, style, structure, per-topic cap).
   - Check the open-questions watchlist; surface movement, otherwise say "no update".
3. Save the digest to the standard path: <vault>/News/YYYY-MM-DD-digest.md
   (or ~/.claude/news/YYYY-MM-DD-digest.md).
4. Do not fabricate items. If sources are unreachable, write that explicitly with the
   timestamp and the queries attempted. An empty digest is better than a fake one.
5. Stop. Do not auto-open or notify; I'll read it when I sit down.
```

Then tell the user:

> Cowork → **Scheduled** → **+ New task** → paste prompt → time `<HH:MM>`, days `<answer>` → save.
>
> Recommended Cowork connectors for richer digests: **Gmail** (newsletters folder/label) and **Google Drive** (only if you keep a curated reading-list folder there). Skip Slack — too noisy.
>
> The digest runs even when you're not at the desk; it lands in `News/<date>-digest.md` and you read it whenever.

## Step 3b — Code path

```cron
<minute> <hour> * * <day-pattern> /usr/local/bin/claude -p "Run the news-digest skill against my saved preferences and save to the standard path" >> ~/.claude/journal/cron.log 2>&1
```

Tell the user that the Code path uses `WebSearch` / `WebFetch` only — no Gmail or Drive connectors, so newsletter-derived items won't appear unless the digest URLs are listed directly in news-preferences `Preferred sources`.

## Step 4 — Save the choice

Append to `~/.claude/personal-coach.local.md`:

```markdown
- <YYYY-MM-DD> setup-news-digest — host: <Cowork | Code>, time: <HH:MM>, days: <pattern>, connectors: <gmail | drive | none>
```

## Step 5 — Hand off

> Setup recorded. First digest fires `<HH:MM>` `<TZ>` on the next matching day. The digest reads `news-preferences` every run — if you change interests, sources, or exclusions in that file, the next morning's digest reflects them automatically; you don't need to re-run this command.
>
> If you also set up `morning-briefing` and want them bundled into one scheduled task instead of two separate ones, use `/personal-coach:setup-morning-briefing` and answer "yes" when it asks whether to include news-digest.

## Privacy note (mention briefly)

> A note on Cowork Routines vs Scheduled Tasks: Routines run in Anthropic's cloud (laptop closed, etc.). For news digests this is mostly fine because news search queries already touch the public web. But if your **Hard exclusions** include identity-related topics (health, sexuality, recovery), the cloud-routed search query for the *interest* topic still passes through Anthropic — even though the digest excludes the *result* topic. If that matters to you, stick to desktop Scheduled Tasks (this command's default).

## Hard rules

- **No scheduling without preferences.** Step 0 is a gate, not a recommendation.
- **No editing news-preferences from this command** (other than the cadence section, with explicit user consent). Use the `news-preferences` skill for everything else.
- **No "test the digest now" auto-run.** The skill is daily; the test is tomorrow morning.
