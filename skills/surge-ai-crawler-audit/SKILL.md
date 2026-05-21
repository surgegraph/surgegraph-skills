---
name: surge-ai-crawler-audit
description: Audit a site's robots.txt for blocked AI crawlers (GPTBot, ClaudeBot, PerplexityBot, Google-Extended, etc.) and produce a corrected robots.txt that allows the right ones. Use this skill whenever the user asks if their site can be crawled by AI bots, mentions GPTBot or ClaudeBot or Perplexity being blocked, wants to check what AI crawlers can access their content, asks why their content isn't showing up in AI answers, asks about robots.txt and AI search, or wants to maximize AI visibility through crawler access — even if they don't say "robots.txt" or "crawler." Also use as a foundational diagnostic step before more complex AI Visibility work, because blocked crawlers make all other optimization moot.
allowed-tools:
  - Read
  - Bash
  - WebFetch
  - Write
---

# AI crawler access audit

If AI crawlers are blocked in `robots.txt`, the site's content cannot appear in AI-generated responses regardless of how well-optimized the content is. This is the most fundamental — and easiest to miss — gate on AI Visibility.

Many sites inherit overly aggressive robots.txt rules from legacy SEO configurations and inadvertently block AI crawlers without realizing it.

This skill is **self-contained**: handle the fetch, parsing, and corrected-file generation directly with `WebFetch` + reasoning. Do not call any external service or MCP tool.

## The 13 AI crawlers to check

Group by impact tier. Blocking tier 1 crawlers makes the site invisible to that AI engine; blocking tier 2 reduces broader ecosystem coverage; tier 3 is training-only.

### Tier 1 — live AI search visibility (must allow)

| User-agent | Platform | Notes |
|---|---|---|
| `GPTBot` | ChatGPT (OpenAI) | Powers ChatGPT web browsing and search. Highest-impact crawler. |
| `OAI-SearchBot` | ChatGPT Search | Search-only, not used for training. No reason to block. |
| `ChatGPT-User` | ChatGPT user-initiated | Fetches pages when a user explicitly visits a URL via ChatGPT. |
| `ClaudeBot` | Anthropic Claude | Powers Claude web search, citations, and analysis. |
| `PerplexityBot` | Perplexity AI | Powers Perplexity. Best AI referral traffic among the engines. |

### Tier 2 — broader AI ecosystem (likely should allow)

| User-agent | Platform | Notes |
|---|---|---|
| `Google-Extended` | Gemini training / AI Overviews | Does NOT affect Google Search rankings. Only Gemini + AIO. |
| `GoogleOther` | Google AI research | Non-search-ranking AI research crawls. |
| `Applebot-Extended` | Apple Intelligence | Affects Apple Intelligence features on Apple devices. |
| `Amazonbot` | Alexa / Amazon AI | Powers Alexa answers. |
| `FacebookBot` | Meta AI | Meta AI across Facebook, Instagram, WhatsApp. |

### Tier 3 — training-only (context-dependent)

| User-agent | Platform | Notes |
|---|---|---|
| `CCBot` | Common Crawl | Many AI models train on Common Crawl datasets. No live AI search impact. |
| `anthropic-ai` | Anthropic training | Separate from ClaudeBot. Training only. |
| `Bytespider` | ByteDance / TikTok / Doubao | Aggressive crawler. Block for most Western sites. |

## Workflow

### 1. Confirm the domain

Single domain per run. Ask the user if not provided.

### 2. Fetch the robots.txt

Use `WebFetch` to retrieve `https://<domain>/robots.txt`:

```
WebFetch https://<domain>/robots.txt and return the full text. If status is 404, report that no robots.txt exists.
```

Possible outcomes:

- **200**: parse the body (step 3).
- **404**: no robots.txt → default is "all crawlers allowed by wildcard." Skip to step 4 and generate a new file.
- **Other**: surface the error and stop.

### 3. Parse the robots.txt

The format is line-based with `User-agent:`, `Allow:`, and `Disallow:` directives grouped by user-agent block.

For each of the 13 AI crawlers above, determine status by checking:

1. **Specific block exists** (e.g. `User-agent: GPTBot` followed by rules)
   - `Disallow: /` → **blocked**
   - `Allow: /` or any non-blocking rules only → **allowed**
