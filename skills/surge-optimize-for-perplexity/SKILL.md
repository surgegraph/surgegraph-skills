---
name: surge-optimize-for-perplexity
description: Deep-dive optimization for Perplexity AI specifically — covers Reddit and forum presence (Perplexity's strongest signal), content freshness, original-data positioning, quotable-passage structure, and YouTube transcripts. Use this skill whenever the user wants to optimize specifically for Perplexity, asks why Perplexity does not cite their brand, mentions Reddit being important for AI search, wants to win on community-driven AI engines, asks about Perplexity Pages, or wants engine-specific Perplexity tactics. Also use after [[surge-engine-optimizer]] identifies Perplexity as the priority engine.
allowed-tools:
  - Read
  - Bash
  - WebFetch
  - Write
---

# Optimize for Perplexity

Perplexity is the AI search engine most aggressive about community-validated sources. It cites Reddit, Hacker News, Stack Exchange, and forum threads at a rate other engines don't approach. It also drives meaningful referral traffic because it always attributes sources prominently.

This skill is self-contained: apply the rubric using `WebFetch` for site audits. No backend tool calls.

## How Perplexity selects sources

1. **Reddit dominates.** Industry studies (Profound 2025) put Reddit at ~45-47% of Perplexity citations. The community-validation thesis is central — Perplexity prefers claims that have been debated, validated, or expanded by multiple participants.
2. **Multiple sources per answer.** Perplexity typically cites 5-15 sources per response (more than ChatGPT or AIO). This creates room for mid-authority sites that wouldn't win on other engines.
3. **Freshness matters more.** Publication and last-updated dates are stronger ranking signals on Perplexity than elsewhere. Stale content is deprioritized aggressively.
4. **Original data wins.** Surveys, benchmarks, case studies, proprietary analysis — Perplexity favors primary sources over synthesis pieces.
5. **Own crawling infrastructure.** Perplexity runs PerplexityBot rather than relying purely on third-party indexes. Allowing PerplexityBot is non-negotiable.

## Audit workflow

### 1. Confirm the target

- Brand name + primary domain.
- Topics the brand wants to win on Perplexity.

### 2. Run the per-area audit

| # | Area | Weight | What to check |
|---|---|---|---|
| 1 | Reddit presence | 20 | Active discussion in relevant subreddits. Sentiment. Volume of mentions. Official account. |
| 2 | Forum / community presence | 10 | Hacker News, Stack Overflow, Quora, industry-specific forums. |
| 3 | Content freshness | 10 | What % of key pages updated in the last 6 months? Stale → low score. |
| 4 | Original research / data | 15 | Original surveys, benchmarks, datasets, case studies with named outcomes? |
| 5 | YouTube content with transcripts | 10 | Active channel with descriptive titles and (auto-generated) transcripts. |
| 6 | Quotable, standalone paragraphs | 10 | Paragraphs that make sense extracted in isolation. Self-contained, fact-rich. |
| 7 | Multi-source claim validation | 10 | When the brand makes a claim, do other (independent) sources confirm it? |
| 8 | Discussion-generating content | 10 | Opinion pieces, contrarian takes, original data — content that gets shared and debated. |
| 9 | Wikipedia/Wikidata presence | 5 | Bonus — Perplexity uses these but less than ChatGPT does. |

### 3. Sub-rubric for Reddit presence (20 pts)

| Score | Criteria |
|---|---|
| 18-20 | Brand frequently recommended in relevant subreddits, predominantly positive sentiment, active and authentic official account, own subreddit with 5K+ members, top recommendation for industry queries. |
| 14-17 | Brand regularly mentioned across multiple subreddits, mostly positive sentiment, some official presence, appears in recommendation threads. |
| 10-13 | Brand mentioned in several relevant threads, mixed sentiment, recognized by community. |
| 5-9 | Occasional Reddit mentions, limited to 1-2 subreddits, no official presence. |
| 0-4 | Rare or no Reddit mentions. Brand is largely unknown on Reddit. |

### 4. Action plan by score range

**Score 80-100 (excellent):**
Maintain authentic engagement (the moment Reddit detects astroturfing, citation health degrades). Continue publishing original research quarterly. Refresh top pages every 3-6 months.

**Score 60-79 (good):**
Targeted gaps. Most common: strong Reddit presence but stale content (publish dates >12 months) or thin original-research output. Adding one original survey or benchmark dataset per quarter often closes the gap.

**Score 40-59 (mixed):**
Two structural gaps. Either Reddit presence is weak OR original-data content is missing. Build whichever is more achievable first — Reddit takes 6-12 months of authentic engagement, original research can ship in 4-8 weeks.

**Score 0-39 (heavy lift):**
No Reddit presence, no original data, possibly content freshness issues. Perplexity is unlikely to cite this brand. Year-long effort:
1. Pick 3-5 relevant subreddits. Have a senior team member participate authentically — answer questions, contribute helpfully, no promotional posts.
2. Publish one piece of original research within 60 days (survey, benchmark, dataset).
3. Refresh top 10 pages with updated dates and current statistics.
4. Verify PerplexityBot is allowed in robots.txt.

## Decisions

| Situation | What to do |
|---|---|
| Brand can't authentically participate on Reddit (regulated industry, etc.) | Pivot to Hacker News, Stack Overflow (if technical), Quora, or industry-specific forums. Perplexity weights these too — Reddit is just the largest. |
| Reddit sentiment is negative | Do NOT remove or astroturf. Engage authentically — acknowledge issues, address them publicly, build trust over time. Astroturfing detection has improved and Perplexity has explicitly flagged this. |
| User asks about Perplexity Pages | Perplexity Pages are curated summary pages Perplexity creates about topics. Check if Perplexity has created one about the user's brand or topics. Influence by being a frequently-cited source on the topic — there's no direct submission flow. |
| Content freshness audit shows mostly old pages | Don't just bump dates ("last updated: today" with no actual updates). Perplexity (and other engines) detect mismatch between dated content and actual content age. Do real refreshes — update stats, revise outdated claims, add new sections. |
| PerplexityBot blocked in robots.txt | Foundational fix — see [[surge-ai-crawler-audit]]. Until PerplexityBot is allowed, no Perplexity optimization matters. |

## Common Issues

- **High Reddit volume but low Perplexity citations** — usually sentiment or relevance issue. Perplexity prefers positive-sentiment recommendation threads over neutral mentions. Audit which threads are being cited and which aren't.
- **Original research published but no Perplexity uptake** — distribution problem. Original data needs to be discovered. Pitch the research to industry newsletters, post on Reddit (where allowed), get coverage in pubs Perplexity already cites.
- **Content is fresh but Perplexity still cites older competitor content** — the competitor likely has more multi-source validation. Quality wins over recency when validation is unequal.
- **YouTube channel exists but no Perplexity citations** — transcripts must be enabled and titles/descriptions must contain the target keywords. Auto-generated transcripts work; manual ones are better.

## References

- [[surge-engine-optimizer]] — cross-engine view that routes here when Perplexity is the priority.
- [[surge-brand-authority-scan]] — Reddit and YouTube presence audits.
- [[surge-ai-crawler-audit]] — PerplexityBot must be allowed.
- [[surge-page-citability-score]] — for quotable-passage audit.
