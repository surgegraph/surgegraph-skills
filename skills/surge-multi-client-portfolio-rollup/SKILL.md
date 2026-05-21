---
name: surge-multi-client-portfolio-rollup
description: Aggregate AI Visibility metrics across all SurgeGraph projects in an organization for an agency portfolio view — one row per client, ranked by citation share, week-over-week movement, sentiment, and other agency-relevant signals. Use this skill whenever the user manages multiple SurgeGraph projects, asks for an organization-wide view, wants to compare clients side-by-side, asks "how are my clients doing," needs to identify which client needs attention most, asks for a portfolio overview or dashboard, or wants a quick scan of agency book-of-business AEO health — even if they don't say "portfolio" or "rollup." Also use as the entry point for agency Monday-morning reviews.
---

# Multi-client portfolio rollup

Agencies running SurgeGraph for many clients spend significant time switching between projects to assemble a portfolio view. This skill produces that view in one call: one row per project, comparable side-by-side.

The output is data-shaped (sorted table). It is not a report deliverable — agencies build their own deliverables on top of this.

## Quick Start

1. The skill auto-detects all projects in the user's current organization.
2. Pull headline metrics per project: citation share, share of voice, WoW delta, sentiment, last-refresh recency.
3. Rank by the user's chosen sort (default: WoW delta, biggest losers first — these need attention).
4. Output a table.

## Workflow

### 1. Pull the project list

The skill scopes to the user's current organization. Pulls all active projects (skips demo projects).

### 2. Per-project metrics

For each project, fetch:

- **Citation share** — current 7-day average.
- **WoW delta** — change vs. prior 7-day window.
- **Share of voice** — current value.
- **Sentiment composite** — positive/neutral/negative weighted score.
- **Volatility flag** — true if the project's data has high variance (more than ±20% across the window).
- **Last refresh** — timestamp of latest refresh cycle.

All deltas are computed against 7-day rolling averages — single-day comparisons are too noisy for portfolio review. Industry data: only ~20% of brands stay visible across 5 consecutive runs, so daily numbers are mostly random sampling variance.

### 3. Rank by relevance

Default sort: WoW delta ascending — biggest *losers* first. Agencies need to triage attention, and the projects in decline are where attention is most valuable.

Other sort options:

- **Biggest gainers**: WoW delta descending. Useful for "what's working" review.
- **Lowest citation share**: helps identify under-performing clients regardless of recent movement.
- **Sentiment risk**: most-negative sentiment first.
- **Alphabetical**: for systematic review.

### 4. Output

Structured table, one row per project:

```
| Project          | Citation Share | WoW Δ   | Share of Voice | Sentiment | Volatility | Last Refresh |
|------------------|----------------|---------|----------------|-----------|------------|--------------|
| Acme Corp        | 12.4%          | -2.1pp  | 8.7%           | +0.3      | High       | 2h ago       |
| Globex           | 8.2%           | +1.8pp  | 6.1%           | +0.5      | Low        | 6h ago       |
...
```

Followed by a short narrative summary: "X projects gained, Y declined, Z need attention this week."

## Decisions

| Situation | What to do |
|---|---|
| User is a solo brand (one project in the org) | This skill is meant for agencies. Surface that and pivot to [[surge-weekly-visibility-review]] for single-project review. |
| Organization has 30+ projects | Cap the table at top 20 by the chosen sort. Offer to filter by sentiment or volatility for the long tail. |
| Some projects have insufficient history for WoW deltas | New projects (less than 2 refresh cycles) get `N/A` in the delta column. Flag in the narrative. |
| User asks "which clients are most at risk of churning" | The portfolio rollup is the input. Combine: declining WoW delta + negative sentiment + high volatility = risk. Don't auto-compute "churn risk" as a single score — too many false positives. Surface the inputs and let the user judge. |
| User wants per-engine portfolio breakdown | Engine-by-engine portfolio view is overkill for most agencies. If asked, output a separate table per engine — but flag it as a deep-dive view. |
| Refresh cadence varies across projects | Surface last-refresh timestamps. Plan-gated cadence means projects on lower tiers refresh less often; that's not a bug. |

## Common Issues

- **All projects show the same WoW delta** — almost always means the refresh hasn't run since the comparison window. Check last-refresh timestamps. Wait for the next cycle.
- **Some projects missing from the rollup** — projects with no tracked prompts produce no citation data. Surface a "0 tracked prompts" note for these rather than dropping them silently.
- **Demo projects in the rollup** — should be filtered by default. If they appear, the org has demo access and the user can ignore them.
- **Sentiment column is empty for some projects** — sentiment requires sufficient prompt responses to compute reliably. Sparse data shows `N/A`.

## References

- [[surge-weekly-visibility-review]] — single-project deep-dive after the rollup identifies a project to focus on.
- [[surge-cross-client-anomaly-detector]] — companion skill that pre-filters the rollup to only show notable changes.
