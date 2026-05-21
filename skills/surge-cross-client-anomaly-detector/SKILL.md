---
name: surge-cross-client-anomaly-detector
description: Across all SurgeGraph projects in an organization, surface only the clients with statistically notable changes — gainers, losers, sentiment shifts, volatility spikes — pre-filtered for an agency reviewing many clients at once. Use this skill whenever an agency user manages multiple projects and wants noise filtered, asks "who needs attention this week," wants a digest of portfolio movements, needs to triage where to focus their time, asks for a portfolio-level signal-not-noise view, wants to know which clients to email about urgent changes, or asks for an exception report — even if they don't say "anomaly." Also use as the recurring Monday-morning triage skill for agencies running 10+ clients.
---

# Cross-client anomaly detector

[[surge-multi-client-portfolio-rollup]] gives the full portfolio table. This skill pre-filters that view to show only the projects with notable, signal-grade movements — saving the agency from scanning 30 rows to find the 3 that matter.

A "notable" movement means the delta exceeds the project's noise floor by a clear margin. This skill is the agency Monday-morning triage skill: which clients warrant a call, an email, a deeper dive.

## Quick Start

1. Auto-scope to the user's organization.
2. For each project, compute WoW deltas across key metrics.
3. Apply per-project noise floor (each project has its own volatility profile).
4. Surface only the projects with deltas exceeding noise floor.
5. Categorize: gainers, losers, sentiment shifts, volatility spikes.

## Workflow

### 1. Pull project list

All active projects in the organization, excluding demos.

### 2. Per-project signal extraction

For each project:

- Citation share WoW delta (7-day rolling current vs. 7-day rolling prior).
- Share of voice WoW delta.
- Sentiment composite WoW delta.
- Volatility (std-dev of last 14 days of daily values).
- Volatility change WoW (volatility itself can spike — that's a signal).

### 3. Per-project noise floor

Each project's noise floor is computed from its own daily variance over the prior 30 days. Projects with sparse data (new, low-refresh-cadence) have higher noise floors and need larger deltas to flag.

### 4. Flag anomalies

For each project, flag if:

- WoW delta on any metric exceeds 2× the project's noise floor → **material change**.
- Sentiment delta exceeds ±0.2 (on a -1 to +1 scale) → **sentiment shift**.
- Volatility itself spiked (current 14-day std-dev > 1.5× prior 14-day std-dev) → **volatility spike** (often indicates an external event affecting the brand).

A project may be flagged for multiple reasons — list all.

### 5. Categorize and rank

Group flagged projects into four buckets:

- **Big gainers**: positive material change on citation share or share of voice.
- **Big losers**: negative material change.
- **Sentiment shifts**: significant sentiment movement (either direction).
- **Volatility spikes**: instability — even if direction is unclear, this warrants checking.

Within each bucket, rank by absolute delta.

### 6. Output

Short, prioritized list:

```markdown
## Big losers (need attention)
1. [Project] — citation share -3.2pp WoW (noise floor ±1.1pp). Investigate.
2. [Project] — share of voice -2.4pp WoW. Coincides with sentiment drop -0.3.

## Big gainers
1. [Project] — citation share +4.1pp WoW. Study what's working.

## Sentiment shifts
1. [Project] — sentiment -0.4 WoW. May indicate negative press or AI engine retraining.

## Volatility spikes
1. [Project] — volatility 2.1× prior. Often external — check news.

## Quiet (28 projects)
No material change detected. Skip this week.
```

The "Quiet" count is itself useful — it tells the agency owner the bulk of clients are stable and they can focus on the 5 flagged.

## Decisions

| Situation | What to do |
|---|---|
| Organization has only 1-3 projects | This skill is built for agencies with many clients. Surface that; pivot to [[surge-weekly-visibility-review]] per project. |
| All projects flagged | Either there's a real cross-portfolio event (likely an AI engine update affecting many brands at once), or the noise floor calculation is off. Surface that this is unusual and worth investigating both interpretations. |
| Nothing flagged across portfolio | Good outcome — confirm to the user. Suggest re-running next week. Don't manufacture findings to seem useful. |
| Same project flagged repeatedly across runs | That project has a sustained shift, not a one-time spike. After 2-3 consecutive runs, it's a new baseline. Surface this; the noise floor may need to be recomputed against more recent data. |
| User asks for daily anomaly check | The skill is designed for weekly cadence (matches 7-day rolling averages). Daily checks are too noisy by design — single-day movement is usually sampling variance in how AI engines respond, not signal. Industry data: only ~20% of brands stay visible across 5 consecutive runs. |
| Sentiment shift but no citation movement | Possible — sentiment can move independently. Often indicates AI engine retraining picked up a recent piece of content (positive or negative). Surface but don't auto-recommend action; let the user investigate. |

## Common Issues

- **"Big losers" includes projects that gained citations but lost share of voice** — share of voice is relative; competitors winning citations can shift relative share without the user actually losing. Surface both metrics so the user can interpret.
- **New projects always flagged for high volatility** — expected; sparse data means high noise floor and high apparent variance. Surface that projects need 2-3 refresh cycles before anomaly detection produces reliable signal.
- **Volatility spike with no sentiment or citation movement** — usually a refresh-cadence change or external event AI engines partially reacted to. Worth investigating but not necessarily worth client outreach.
- **Output empty when the user expected drama** — sometimes everything is genuinely stable. Don't engineer a finding; trust the noise floor.

## References

- [[surge-multi-client-portfolio-rollup]] — full unfiltered portfolio view; this skill is the filtered triage version.
- [[surge-weekly-visibility-review]] — single-project deep-dive once anomaly detection surfaces which client to review.
