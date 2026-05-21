---
name: surge-competitor-citation-analysis
description: Identify exactly where competitors win AI citations the user doesn't — by tracked prompt, by topic, by citing domain — and produce a prioritized list of gaps with the specific reasons competitors are cited. Use this skill whenever the user asks who's beating them on AI search, wants a competitor analysis, asks why a specific competitor is cited and they're not, mentions losing visibility to competitors, asks for a citation gap report, wants to know what competitors are doing differently, or asks to compare their AI presence against named competitors — even if they don't say "citation" or "competitor analysis" explicitly. Also use after a weekly review surfaces that the user's citation rank dropped.
---

# Competitor citation analysis

This skill answers a single question: where are competitors cited that the user isn't, and why?

Output is a ranked list of citation gaps, each with concrete evidence (which prompt, which engine, what the competitor's cited passage contains).

## Quick Start

1. Confirm the project ID and the competitors to compare against.
2. Pull own-domain citation rank and citing-domain rankings.
3. For each tracked prompt, identify which competitors are cited and which engines cite them.
4. For top gaps, pull the actual prompt responses to see what the competitor is doing.
5. Cross-reference with topic gaps to identify whether the issue is missing content or under-optimized content.
6. Produce a prioritized gap report with concrete observations per row.

## Workflow

### 1. Confirm scope

Ask if not provided:

- Project ID.
- Competitors — if the user lists names, use them. If not, derive the top 3-5 competing domains from the citing-domain data (excluding the user's own domain, news sites, and Wikipedia).

### 2. Pull citation rankings

Two queries:

- **Top citing domains** — domains AI engines reference across the project's tracked prompts.
- **Own-domain citation rank** — where the user's domain falls in that ranking.

For each competitor, record total citations and rank.

### 3. Per-prompt competitor presence

For each tracked prompt:

- Pull citation data scoped to that prompt.
- Record which engines cite the competitor.
- Record whether the user is cited (yes / no / partially — some engines but not others).

Group into three buckets:

- **Pure gap**: competitor cited, user not cited, any engine.
- **Engine asymmetry**: user cited on engine A, competitor cited on engine B (different engines).
- **Joint presence**: both cited, but compare position and prominence.

### 4. Pull evidence for top gaps

For the top 5-10 pure gaps, pull the actual prompt responses. Note:

- The passage the engine cited from the competitor.
- Format (list, paragraph, table).
- Length (rough word count).
- Named entities mentioned.
- Sources backing the passage.

This evidence is what makes the skill actionable — the user needs to know *what* the competitor said, not just that they said something.

### 5. Cross-reference with topic gaps

For each gap, query topic-gap data for the relevant topic. Determine:

- **Missing content**: user has no article covering this topic. Action → [[surge-opportunities-to-content]] or [[surge-topic-research-to-article]].
- **Under-optimized content**: user has an article but it's not extractable. Action → [[surge-optimize-content]].

### 6. Produce the gap report

Structured markdown:

```markdown
# Competitor citation gaps — [Project] — vs [Competitors]

## Summary
- N pure gaps across M tracked prompts.
- User's citation rank: #X of Y citing domains (vs. competitor ranks: ...).
- K gaps map to missing content; L gaps map to under-optimized content.

## Top gaps

### 1. Prompt: "[prompt text]"
- **Cited competitor**: domain (engine — engine — engine)
- **Their passage**: [excerpt, 1-2 sentences]
- **Why**: list / table / fact density / etc.
- **Action**: [[link to recommended skill]]

[repeat 5-10x]

## By competitor

| Competitor | Citation count | Top engines | Top topics |
|---|---|---|---|

## Recommended next steps
- [2-4 prioritized actions]
```

## CLI shortcut (faster path)

If the user has the SurgeGraph CLI installed, `surgegraph visibility citation-domains rank-shift --project <id> --window 7d` returns the citation-domain delta from the local cache. Useful when the user wants to see *changes* in competitor presence vs. an absolute snapshot. The MCP chain above gives the absolute view.

## Decisions

| Situation | What to do |
|---|---|
| User on Trial plan | Only ChatGPT and Google AI Overview will appear in engine breakdowns. State this once in the report header. |
| User names a competitor not present in citation data | The competitor may not be cited for the user's tracked prompts. Surface this — it's a finding, not a failure. They're either not competing on these prompts or AI engines don't recognize them as a source. |
| User asks "how do I beat them" | This skill diagnoses; it doesn't draft content. Route to [[surge-opportunities-to-content]] for new articles or [[surge-optimize-content]] for existing ones. |
| Too few tracked prompts to be meaningful | Surface the limit. Comparison is noisy below ~10 tracked prompts. Suggest expanding tracking via [[surge-setup-aeo-tracking]]. |
| User wants a one-shot vs. ongoing competitive monitoring | This skill is one-shot. For ongoing, suggest scheduling [[surge-weekly-visibility-review]] which surfaces citation movements alongside other metrics. SurgeGraph does not push alerts. |

## Common Issues

- **All competitors show low citation counts** — could be a small-sample issue (few tracked prompts, short refresh history). Could also genuinely mean those competitors aren't winning AI citations. Distinguish by checking total citing-domain volume.
- **Top citing domains are mostly Wikipedia, Reddit, news outlets** — these aren't direct competitors. Filter them out before ranking competitors. The user wants to compare against peers, not infrastructure-layer sources.
- **A competitor appears at #1 with one citation** — sort ties carefully. Cap by minimum citation count (e.g. ≥3) before ranking.
- **Engine asymmetry is the whole story** — sometimes a competitor only wins on one engine. That's a real finding; the recommended action is different (engine-specific optimization vs. content gap). Don't collapse this into a generic gap.

## References

- [[surge-setup-aeo-tracking]] — prerequisite (need tracked prompts and refresh history).
- [[surge-weekly-visibility-review]] — surfaces citation movements that motivate this skill.
- [[surge-opportunities-to-content]] — act on missing-content gaps.
- [[surge-optimize-content]] — act on under-optimized-content gaps.
