---
name: surge-optimize-for-chatgpt
description: Deep-dive optimization for getting cited specifically on ChatGPT — covers Wikipedia/Wikidata entity strategy, Bing index coverage (ChatGPT uses Bing's index, not Google's), Reddit presence, comprehensive-article framing, and authoritative backlink patterns. Use this skill whenever the user wants to optimize specifically for ChatGPT, asks why ChatGPT does not cite their brand, mentions wanting to win on ChatGPT or OpenAI search, asks about Wikipedia or Bing for AI search, or wants engine-specific ChatGPT tactics — not the general cross-engine view. Also use after [[surge-engine-optimizer]] identifies ChatGPT as the priority engine.
allowed-tools:
  - Read
  - Bash
  - WebFetch
  - Write
---

# Optimize for ChatGPT

ChatGPT is the largest AI search surface (300M+ weekly active users as of 2025) and the engine most users default to when asking AI questions. It has distinct source-selection patterns that don't transfer cleanly from Google SEO — winning here requires a specific playbook.

This skill is self-contained: apply the rubric below using `WebFetch` for any site audits. No backend tool calls.

## How ChatGPT selects sources

Five facts that shape every optimization decision for ChatGPT:

1. **It's Bing-indexed, not Google-indexed.** ChatGPT (and OAI-SearchBot) use Bing as their foundation. A page that ranks #1 on Google but isn't in Bing is invisible to ChatGPT.
2. **Wikipedia is the #1 single source.** Multiple industry studies put Wikipedia at ~40-50% of ChatGPT citations. Brands without a Wikipedia article are at a structural disadvantage on entity-related queries.
3. **Reddit is second.** Around 10-15% of ChatGPT citations come from Reddit, especially for product comparisons, opinions, and "what do you recommend" queries.
4. **Comprehensive articles win.** ChatGPT prefers single authoritative 2000+ word sources over combining multiple thin pages on the same topic.
5. **Entity grounding matters.** ChatGPT uses Wikipedia, Wikidata, Crunchbase, and LinkedIn to confirm "is this a real entity worth citing." Brands that exist as structured entities are cited more.

## Audit workflow

### 1. Confirm the target

- Brand name (canonical form).
- Primary domain.
- Key topics the brand wants to be cited on.

### 2. Run the per-area audit

For each of the 9 areas below, score the brand 0-100 using the rubric, then weight per the points column. Total composite = ChatGPT optimization score (0-100).

| # | Area | Weight (pts) | What to check |
|---|---|---|---|
| 1 | Wikipedia presence | 20 | Does an accurate, non-stub article exist? See sub-rubric. |
| 2 | Wikidata entity | 10 | Does an entity with ≥5 properties exist? P856 (official website) matches domain. |
| 3 | Bing index coverage | 10 | Sample 10 key pages — are they in Bing? Use `site:domain.com` on Bing. |
| 4 | Reddit brand mentions | 10 | Search `site:reddit.com "<brand>"` — count threads, sentiment, official account. |
| 5 | YouTube presence | 10 | Active channel with topic-aligned content + third-party videos mentioning the brand. |
| 6 | Authoritative backlinks | 15 | Sample: do they have .edu, .gov, major-publication backlinks? |
| 7 | Entity consistency | 10 | Brand name, founding date, HQ, leadership — consistent across Wikipedia, Wikidata, Crunchbase, LinkedIn, own site? |
| 8 | Content comprehensiveness | 10 | Top pages 2000+ words? Or thin pages on same topic? |
| 9 | Crawler access | 5 | GPTBot, OAI-SearchBot, ChatGPT-User all `Allow: /` in robots.txt? |

### 3. Sub-rubric for Wikipedia (20 pts)

| Score | Criteria |
|---|---|
| 18-20 | Detailed Wikipedia article (B-class+), 5+ external references, brand domain cited as a source, founder/leadership pages exist. |
| 14-17 | Wikipedia article exists (start-class), 2-4 external references. |
| 10-13 | Stub-class article exists. Improving the article is the single highest-leverage action. |
| 5-9 | No article, but brand is mentioned in other articles. Recommend establishing notability and drafting. |
| 0-4 | No Wikipedia or Wikidata presence. |

### 4. Action plan by score range

**Score 80-100 (excellent):**
Maintain. Monitor Wikipedia article quality monthly (edits can degrade it). Refresh content on top pages every 6 months for freshness signal.

**Score 60-79 (good):**
Targeted gaps. Most common: Wikipedia article is a stub (improve depth + sources), or Bing index coverage is incomplete (submit sitemap to Bing Webmaster Tools).

**Score 40-59 (mixed):**
Two of three structural gaps: missing Wikipedia, weak entity consistency across platforms, sparse Reddit presence. Address Wikipedia + entity consistency first — both compound.

**Score 0-39 (heavy lift):**
No Wikipedia, no Wikidata, sparse Reddit, possibly Bing index gaps. ChatGPT is unlikely to cite this brand at all. Year-long entity-building effort:
1. Establish Bing index coverage (Bing Webmaster Tools + sitemap).
2. Create Wikidata entity (looser inclusion criteria than Wikipedia).
3. Build Reddit presence through authentic engagement (no astroturfing).
4. Earn 3-5 authoritative backlinks (industry pubs, .edu).
5. Once notable, draft Wikipedia article via a specialized editor.

## Decisions

| Situation | What to do |
|---|---|
| Brand fails Wikipedia notability bar | Skip Wikipedia; focus on Wikidata (which has looser criteria) + Reddit + LinkedIn + authoritative-publication mentions. ChatGPT still uses these signals even without a Wikipedia article. |
| Bing Webmaster Tools shows partial index coverage | Submit sitemap, fix any crawl errors, request indexing on missing pages. Index coverage improvements take 2-4 weeks to reflect in ChatGPT citations. |
| Reddit sentiment is mixed/negative | Authentic engagement is the answer (answer questions in relevant subreddits, host AMAs, contribute helpfully). Do NOT astroturf — ChatGPT increasingly detects manipulated discussion patterns. |
| GPTBot is blocked in robots.txt | Foundational fix — see [[surge-ai-crawler-audit]]. Until GPTBot is unblocked, no other optimization matters. |
| User asks about prompt engineering for ChatGPT | Out of scope — this skill is about getting the brand cited by ChatGPT, not about prompting ChatGPT effectively. |

## Common Issues

- **High Google rankings but low ChatGPT visibility** — almost always a Bing index problem. Check `site:domain.com` on Bing first.
- **Strong Wikipedia article but still not cited** — content comprehensiveness gap. ChatGPT prefers long-form (2000+ words) on the user's own site. Audit top pages.
- **One competitor dominates ChatGPT despite having worse content** — usually entity grounding (Wikipedia + Wikidata + sameAs network). Authority compounds; one good Wikipedia article beats 50 better landing pages.
- **Ranking on ChatGPT but losing to a competitor on a specific prompt** — drill into the prompt with [[surge-citation-momentum-tracker]] to see what URL is winning, then study that URL's structure.

## References

- [[surge-engine-optimizer]] — cross-engine view that routes here when ChatGPT is the priority.
- [[surge-brand-authority-scan]] — Wikipedia/Wikidata gap audit drives most of the action items here.
- [[surge-ai-crawler-audit]] — GPTBot must be allowed before any of this matters.
- [[surge-page-citability-score]] — for comprehensiveness audit on individual pages.
- [[surge-optimize-for-bing-copilot]] — shares the Bing index foundation; some actions help both engines.
