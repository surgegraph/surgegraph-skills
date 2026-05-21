---
name: surge-optimize-for-gemini
description: Deep-dive optimization for Google Gemini specifically — covers Google Knowledge Panel claiming, Google Business Profile, YouTube content strategy (Gemini cites YouTube more than any other engine), Schema.org markup depth, and the Google ecosystem integrations. Use this skill whenever the user wants to optimize specifically for Gemini, asks why Gemini does not surface their brand, mentions Google Knowledge Panel or Knowledge Graph for AI search, wants to leverage YouTube for AI visibility, or asks about Schema.org and AI search. Also use after [[surge-engine-optimizer]] identifies Gemini as the priority engine.
allowed-tools:
  - Read
  - Bash
  - WebFetch
  - Write
---

# Optimize for Google Gemini

Gemini sits on top of Google's full search index plus the Google ecosystem (Knowledge Graph, Business Profile, YouTube, Merchant Center). It cites more from Google-owned properties than any other AI engine. If a brand has invested in Google's ecosystem already, Gemini optimization is mostly about completing what's started.

This skill is self-contained: apply the rubric using `WebFetch` for any site audits. No backend tool calls.

## How Gemini selects sources

1. **Google ecosystem first.** Gemini draws from Knowledge Graph, Knowledge Panels, Business Profiles, YouTube transcripts, Maps, Scholar, and News. A brand's presence in the Google ecosystem matters more for Gemini than for any other engine.
2. **YouTube is weighted heavily.** Multiple industry studies (Ahrefs Brand Radar 2025) put YouTube transcripts at the strongest single non-Google-property signal for Gemini, with a correlation around 0.7 — higher than any other surface.
3. **Schema.org gets consumed directly.** Gemini reads structured data (JSON-LD) for entity understanding more aggressively than other engines. A page with rich `Organization`, `Article`, `Person`, `Product` markup is materially advantaged.
4. **Multi-modal.** Gemini references images and videos alongside text in the same answer. Image alt text and video transcripts contribute to citation probability.
5. **E-E-A-T weighted strongly.** Google's E-E-A-T framework (Experience, Expertise, Authoritativeness, Trustworthiness) applies to Gemini outputs with extra weight — Gemini inherits Google's quality-rater guidelines wholesale.

## Audit workflow

### 1. Confirm the target

- Brand name + primary domain.
- Topics the brand wants to win on Gemini.

### 2. Run the per-area audit

| # | Area | Weight | What to check |
|---|---|---|---|
| 1 | Google Knowledge Panel | 15 | Does one exist for the brand? Is it complete and accurate? |
| 2 | Google Business Profile | 10 | Fully completed (hours, services, photos, posts, Q&A)? Local businesses especially. |
| 3 | YouTube channel + topic content | 20 | Active channel with topic-aligned videos, chapters/timestamps, detailed descriptions. |
| 4 | Schema.org structured data | 15 | Organization, Article, Author, Product schemas on key pages. JSON-LD format. |
| 5 | Google ecosystem presence | 10 | Google Scholar (research), Google News (publishers), Google Maps (local), Google Workspace Marketplace (if applicable). |
| 6 | Image optimization | 10 | Alt text, descriptive filenames, high-quality images on key pages. |
| 7 | E-E-A-T signals | 10 | Author pages with credentials, About page with team + history, editorial standards, corrections policy. |
| 8 | Google Merchant Center (e-commerce only) | 5 | Products feed into Merchant Center with full attribute coverage. |
| 9 | Multi-modal content (text + image + video) | 5 | Key topics covered across multiple media types, not just text. |

### 3. Sub-rubric for YouTube channel (20 pts)

| Score | Criteria |
|---|---|
| 18-20 | Active channel with regular uploads, topic-aligned content, chapters/timestamps in descriptions, transcripts enabled, third-party videos also discuss the brand. |
| 14-17 | Active channel with topic content. Some videos have chapters; transcripts enabled. |
| 10-13 | Channel exists, some content, but inconsistent uploads or thin coverage of key topics. |
| 5-9 | Channel exists but inactive or off-topic. |
| 0-4 | No YouTube channel or empty channel. Gemini has nothing to surface from your own video presence. |

