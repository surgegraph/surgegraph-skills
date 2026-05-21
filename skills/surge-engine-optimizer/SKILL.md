---
name: surge-engine-optimizer
description: Per-AI-engine optimization audit — diagnose exactly why a brand is under-cited on ChatGPT vs. Google AI Overviews vs. Perplexity vs. Gemini vs. Bing Copilot, and produce platform-specific fixes for each. Use this skill whenever the user wants to know why one engine cites them and another doesn't, asks about ChatGPT-specific or Perplexity-specific optimization, mentions that visibility varies wildly across engines, wants per-engine action items, asks how to win on a specific AI engine, or wants to understand why each engine surfaces different sources for the same query — even if they don't name the engines explicitly.
---

# Per-engine optimization

Each AI search engine uses different indexes, ranking heuristics, and source preferences. A page that ChatGPT cites may be invisible to Google AI Overviews, and vice versa. This skill produces per-engine scorecards with the specific levers to pull for each.

This skill reuses existing SurgeGraph visibility data — no new tools required. Output is diagnostic, not data-collection.

## Quick Start

1. Confirm the project ID.
2. Pull per-engine visibility data — citation share, position, sentiment.
3. For each engine, identify the dominant gap pattern.
4. Generate per-engine action items grounded in how that engine selects sources.
5. Prioritize across engines based on the user's traffic mix.

## Engine source-selection patterns

A short field guide. Skills built on top of this should keep these patterns up to date as engines change.

### Google AI Overviews

- Strongly favors pages that already rank in the top 10 of organic Google search results.
- Heavy use of tables and lists in AI Overview cards.
- Cites pages with a clear direct answer in the opening sentence of a section.
- Featured-snippet optimization overlaps heavily with AIO optimization.

### ChatGPT (web search)

- Built on Bing's index; not Google's.
- Strong preference for canonical, well-established sources — Wikipedia, Reddit, major publications appear frequently.
- Entity recognition matters: brands with Wikipedia or Wikidata records are more likely to be cited.
- Tends to cite longer, comprehensive articles over short pieces.

### Perplexity

- Heavy use of community-validated sources — Reddit, Q&A sites, discussion forums.
- Cites multiple sources per answer (typically 5-15), creating opportunity for mid-authority sites.
- Recency-sensitive: published / last-updated dates affect ranking.

### Google Gemini

- Built on Google's full search index plus Google-owned properties (YouTube, Knowledge Graph, Google Business Profile).
- Multi-modal: references images and video transcripts alongside text.
- Schema.org structured data is consumed heavily for entity understanding.

### Bing Copilot

- Same Bing index foundation as ChatGPT, different ranking layer.
- Supports IndexNow for near-instant indexing.
- LinkedIn and GitHub content weighted more heavily than on other engines.

## Workflow

### 1. Pull per-engine visibility

For the project, fetch:

- Engine-level citation share, position, and sentiment from the visibility overview.
- Engine breakdown of citation-domain rankings.
- Sample prompt responses per engine to see qualitative differences.

### 2. Identify the dominant gap per engine

For each engine, classify into one (or more) of:

- **Authority gap** — the brand isn't recognized as an entity by this engine's preferred sources (Wikipedia for ChatGPT, Knowledge Graph for Gemini, Reddit presence for Perplexity).
- **Format gap** — the brand's content doesn't match this engine's preferred extraction shape (lists/tables for AIO, comprehensive articles for ChatGPT, fresh community-validated for Perplexity).
- **Index gap** — the brand's pages aren't in this engine's index at all (Bing for ChatGPT/Copilot, lack of IndexNow for Bing).
- **Engine asymmetry** — the brand wins on adjacent engines but not this one; the same content works elsewhere, so the issue is engine-specific signals.

### 3. Generate per-engine action items

For each engine, the prescription differs:

**AIO weakness:**
- Audit Google organic rankings for tracked-prompt queries; pages outside top 10 are unlikely to surface in AIO.
- Restructure cited passages as tables or numbered lists.
- Open H2 sections with a one-sentence direct answer.

**ChatGPT weakness:**
- Audit Wikipedia/Wikidata presence (use [[surge-brand-authority-scan]]).
- Verify Bing index coverage of key pages.
- Lengthen and consolidate top pages into comprehensive single-source articles.

**Perplexity weakness:**
- Audit Reddit and forum presence for the brand and its topics.
- Update publication dates on key pages; refresh stale content.
- Encourage community discussion / AMAs / authentic forum participation.

**Gemini weakness:**
- Audit Google Knowledge Panel presence and Google Business Profile.
- Add Schema.org structured data to key pages.
- Build a YouTube channel with topic-aligned video content; add chapters/timestamps.

**Bing Copilot weakness:**
- Verify Bing Webmaster Tools is registered and the sitemap is submitted.
- Implement IndexNow for instant indexing of new and updated content.
- Strengthen LinkedIn company page; encourage employee thought leadership posting.

### 4. Prioritize across engines

Engines aren't equally valuable to every user. Prioritize fixes by the user's traffic mix:

- If most AI-referred traffic is ChatGPT, fix ChatGPT first.
- If the user is targeting Google's commercial-intent searchers, AIO is the highest-leverage engine.
- If the user's audience skews technical, Perplexity (Reddit + Stack Overflow weights) may over-index.

Don't default to "fix all engines equally." Most teams can sustain one focused engine push at a time.

## Decisions

| Situation | What to do |
|---|---|
| User on Trial plan | Only ChatGPT and Google AI Overview are tracked. The other engines won't appear in the per-engine breakdown — skip them in the prescription unless the user explicitly asks about future plan tiers. |
| Brand is strong on one engine, weak on another | This is engine asymmetry. Focus on what's different about the weak engine — index, source preferences, content format — rather than trying to make every engine equal. |
| User wants alerts when an engine's rank drops | SurgeGraph doesn't push alerts. Schedule [[surge-weekly-visibility-review]] for recurring monitoring instead. |
| User asks for the single most important engine | No universal answer. Best proxy: which engine drives the most referral traffic to their site today. If unknown, ChatGPT has the largest active user base in 2026 and is often the right first focus. |
| User asks about Apple Intelligence, Claude.ai web search, etc. | These aren't in SurgeGraph's tracked engine set yet. Note this; the optimization principles for "canonical entity recognition + structured data" apply across most LLM-powered search surfaces. |

## Common Issues

- **All engines look equally bad** — the project may be too new to have meaningful per-engine signal. Wait 2-3 more refresh cycles.
- **One engine dominates and others are sparse** — this is normal early on; brands often start with strong organic Google presence (driving AIO) and lower visibility on community-driven engines (Perplexity, ChatGPT). The action items address exactly this.
- **Engine source patterns are out of date** — these patterns change. If a recent change is reported in the press, update this skill's source-selection field guide.

## References

- [[surge-brand-authority-scan]] — directly addresses the "authority gap" diagnosis for ChatGPT and Gemini.
- [[surge-ai-crawler-audit]] — directly addresses the "index gap" diagnosis.
- [[surge-llmstxt-builder]] — supports cross-engine entity recognition by exposing structured site context.
- [[surge-weekly-visibility-review]] — recurring measurement to confirm per-engine fixes are working.
- [[surge-optimize-content]] — page-level edits backing the per-engine prescriptions.
