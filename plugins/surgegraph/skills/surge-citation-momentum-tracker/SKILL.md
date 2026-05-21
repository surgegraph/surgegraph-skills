---
name: surge-citation-momentum-tracker
description: Identify external URLs (own pages and competitor pages) that are gaining or losing AI engine citations week-over-week — the page-level "what's rising and falling in AI Visibility" view. Use this skill whenever the user wants to know which pages are gaining ground in AI citations, asks what content is rising or falling, wants to identify trending source pages, asks who's gaining citation share at the URL level, wants to spot a competitor's new ranking content before it's obvious, asks about citation momentum or movement, or wants to learn from pages that are suddenly winning citations — even if they don't say "momentum." Also use after [[surge-competitor-citation-analysis]] surfaces a top citing domain and the user wants to drill into the specific pages.
---

# Citation momentum tracker

Aggregate citation tracking shows *who* is winning — domains, brands. Citation momentum shows *what* is winning — specific URLs that are gaining (or losing) citations week-over-week. This is page-level intelligence that aggregate trackers miss.

Use cases:

- **Defensive**: spot when one of the user's pages is *losing* citation share before it drops below the threshold of visibility.
- **Offensive**: spot competitor pages that are suddenly winning citations, study their structure, and counter-publish.
- **Learning**: identify own-page winners — "this article gained 40 citations this week, what made it work?"

## Quick Start

1. Confirm the project ID and the time window (default 7 days vs. prior 7).
2. Pull citation data over the window, grouped by URL.
3. Compute per-URL deltas.
4. Rank by absolute delta — biggest gainers AND biggest losers.
5. Surface 5-10 of each, with the engines that drove the change.

## Workflow

### 1. Confirm scope

Defaults:

- **Window**: last 7 days vs. prior 7 days.
- **Scope**: all URLs across the project's tracked prompts (own domain + competitors + reference sources like Wikipedia and Reddit).

User can narrow:

- "Own domain only" — defensive view of the user's pages.
- "Competitor domains" — offensive view.
- "Excluding Wikipedia/Reddit" — drop infrastructure-layer sources that dominate noise.

### 2. Pull citation data over time

The momentum tool queries citation history bucketed by week and grouped by URL. Each row has: URL, current-week citation count, prior-week citation count, per-engine breakdown.

### 3. Rank

Two complementary rankings:

- **Biggest gainers**: top URLs by positive delta. Each entry includes which engines drove the gain.
- **Biggest losers**: top URLs by negative delta. Each entry flags whether the URL still appears at all (some losers drop to zero — those are the most urgent).

### 4. Annotate

For each surfaced URL, add a one-line "why this matters":

- **Own-domain gainer**: "Your article. Study what's working — replicate the format, headings, citations."
- **Own-domain loser**: "Your article. Run [[surge-page-citability-score]] to diagnose; consider [[surge-optimize-content]]."
- **Competitor gainer**: "Competitor page. Study and counter — see [[surge-opportunities-to-content]]."
- **Competitor loser**: "Competitor losing ground. Their replaceable territory."
- **Wikipedia/Reddit movement**: "Source authority shift. May reflect AI engine ranking changes more than content changes."

## Decisions

| Situation | What to do |
|---|---|
| URL has noisy delta (gain one engine, lose another) | Net delta is the headline; per-engine breakdown is in the row. Surface both — agencies want to see "moved on ChatGPT but not Perplexity" because the prescription differs. |
| Project has limited citation history | Need at least 2 weeks of refresh cycles for meaningful momentum. Surface the limit; suggest re-running in 1-2 weeks. |
| URL momentum doesn't match overall visibility delta | Possible if the citation movement is on long-tail prompts not driving top-line visibility. Don't over-interpret — both views are valid for different questions. |
| User asks to track a specific URL's momentum over time | Out of scope for this skill (one-shot view). Pair with [[surge-weekly-visibility-review]] run repeatedly, or schedule via `/loop`. SurgeGraph does not push alerts on URL-level changes. |
| User wants competitor momentum but the competitor isn't in the data | Citation data includes whatever URLs the AI engines cited in responses to tracked prompts. If a competitor never appears, they don't rank for the user's tracked topics. That's a finding. |

## Common Issues

- **All movement is in Wikipedia/Reddit** — these dominate citation source pools. Use the "exclude Wikipedia/Reddit" filter when looking for actionable competitor signals.
- **Deltas all look small** — citation counts at the URL level are small numbers (often single digits per week). A delta of +3 is meaningful. Don't expect percentage-style swings.
- **A URL gained citations on a different prompt than expected** — citation data is per-prompt. The URL may have gained on Prompt B but lost on Prompt A. Drill into the per-prompt detail if needed.
- **URL momentum doesn't match the dashboard** — momentum view operates at URL level, not domain level. Dashboard often aggregates to domain. Both are correct views.

## References

- [[surge-competitor-citation-analysis]] — broader competitive intelligence; this skill drills into URL-level movement.
- [[surge-weekly-visibility-review]] — recurring overview that may surface URLs worth tracking via this skill.
- [[surge-page-citability-score]] — diagnose why a specific URL is gaining or losing.
- [[surge-opportunities-to-content]] — act on competitor URL gains.