### 4. Action plan by score range

**Score 80-100 (excellent):**
Maintain. Publish 1-2 videos per month covering tracked topics. Refresh Knowledge Panel quarterly via Google Business Profile updates.

**Score 60-79 (good):**
Targeted gaps. Most common: strong Knowledge Panel + GBP but YouTube is thin. Plan 6-12 videos covering top tracked prompts — one video per major topic, with chapter timestamps and detailed descriptions.

**Score 40-59 (mixed):**
Multiple gaps. Knowledge Panel may be incomplete or non-existent, YouTube is sparse, Schema.org is thin. Prioritize: (1) claim/build Knowledge Panel via GBP, (2) implement Organization + Article schemas across top 20 pages, (3) start YouTube channel with first 6 topic videos.

**Score 0-39 (heavy lift):**
Brand has minimal Google ecosystem presence. Year-long buildout:
1. Set up Google Business Profile (even for non-local businesses — claim it).
2. Implement Schema.org markup site-wide (Organization, key pages with appropriate schemas).
3. Launch YouTube channel with 12 topic-aligned videos in first 6 months.
4. Build E-E-A-T signals: author pages with credentials, About page with team, editorial policy.

## Decisions

| Situation | What to do |
|---|---|
| Brand has no physical location (SaaS, online-only) | Google Business Profile still applies — claim it for service-area businesses. Less leverage than for local-business brands, but still establishes Knowledge Graph entity. |
| User asks about FAQPage rich results | Google restricted FAQ rich results in 2023 except for govt/health sites. The content pattern still helps Gemini (FAQ structure aids extraction). Implement the FAQ section; don't expect rich-result display. |
| Brand can't sustain a YouTube channel | Skip multi-video production. Focus on Schema.org + Knowledge Panel + GBP — these get most of the way without video. Plan one strong "channel trailer + 3 cornerstone topic videos" as a minimum viable presence. |
| Knowledge Panel exists but shows wrong/outdated info | Suggest edits through GBP or via Google's "Suggest edit" flow on the Knowledge Panel itself. Verification + propagation takes 2-8 weeks. |
| Schema.org validates but Gemini still doesn't cite | Schema is necessary but not sufficient. Gemini also needs strong E-E-A-T signals — author credentials, editorial policy, About page depth. Audit those next. |
| Google-Extended is blocked in robots.txt | Affects Gemini training but not Google Search rankings. Gemini may still cite based on standard search indexing, but features that draw on training data are reduced. Allow unless the user has a specific data-licensing concern — see [[surge-ai-crawler-audit]]. |

## Common Issues

- **Brand is heavily cited on ChatGPT but invisible on Gemini** — almost always means the brand has built Bing/Reddit/Wikipedia presence (good for ChatGPT) but not Google ecosystem presence (needed for Gemini). YouTube + Schema.org + GBP buildout closes this gap.
- **Schema.org implemented but JSON-LD doesn't validate** — use Google's Rich Results Test or Schema.org's validator. Common errors: missing required fields, wrong nesting, conflicting properties.
- **YouTube channel exists with 50+ videos but no Gemini citations** — usually transcripts aren't enabled or video titles don't match query phrasings. Enable auto-transcripts on all videos; restructure titles to mirror "People Also Ask" patterns.
- **Brand is well-cited but rarely appears in Gemini's image responses** — alt text + filename hygiene. Audit images on key pages — descriptive alt text and `kebab-case-keyword-filenames.jpg`, not `IMG_3829.jpg`.

## References

- [[surge-engine-optimizer]] — cross-engine view that routes here when Gemini is the priority.
- [[surge-brand-authority-scan]] — Wikidata feeds Knowledge Graph and matters here too.
- [[surge-ai-crawler-audit]] — Google-Extended decision belongs in this conversation.
- [[surge-optimize-for-aio]] — adjacent engine; Schema.org + E-E-A-T work helps both.
- [[surge-llmstxt-builder]] — entity recognition through llms.txt complements the Google ecosystem signals.
