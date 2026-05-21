---
name: surge-brand-authority-scan
description: Audit a brand's third-party platform presence — Wikipedia, Wikidata, Reddit, YouTube, LinkedIn — to find authority gaps that affect AI search citation. Use this skill whenever the user asks why ChatGPT or Gemini doesn't recognize their brand, wants to check if they have a Wikipedia or Wikidata entry, asks about brand entity recognition for AI, wants to know what third-party platforms they should establish presence on, mentions building topical authority for AI visibility, or asks why a competitor with worse content seems to win on ChatGPT — even if they don't say "authority" or "entity." Also use as a complement to [[surge-ai-crawler-audit]] for a foundational AI Visibility health check.
allowed-tools:
  - Read
  - Bash
  - WebFetch
  - Write
---

# Brand authority scan

AI search engines weight third-party authority signals heavily. Pages with rich content from a brand without a Wikipedia article, Wikidata entity, or community-platform presence often lose to thinner content from better-recognized brands.

This skill is **self-contained**: hit Wikipedia, Wikidata, and other public APIs directly via `WebFetch`. Do not call any external service or MCP tool.

## Platform weights for AI search citation

| Platform | AI engine it most affects | What it provides |
|---|---|---|
| **Wikipedia** | ChatGPT, Gemini, Perplexity | The strongest single signal. A canonical, accurate Wikipedia article significantly increases citation likelihood. |
| **Wikidata** | Gemini, ChatGPT | Machine-readable entity data AI engines consume for entity grounding. Often missing even when a Wikipedia article exists. |
| **Reddit** | Perplexity, ChatGPT | Community discussion. Perplexity cites Reddit heavily. |
| **YouTube** | Gemini, Perplexity | Video transcripts increasingly cited; Gemini cites YouTube more than any other AI engine. |
| **LinkedIn** | Bing Copilot, ChatGPT (B2B) | Professional authority signal. |

## Quick Start

1. Confirm the brand name (canonical form) and the primary domain.
2. Check Wikipedia + Wikidata via their public APIs.
3. Check Reddit, YouTube, LinkedIn via search.
4. Walk through the per-platform findings.
5. Produce a prioritized action plan based on which AI engines the user cares about.

## Workflow

### 1. Confirm scope

Ask if not provided:

- **Brand name** — exact canonical form ("Stripe," not "Stripe Inc.").
- **Primary domain** — used to confirm the entity matches the user's brand (vs. another brand with the same name).

### 2. Wikipedia check

Use `WebFetch` to call Wikipedia's open-search API:

```
WebFetch https://en.wikipedia.org/w/api.php?action=opensearch&format=json&limit=5&search=<BRAND_NAME>
```

Response is `[query, titles[], descriptions[], urls[]]`. Take the top title.

Then fetch the summary:

```
WebFetch https://en.wikipedia.org/api/rest_v1/page/summary/<TOP_TITLE>
```

Determine status:

- **Present and matched** — article exists and the summary mentions the user's domain or matches the brand's industry.
- **Present but unmatched** — article exists but the summary appears to be about a different entity with the same name.
- **Present but thin** — article exists with a short summary (<200 chars in `extract`). Stub-class article — weak signal.
- **Absent** — no article found.

### 3. Wikidata check

Use `WebFetch` to call Wikidata's search API:

```
WebFetch https://www.wikidata.org/w/api.php?action=wbsearchentities&search=<BRAND_NAME>&language=en&limit=5&format=json
```

Take the top entity ID (e.g. `Q12345`). Then fetch its full data:

```
WebFetch https://www.wikidata.org/wiki/Special:EntityData/<ENTITY_ID>.json
```

Look at the `claims` object. Key properties to check:

- `P856` — official website. If present, compare against the user's domain.
- `P571` — founding date.
- `P159` — headquarters location.
- `P112` — founder.
- `P452` — industry.

Determine status:

- **Present and matched** — entity exists, `P856` matches user's domain, ≥5 properties total.
- **Present but unmatched** — entity exists but `P856` is a different domain.
- **Present but thin** — entity exists but has fewer than 5 properties (sparse — weak signal).
- **Absent** — no entity found.

### 4. Reddit check

Use `WebFetch` to search Reddit:

```
WebFetch https://www.reddit.com/search.json?q=<BRAND_NAME>&limit=25
```

Parse the JSON `data.children` array. Count results by subreddit and tally sentiment (positive/negative keywords in titles and selftexts is a rough proxy).

Determine status:

- **Strong** — 10+ relevant threads across 3+ subreddits, predominantly positive sentiment.
- **Moderate** — 5-10 mentions, mixed sentiment.
- **Weak** — 1-4 mentions or all from a single subreddit.
- **Absent** — no relevant results.