2. **No specific block, but wildcard exists** (`User-agent: *`)
   - Wildcard `Disallow: /` → **blocked**
   - Wildcard allows → **allowed**
3. **Neither specific nor wildcard rule** → **not_specified** (defaults to allow at the protocol level but explicit is better)

### 4. Generate the per-crawler verdict

For each of the 13 crawlers, output:

- User-agent name
- Platform / engine
- Tier (1, 2, or 3)
- Status (`allowed`, `blocked`, or `not_specified`)
- Recommendation:
  - Tier 1 blocked → "BLOCKED — high-impact fix. Allow this crawler immediately to be visible in this AI engine."
  - Tier 1 not specified → "Not specified — relies on wildcard default. Add an explicit Allow rule for clarity."
  - Tier 2 blocked → "BLOCKED — likely should allow unless intentional. Reduces presence in broader AI ecosystem."
  - Tier 3 blocked → "Blocked — context-dependent. Training-only crawler, no live AI search impact."
  - Allowed → "Allowed."

### 5. Summary verdict

- `critical` if any tier-1 crawler is blocked.
- `improvable` if no tier-1 is blocked but any tier-2 is blocked.
- `healthy` if no tier-1 or tier-2 crawlers are blocked.

### 6. Produce a corrected robots.txt

Generate a clean file the user can drop in:

```
# AI crawlers — allow for AI search visibility
User-agent: GPTBot
Allow: /

User-agent: OAI-SearchBot
Allow: /

User-agent: ChatGPT-User
Allow: /

User-agent: ClaudeBot
Allow: /

User-agent: PerplexityBot
Allow: /

User-agent: Google-Extended
Allow: /

User-agent: GoogleOther
Allow: /

User-agent: Applebot-Extended
Allow: /

User-agent: Amazonbot
Allow: /

User-agent: FacebookBot
Allow: /

# Training-only crawlers — set per your data licensing stance
User-agent: CCBot
Allow: /

User-agent: anthropic-ai
Allow: /

User-agent: Bytespider
Disallow: /
```

Preserve the user's existing non-AI rules (anything not matching the 13 crawlers). Append them after the AI crawler block.

## Decisions

| Situation | What to do |
|---|---|
| `robots.txt` does not exist | Treat as "all crawlers allowed by default." Recommend creating one with explicit Allow rules — documents the intent and prevents future regressions. |
| robots.txt has `User-agent: *` `Disallow: /` | Catastrophic. Flag as the highest-priority fix. Confirm with the user — this is sometimes a misconfigured CDN or staging site. |
| Google-Extended is blocked | Explain it does NOT affect Google Search rankings — only Gemini training and AI Overviews. The user may have blocked it deliberately for data-licensing reasons. Don't change without consent. |
| User asks if blocking crawlers protects from training | Partial protection at best. Tier 1 crawlers are not training-only; blocking them costs visibility without meaningful training protection. Tier 3 (CCBot, anthropic-ai) are training-only — block those if data control is the goal. |
| Bytespider is blocked | Often deliberate (aggressive crawler, low Western-market benefit). Confirm context before changing. |
| User wants rate limits per-crawler | robots.txt doesn't support rate limits uniformly. Suggest Cloudflare or edge rules. |

## Common Issues

- **Tool reports "blocked" but the user thinks they allow everything** — `User-agent: *` `Disallow: /private/` only blocks `/private/`. Walk through specific paths with the user.
- **CDN-level robots.txt overrides** — Cloudflare, Fastly, etc. may inject their own robots.txt. Always fetch via a fresh HTTP client to see what crawlers actually see.
- **Staging or sub-domain robots.txt** — `app.example.com` has its own robots.txt separate from `example.com`. Audit each domain the user wants AI Visibility on.
- **User updates robots.txt but doesn't see AI Visibility changes** — robots.txt is cached by crawlers (typically 24h). Wait at least a day after changes before re-running visibility audits.

## References

- [[surge-engine-optimizer]] — crawler audit is the diagnostic that drives the "index gap" branch.
- [[surge-setup-aeo-tracking]] — run this skill before setting up tracking, to avoid measuring blocked content.
- [[surge-brand-authority-scan]] — pair for a foundational AI Visibility health check.
- [[surge-llmstxt-builder]] — `llms.txt` complements `robots.txt`.

