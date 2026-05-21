---
name: surge-optimize-for-aio
description: Deep-dive optimization for Google AI Overviews (AIO) specifically — covers organic top-10 ranking (which AIO heavily depends on), question-based headings, direct answers, tables, FAQ sections, and the featured-snippet overlap. Use this skill whenever the user wants to optimize for Google AI Overviews specifically, asks about AIO citations, mentions featured snippets and AI search overlap, asks how to get into Google's AI search results, wants to optimize for the boxes at the top of Google searches, or wants engine-specific Google AI tactics. Also use after [[surge-engine-optimizer]] identifies AIO as the priority engine.
allowed-tools:
  - Read
  - Bash
  - WebFetch
  - Write
---

# Optimize for Google AI Overviews (AIO)

Google AI Overviews is the answer panel that appears above traditional search results for many queries. It draws from Google's existing search index but applies a distinct selection layer — pages that rank well organically don't always make it into AIO, and pages outside the top 10 occasionally do.

This skill is self-contained: apply the rubric using `WebFetch` to audit pages. No backend tool calls.

## How AIO selects sources

1. **Organic top-10 is the gateway, but not sufficient.** Industry studies (Seer Interactive 2025) show roughly 38% of AIO citations come from pages ranking outside the organic top 10 — down from 76% earlier. So strong SEO is necessary but doesn't guarantee inclusion.
2. **Featured-snippet overlap is ~70%.** A page that wins a featured snippet for a query usually also wins the AIO citation. Optimize for both as one effort.
3. **AIO favors clean structure.** Tables, ordered lists, question-based headings, and direct answer paragraphs all extract reliably.
4. **Hedging hurts.** Sentences like "It depends on various factors" or "Generally speaking" reduce citation probability. AIO prefers declarative, factual statements.
5. **Mobile-first indexing.** AIO follows Google's mobile-first indexing. A page that renders poorly on mobile is invisible to AIO.

## Audit workflow

### 1. Confirm the target

- Domain.
- Tracked prompts the user wants to win on AIO.

### 2. Run the per-area audit

| # | Area | Weight | What to check |
|---|---|---|---|
| 1 | Organic top-10 rank for tracked prompts | 20 | Sample 10 prompts — what % rank top 10 organically? |
| 2 | Question-based H2/H3 headings | 10 | Check if headings phrase the prompt as a question. Mirror Google "People Also Ask." |
| 3 | Direct answer in first 1-2 sentences | 15 | After each question heading, is there an immediate factual answer? Or backstory first? |
| 4 | Tables for comparisons | 10 | Pricing, specs, feature comparisons — rendered as `<table>` HTML, not as bullet lists or paragraphs? |
| 5 | Ordered/unordered lists | 10 | Step-by-step content uses ordered lists. Feature catalogs use unordered lists. |
| 6 | FAQ section | 10 | At least 5 questions per key page in a structured FAQ section. |
| 7 | Statistics with cited sources | 10 | "73% of marketers report…" with named source. Avoid vague quantifiers. |
| 8 | Publication + last-updated dates | 5 | Visible on the page. AIO deprioritizes undated content for time-sensitive queries. |
| 9 | Author byline + credentials | 5 | Visible byline linked to an author page with bio, credentials, and `sameAs` links. |
| 10 | URL + heading hierarchy | 5 | Clean URLs (no session IDs), H1>H2>H3 hierarchy without skips. |

### 3. Sub-rubric for organic top-10 rank (20 pts)

| Score | Criteria |
|---|---|
| 18-20 | ≥80% of tracked-prompt queries rank in Google's organic top 10. |
| 14-17 | 60-79% rank top 10. Tactical SEO fixes will close most of the gap. |
| 10-13 | 30-59% rank top 10. Real SEO work needed before AIO becomes viable. |
| 5-9 | <30% rank top 10. Major content + authority work needed first. |
| 0-4 | Few or no tracked prompts rank. AIO is downstream of SEO that doesn't yet exist. |

### 4. Action plan by score range

**Score 80-100 (excellent):**
Pages already structured for AIO. Maintain freshness (re-publish key pages with updated stats every 3-6 months). Monitor featured-snippet wins as proxies for AIO citation health.

**Score 60-79 (good):**
Targeted gaps. Most common: pages have direct answers but bury them mid-paragraph instead of opening with them. Or comparison pages use bullet lists where tables would extract more reliably.

**Score 40-59 (mixed):**
Structural rewrite needed. Start with the top 10 pages by traffic — restructure each with question headings, direct-answer openings, and at least one table or list. Expect 2-3 refresh cycles before AIO citations shift.

**Score 0-39 (heavy lift):**
SEO foundation is missing. Top 10 rankings are sparse. AIO optimization is premature until the brand wins organic top 10 for the queries it wants to be cited on. Pivot to SEO basics first (content quality, internal linking, authoritative backlinks). Re-audit AIO readiness after 6 months.

## Decisions

| Situation | What to do |
|---|---|
| Brand wins featured snippets but not AIO citations | Unusual — typically these correlate. May be a query-intent mismatch (the brand's content answers a related-but-not-identical query). Audit the specific prompts where it loses. |
| Tables look good in the editor but render as bullet lists in production | CMS or theme issue. Inspect the HTML — AIO extracts from `<table>` markup, not from styled lists that look like tables. |
| User wants FAQPage schema | Google deprecated FAQ rich results for most sites in 2023 (still allowed for govt/health), but the FAQ *content pattern* still helps AIO extraction. Add the FAQ section; don't worry about the schema for rich results. |
| Brand is ranking top 10 but losing AIO citations to a lower-ranked competitor | Almost always a structure issue. The competitor's page has a direct answer in the first sentence; the brand's buries it. |
| User asks about ranking tracking | Out of scope. Use third-party SEO tools (Ahrefs, Semrush, SE Ranking) for organic rank tracking; this skill audits page-level AIO readiness. |

## Common Issues

- **Direct answer but no AIO citation** — the answer is correct but uses passive voice or hedging. "It is generally considered that…" loses to "X is Y." Rewrite for active, declarative voice.
- **Strong on desktop, weak on mobile** — Google indexes mobile-first. A page that renders perfectly on desktop but breaks on mobile is invisible to AIO. Use Google's Mobile-Friendly Test.
- **All pages have FAQ sections but still no AIO traction** — the FAQ questions don't match real user queries. Pull from Google's "People Also Ask" for the target topic and mirror those phrasings verbatim.
- **AIO citation appears one week, gone the next** — AIO selection is volatile, more than organic search. Don't over-react to one-day disappearances; wait 2-3 refresh cycles before judging movement. Single-day disappearances are usually sampling variance.

## References

- [[surge-engine-optimizer]] — cross-engine view that routes here when AIO is the priority.
- [[surge-page-citability-score]] — page-level audit drives most of the action items here.
- [[surge-ai-crawler-audit]] — Google-Extended doesn't affect Google Search but does affect AIO inclusion for some queries.
- [[surge-optimize-for-gemini]] — adjacent engine; some actions (Schema.org, E-E-A-T) help both.
