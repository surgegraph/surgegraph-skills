---
name: surge-emerging-topics-monitor
description: Surface new topics that AI search engines are increasingly answering questions about in the user's space — topics that aren't yet tracked in their SurgeGraph project. Use this skill whenever the user asks what's new or trending in their industry, wants topic discovery, asks what they should track next, wonders what queries are emerging in AI search, asks about hot topics in their niche, wants to expand their tracking surface, or asks for proactive recommendations on AI Visibility — even if they don't say "emerging" or "topic." Also use as a recurring check (weekly or biweekly) for active users who want to stay ahead of shifts in AI-search interest.
---

# Emerging topics monitor

This skill surfaces new topics AI engines are increasingly answering questions about — topics adjacent to what the user already tracks but not yet in their tracking surface. Output is a short list of topic candidates with mention counts, examples, and a recommended action per topic.

This is a discovery skill. It does not push notifications — SurgeGraph does not have alerts. Run it on the cadence the user wants.

## Quick Start

1. Confirm the project ID.
2. Pull emerging topics (raw signal).
3. Pull topic-gap data (where the user's existing content doesn't yet cover).
4. Filter, rank, and dedupe.
5. For each candidate, suggest an action: add to tracking, write content, or watch.

## Workflow

### 1. Confirm scope

- Project ID.
- Window: last 7 days, last 30 days, or "since I last ran this." Default: last 30 days for emerging-topic signal, since the signal-to-noise ratio improves over a longer window.
- Minimum mention threshold: ≥3 by default.

### 2. Pull emerging topics

Fetch the emerging topics for the project. Each item includes:

- Topic name (extracted from AI responses).
- Mention count over the window.
- Sample prompts where the topic appeared.
- Adjacency score — how related the topic is to the project's existing topics.

### 3. Pull topic gaps

Fetch topic-gap data to cross-reference. Topics that show up in both lists (emerging + already-flagged-as-a-gap) are the highest-value candidates.

### 4. Rank candidates

Sort by composite score:

1. Mention count above threshold (primary signal).
2. Adjacency score to existing tracked topics (relevance filter).
3. Whether the topic also appears in topic-gap data (validates the gap).

Cap at the top 10. Show top 5 inline, optionally expand.

### 5. Per-topic action recommendation

For each candidate, recommend one of three actions:

- **Add to tracking** — route to [[surge-setup-aeo-tracking]] with the topic name pre-filled, to generate prompts for it.
- **Write content** — route to [[surge-topic-research-to-article]] for a research-grounded article, or [[surge-opportunities-to-content]] if the topic also shows up in opportunities.
- **Watch** — high-noise or low-relevance candidates that may grow. Suggest the user re-run this skill in 2-4 weeks.

### 6. Output

```markdown
# Emerging topics — [Project] — [Window]

## Top candidates

### 1. [Topic]
- Mentions: N over the window
- Adjacency: high / medium / low (to existing topics)
- Examples: [1-3 sample prompts where this surfaced]
- Suggested action: [add to tracking / write content / watch]

[repeat]

## Already in your gap list
[topics appearing in both emerging + topic-gap data]

## Watch list
[lower-confidence candidates worth re-checking later]
```

## Decisions

| Situation | What to do |
|---|---|
| User asks to be alerted when new topics emerge | SurgeGraph does not push alerts or notifications. Suggest scheduling this skill (e.g. `/loop /emerging-topics-monitor` biweekly) instead. |
| User wants to auto-track everything that emerges | Don't. Topic count counts against tracking surface, and unfiltered emergence creates noise. Recommend the user pick deliberately. |
| All "emerging" topics are duplicates of existing tracked topics with slight rewording | This usually means refresh window is too short — the same prompt is being detected fresh. Extend the window to at least 30 days. |
| Mention count looks low across the board | The project may be too new to generate enough refresh data for emergence detection. Suggest re-running after 2-3 more refresh cycles. Refresh cadence is plan-gated. |
| User asks "is this topic trending or just spiking" | Look at the trend curve if available. If only a snapshot is available, recommend re-running in 2 weeks to confirm. |
| Topic is adjacency=low | Surface but flag clearly. Adjacency=low often means a topic surfaced because of a one-off response, not a real expansion of the user's space. |

## Common Issues

- **No emerging topics returned** — either the project is performing perfectly and isn't shifting, or there's insufficient data. Distinguish by checking the project's refresh history. New projects need at least 3-4 cycles to surface emergence.
- **All candidates are low-adjacency** — the model behind emerging-topic detection picked up tangents from off-topic responses. Tighten the threshold or extend the window.
- **Topic names look garbled / fragmented** — extraction quality. Surface to support; meanwhile, the user can manually interpret.
- **Same topic keeps appearing as "emerging" across multiple runs** — it's not really emerging if it persists. Treat as "high baseline" and consider whether the topic should be tracked normally via [[surge-setup-aeo-tracking]].

## References

- [[surge-setup-aeo-tracking]] — natural next step for emerging topics worth tracking.
- [[surge-topic-research-to-article]] — write content for an emerging topic.
- [[surge-opportunities-to-content]] — act on emerging topics that also surface as content opportunities.
- [[surge-weekly-visibility-review]] — recurring review skill that may surface emerging topics inline.