If Reddit's API returns a 429 (rate-limited) or 403, fall back to suggesting the user run a manual `site:reddit.com "<brand>"` search and report findings.

### 5. YouTube check

YouTube doesn't have a public unauthenticated search API that returns useful results via simple WebFetch. Fall back to a structured manual check:

Ask the user to:

1. Search `<BRAND_NAME>` on YouTube.
2. Report:
   - Does the brand have an own channel? Subscriber count.
   - Are there third-party videos (reviews, tutorials, comparisons) discussing the brand? Roughly how many in the first page of results?

Record their answers as the YouTube finding.

### 6. LinkedIn check

LinkedIn doesn't have an unauthenticated public search API either. Fall back to a structured manual check:

Ask the user to:

1. Open `linkedin.com/company/<BRAND_SLUG>` (guess the slug from the brand name).
2. Report:
   - Does the company page exist?
   - Follower count and post recency.
   - Are leadership/employees posting thought leadership that mentions the brand?

Record their answers as the LinkedIn finding.

### 7. Compose the action plan

Prioritize by which AI engine the user most wants to win:

- **Targeting ChatGPT** → Wikipedia first, then Wikidata, then Reddit.
- **Targeting Gemini** → Wikidata + YouTube + Google Knowledge Panel.
- **Targeting Perplexity** → Reddit + freshness of content.
- **Targeting Bing Copilot** → LinkedIn + IndexNow (pair with [[surge-ai-crawler-audit]]).

For each gap, recommend the specific action:

- **No Wikipedia** + brand is notable → recommend establishing a Wikipedia article through a specialized editor service (don't write it yourself — Wikipedia has strict notability + conflict-of-interest rules).
- **No Wikidata** → recommend creating a Wikidata entity (looser inclusion criteria; can be done in ~1 hour with a Wikipedia account).
- **No Reddit presence** → recommend authentic community engagement (NOT astroturfing) — answer questions, host an AMA, participate in relevant subreddits.
- **No YouTube** → recommend creating topic-aligned videos with chapters, descriptive titles, detailed descriptions.
- **No LinkedIn** → recommend completing the company page, encouraging leadership thought leadership.

## Output format

```markdown
# Brand authority scan — {Brand} ({domain})

## Summary
- Wikipedia: {verdict}
- Wikidata: {verdict}
- Reddit: {verdict}
- YouTube: {verdict, user-reported}
- LinkedIn: {verdict, user-reported}

## Findings detail
{per-platform expanded notes}

## Prioritized action plan
1. {highest-impact action with engine impact}
2. ...
```

## Decisions

| Situation | What to do |
|---|---|
| Brand has no Wikipedia and is unlikely to meet notability criteria | Don't push Wikipedia. Wikidata is still viable (looser inclusion). LinkedIn, YouTube, Reddit are reliable alternative authority signals. |
| Brand shares a name with another entity (disambiguation page on Wikipedia) | Structural problem. Recommend a disambiguating Wikipedia article (if notable) or at minimum a Wikidata entity. |
| User asks for "Wikipedia article generation" service | Don't write one directly. Wikipedia has strict notability + conflict-of-interest rules. Refer to specialized editor services or ensure notability before drafting. |
| Reddit sentiment is negative | Authentic engagement is the answer, not manipulation. Surface the finding clearly; don't try to "fix" through astroturfing. |
| YouTube channel exists but content is sparse | Recommend topic-aligned videos with descriptive titles, structured chapters, detailed descriptions — transcripts feed AI engine indexes. |
| User wants alerts on entity changes | Not supported. Re-run this skill periodically. SurgeGraph (and this skill) does not push alerts. |
| Reddit API rate-limited | Fall back to manual `site:reddit.com "<brand>"` search; have the user paste findings. |

## Common Issues

- **Wikidata entity exists but isn't detected** — the entity may not have `P856` (official website) set, making it hard to confirm it's about the right brand. Surface the candidate ID and ask the user to verify.
- **Wikipedia article exists but brand is rarely cited by AI engines** — having the article is necessary, not sufficient. The article's content, citations, and sameAs network all matter. Stub articles often perform worse than no article.
- **Action items feel generic** — authority building is medium-term work. For platform-specific tactics, refer to specialized resources.
- **Third-party "brand mention" counts disagree** — different scopes. SurgeGraph's existing brand-mentions tool counts mentions inside AI responses; this skill measures presence on third-party platforms. They answer different questions.

## References

- [[surge-engine-optimizer]] — the "authority gap" branch routes here.
- [[surge-ai-crawler-audit]] — companion foundational diagnostic.
- [[surge-setup-aeo-tracking]] — run brand-authority-scan before tracking, to avoid measuring a brand that can't be cited regardless of content.
