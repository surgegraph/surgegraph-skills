# SurgeGraph Skills

**23 ready-to-use workflows for AI Visibility, content production, and agency operations — installable as a Claude Code plugin.**

These skills wrap the SurgeGraph product into focused workflows your Claude Code agent can route to automatically. Ask "how's my brand doing on AI search this week" and Claude picks `surge-weekly-visibility-review`. Ask "why isn't ChatGPT citing us" and it picks `surge-optimize-for-chatgpt`. No commands to memorize.

> **Requires a paid SurgeGraph subscription.** Most skills call the SurgeGraph API via the `surgegraph` CLI or MCP server. Sign up at [surgegraph.io](https://surgegraph.io).

## Install

```
/plugin marketplace add surgegraph/surgegraph-skills
/plugin install surgegraph@surgegraph-skills
```

Update later with `/plugin marketplace update`.

## What's included

### Setup & foundations
- `surge-setup-aeo-tracking` — end-to-end onboarding for AI Visibility tracking
- `surge-knowledge-library-bootstrap` — ingest brand docs so Writer grounds in real context
- `surge-ai-crawler-audit` — audit robots.txt for AI crawlers (GPTBot, ClaudeBot, etc.)
- `surge-llmstxt-builder` — generate or validate an `llms.txt` for AI assistants

### Monitoring & reviews
- `surge-weekly-visibility-review` — week-over-week AI Visibility report
- `surge-citation-momentum-tracker` — pages gaining/losing AI citations
- `surge-emerging-topics-monitor` — new topics AI engines are answering
- `surge-cross-client-anomaly-detector` — pre-filtered portfolio movements for agencies
- `surge-multi-client-portfolio-rollup` — agency portfolio view

### Content production
- `surge-topic-research-to-article` — research → Writer brief → article
- `surge-opportunities-to-content` — turn citation gaps into a content plan
- `surge-optimize-content` — AEO improvement recommendations for existing articles
- `surge-generate-image` — brand-aligned imagery for articles
- `surge-publish-to-wordpress` — push Writer docs to WordPress

### Diagnostics & analysis
- `surge-page-citability-score` — per-block extractability score with rewrites
- `surge-brand-authority-scan` — Wikipedia/Wikidata/Reddit/YouTube/LinkedIn gap analysis
- `surge-competitor-citation-analysis` — where competitors win AI citations you don't
- `surge-engine-optimizer` — diagnose under-citation per AI engine

### Per-engine deep-dives
- `surge-optimize-for-chatgpt`
- `surge-optimize-for-aio` (Google AI Overviews)
- `surge-optimize-for-perplexity`
- `surge-optimize-for-gemini`
- `surge-optimize-for-bing-copilot`

## How skills work

Each skill is a `SKILL.md` file with YAML frontmatter (`name`, `description`, optional `allowed-tools`). Claude Code reads the `description` field to decide when to invoke the skill — you don't run a command, you just describe what you want.

The skills assume you have either:
- The `surgegraph` CLI installed and authenticated (`surgegraph auth login`), or
- The SurgeGraph MCP server connected (via Claude Desktop or any MCP-capable client)

Skills that don't depend on the SurgeGraph API (e.g. `surge-ai-crawler-audit`, `surge-llmstxt-builder`) work standalone with just `Read`/`Bash`/`WebFetch`/`Write`.

## License

Apache-2.0 — see [LICENSE](./LICENSE).
