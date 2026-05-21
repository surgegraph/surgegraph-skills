---
name: surge-weekly-visibility-review
description: Produce a structured AI Visibility review for a SurgeGraph project — week-over-week deltas, sentiment shifts, top citing domains, gainers and losers per prompt, and emerging topics worth tracking. Use this skill whenever the user asks how their brand is doing on AI search this week, requests a recurring AEO report, wants to know what changed in their citations, asks about brand visibility on ChatGPT or Google AI Overviews, says "give me the weekly numbers," asks for an executive summary of AI search performance, or wants to monitor competitor citation movements — even if they don't mention "SurgeGraph" or "weekly." Also use for any cadence the user asks for (this week, last 30 days, since X) — just adjust the window.
---

# Weekly AI Visibility review

This skill compiles a single coherent report from the AI Visibility data SurgeGraph already collects: how the brand is performing across answer engines, what changed since the last snapshot, and what to act on.

The skill works against any time window the user asks for. "Weekly" is the default because most teams operate on a Monday cadence — adjust if asked.

## Quick Start

1. Confirm the project ID and the comparison window (default: last 7 days vs. prior 7 days).
2. Pull headline numbers: overview, trend, sentiment.
3. Pull citation data: top citing domains, own-domain citation rank, competitor citation movements.
4. Pull prompt-level detail: biggest gainers and losers vs. prior window.
5. Pull discovery signals: emerging topics, brand mentions.
6. Compose the report as structured markdown.

## CLI shortcut (faster path)

If the user has the SurgeGraph CLI installed and a local cache, `surgegraph visibility delta --project <id> --window 7d --agent` produces the same dataset in one call from the SQLite cache. Use it instead of the multi-tool chain below when:

- The output of `which surgegraph` succeeds, AND
- `surgegraph sync --project <id>` has been run at least twice on different days (snapshots needed for deltas).

Both paths converge on the same report shape.

## Workflow

### 1. Confirm scope

Ask if not provided:

- Project ID (default to the user's primary project if they only have one).
- Window: last 7 days, last 30 days, custom range. Default: last 7 days.
- Comparison: prior window of equal length is the default.

### 2. Pull headline metrics

Fetch the AI Visibility overview for both the current and prior windows. Capture:

- Share of voice across engines.
- Total citations.
- Average position.
- Sentiment distribution.

### 3. Pull trend and sentiment shifts

Trend tool gives day-by-day values across the window. Compute deltas. Sentiment tool gives a positive/neutral/negative breakdown — report changes ≥5 percentage points.

### 4. Pull citation data

Three queries:

- **Top citing domains** — which external sources AI engines reference most when discussing the brand or its topics.
- **Own domain citation rank** — how the brand's own domain ranks among citation sources for its tracked prompts.
- **Competitor domain citations** — for each prominent competitor, citation share.

Flag domains that gained or lost ≥3 ranks vs. prior window.

### 5. Pull prompt-level detail

For each tracked prompt, compare current vs. prior window. Surface:

- **Biggest gainers**: prompts whose citation count, position, or sentiment improved most.
- **Biggest losers**: same metrics, worst movers.
- **Newly cited**: prompts that now receive citations but didn't before.
- **Newly uncited**: prompts that lost all citations.

Cap each list at 5 items.

### 6. Pull discovery signals

- **Emerging topics** — new topics surfacing in answer-engine responses that aren't yet tracked. Filter to ≥3 mentions in the window.
- **Brand mentions** — references to the brand inside AI responses where the brand was the subject (vs. cited as a source).

### 7. Compose the report

Produce structured markdown with this skeleton:

```markdown
# AI Visibility — [Project] — [Window]

## Headlines
- [3-5 bullet points: most material movements]

## Engine performance
| Engine | Citations | Δ | Position | Δ |
|---|---|---|---|---|
...

## Top citing domains
[ranked table, with Δ rank vs. prior window]

## Own-domain citation rank: #N (Δ ±X)

## Prompts — biggest gainers
[5 prompts, with the delta and why]

## Prompts — biggest losers
[5 prompts, with the delta and why]

## Emerging topics
[bullets — topic name, mention count, link to track in SurgeGraph]

## Recommended actions
[2-4 bullets, prioritized]
```

## Decisions

| Situation | What to do |
|---|---|
| User on Trial plan | Only ChatGPT and Google AI Overview will appear in engine breakdowns. Mention this once in the report header. Don't pad the table with engines the plan doesn't cover. |
| User asks how often this report should run | Recommend weekly for active brands, monthly for slower-moving B2B. The actual data refresh cadence depends on the user's plan — don't claim a universal "daily" refresh. |
| User asks to be alerted when a metric drops | SurgeGraph does not push alerts or notifications. Suggest running this skill on a schedule (e.g. `/loop /weekly-visibility-review`) or via the CLI in cron. Do not promise pings, emails, or webhooks. |
| Insufficient snapshot history | If the project was created less than 2 refresh cycles ago, there's no prior window to compare against. Produce a "baseline" version of the report with current numbers only and tell the user a delta report needs at least 2 cycles. |
| User wants competitor analysis | The citation-domain tools cover this. Don't speculate beyond what the data shows — name only competitors the user already mentioned as competitors. |

## Common Issues

- **"No snapshots yet" on CLI path** — the local cache needs at least 2 syncs on different days. Run `surgegraph sync --project <id>` once, wait a refresh cycle, sync again, then retry.
- **All zero values across engines** — the project may have no tracked prompts. Use [[surge-setup-aeo-tracking]] first.
- **Numbers don't match the dashboard** — the dashboard's "last X days" rounding may differ. Confirm window boundaries with the user; SurgeGraph dashboards use the user's local timezone, this skill operates in UTC unless overridden.
- **Trend tool returns sparse data** — for low-traffic prompts or short windows, individual days may have zero observations. Surface this as "thin data — extend the window to N days for stronger signal" rather than reporting noisy deltas.

## References

- [[surge-setup-aeo-tracking]] — required before this skill produces meaningful output.
- [[surge-opportunities-to-content]] — natural next step after a review surfaces gaps.
