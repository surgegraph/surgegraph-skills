---
name: surge-opportunities-to-content
description: Turn SurgeGraph AI Visibility opportunities into a content production plan — pick the highest-value gaps where competitors are cited but the user isn't, generate Writer documents seeded with the context, and optionally publish to WordPress. Use this skill whenever the user asks what content they should write next, wants to close citation gaps with competitors, asks how to improve their AI Visibility metrics, wants a content backlog from their analytics, sees the Opportunities dashboard and wants to act on it, asks "where am I losing to competitors on AI search," or wants to bulk-create articles targeted at specific AI-search gaps — even if they don't mention "opportunities" or "content production." Also use as the natural next step after a weekly review surfaces gaps.
---

# From AI Visibility opportunities to shipped content

The Opportunities dashboard in SurgeGraph surfaces specific gaps — prompts where competitors are cited but the user isn't, topics with high AI traffic but no tracked prompts, response structures that favor formats the user doesn't produce. This skill closes that loop: pick the highest-value opportunities, generate articles, optionally publish.

This is the canonical analytics → production workflow.

## Quick Start

1. Confirm the project ID.
2. Pull the opportunities list, sorted by impact.
3. With the user (or via a "top N" default), pick which opportunities to act on.
4. For each picked opportunity, derive a content brief from its context.
5. Bulk-generate Writer documents.
6. Optionally publish via [[surge-publish-to-cms]].

## CLI shortcut (faster path)

If the user has the SurgeGraph CLI installed: `surgegraph research gaps publish --project <id> --research-id <id> --integration <wp_id> --dry-run` runs the full pipeline in one idempotent command, with a dry-run preview. Use it when:

- `which surgegraph` succeeds, AND
- A relevant topic_research record already exists, AND
- The user wants the full pipeline (not just briefs).

Both paths converge on the same outcome.

## Workflow

### 1. Pull opportunities

Fetch the project's AI Visibility opportunities. Each opportunity has:

- A **type** — citation gap, topic gap, response-structure gap, sentiment recovery, etc.
- A **target** — the prompt or topic it relates to.
- **Evidence** — which competitors are winning, what they're doing differently.
- A **priority score** — SurgeGraph's estimate of impact.

Default sort: priority score descending.

### 2. Filter and rank

If the user gave criteria, apply them:

- "Only citation gaps where competitor X is cited" — filter.
- "Only opportunities above traffic threshold N" — filter.
- "Top 10" — slice.

If the user gave no criteria, present the top 5 and ask which to act on.

### 3. Derive a content brief per opportunity

For each picked opportunity:

- **Topic** — from the opportunity's tracked prompt.
- **Target prompts** — the specific user queries the article should answer.
- **Outline** — based on the opportunity's evidence:
  - Citation gap → match the structure and depth of the winning competitor's cited passage.
  - Topic gap → cover the sub-topic the user's content misses.
  - Response-structure gap → use the format AI engines extract from for this prompt (list, table, comparison, etc.).
- **Suggested length** — based on the response-structure data; if AI engines cite passages around N words, aim for sections that produce passages around that length.
- **Knowledge library** — if the project has a knowledge library, attach it as grounding so Writer doesn't hallucinate.

### 4. Generate Writer documents

Use the bulk document creation tool. Each document gets:

- Title derived from the opportunity's target prompt.
- The brief from step 3.
- An `external_id` derived from the opportunity ID — makes the operation idempotent on retry.

Bulk generation is async on the backend. Confirm to the user how many were queued and where to find them.

### 5. Optional: publish

If the user wants the articles to go live, chain into [[surge-publish-to-cms]] one document at a time. Default publish status: `draft`. Live publication should always be an explicit user opt-in.

## Decisions

| Situation                                                             | What to do                                                                                                                                                                                                                                 |
| --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| No opportunities returned                                             | The project may be new (need ≥2 refresh cycles to surface opportunities), or it may already be performing well. Check refresh cadence and refresh history before suggesting either explanation.                                            |
| User says "do them all"                                               | Cap at 10 by default. Bulk generation consumes Writer credits — surface that and confirm.                                                                                                                                                  |
| Opportunity is about brand mentions, not content                      | Some opportunities are "the brand is mentioned on Reddit but not on the user's site" — these aren't writeable as Writer documents. Filter these out and tell the user; they need a different play (a brand-authority effort, not content). |
| User wants to act on opportunities across multiple projects           | Run this skill once per project. There's no portfolio-level opportunities action today.                                                                                                                                                    |
| Knowledge library should be attached but the project doesn't have one | Skip the attachment; flag in the output that grounding could improve quality. Suggest creating a knowledge library if the user has authoritative source content.                                                                           |
| Writer credit quota nearly exhausted                                  | Surface the quota state before queueing bulk generation. Don't queue more than the user has budget for.                                                                                                                                    |

## Common Issues

- **"No opportunities yet"** — needs at least 2 refresh cycles of tracking data. Re-run after the next refresh. Refresh cadence is plan-gated.
- **Generated articles read generic / off-brand** — without a knowledge library or strong brand voice context, Writer falls back to generic prose. Attach a knowledge library or pass brand voice settings before re-running.
- **Bulk creation succeeded but documents never appeared** — bulk generation is async. Check the Writer documents list after a few minutes; if still missing, the OpenAI batch may have failed silently. Contact `hello@surgegraph.io` with the batch ID.
- **Duplicate documents created on retry** — shouldn't happen with `external_id` set. If it did, the retry path bypassed the idempotency key — surface to support.
- **Opportunity targets a topic the user doesn't want to compete on** — let the user filter aggressively. The opportunity score is a heuristic; user judgment on relevance is final.

## References

- [[surge-setup-aeo-tracking]] — prerequisite (tracking history needed for opportunities to surface).
- [[surge-weekly-visibility-review]] — surfaces gaps that motivate using this skill.
- [[surge-publish-to-cms]] — final step when articles are ready to ship live.
- [[surge-optimize-content]] — adjacent skill for _existing_ articles vs. this skill for _new_ articles.
