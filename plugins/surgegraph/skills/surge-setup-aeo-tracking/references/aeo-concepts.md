# AEO concepts: topics, prompts, engines

A 60-second primer for skills working with SurgeGraph AI Visibility data.

## Entities and how they relate

```
Project
  └── Topics (categories the brand cares about)
        └── Prompts (specific questions sent to AI engines)
              └── Prompt responses (per-engine answers, with citations)
```

- **Project** — one brand + one primary domain + a locale (location + language). Bootstrapped with four default answer engines enabled. A user/org can own many projects.
- **Topic** — a thematic grouping ("CRM software," "freelance invoicing tools"). Auto-generated from domain research, or user-added. Topics are the unit users select when starting prompt generation.
- **Prompt** — a single question text that gets sent to AI engines on a recurring cadence ("What's the best CRM for solopreneurs?"). Prompts live under one topic. New prompts default to `isTracking: false` — tracking is opt-in.
- **Engine** — an AI answer engine. Default set: ChatGPT, Google AI Overview, Perplexity, Gemini. Bing Copilot may be available depending on plan.
- **Prompt response** — the actual answer one engine produced for one prompt at one timestamp. Contains citations (which URLs the engine referenced), sentiment, position, etc.

## What gets tracked, when

- A prompt with `isTracking: true` is dispatched to each enabled engine on a **plan-gated refresh cadence** (not universally daily).
- Each dispatch produces one `prompt_response` per engine.
- Aggregations (overview, trend, sentiment, citations, traffic) are computed from `prompt_response` rows.

## Plan-gated behavior

- **Trial accounts** only run ChatGPT and Google AI Overview, regardless of which engines are enabled on the project.
- **Refresh cadence** depends on plan tier. Never claim a universal "daily" cadence to users.
- **Prompt count** is capped per plan. Hitting the cap blocks new prompt creation; existing prompts continue to refresh on their cadence.

## Topic vs. "topic research"

These are **different concepts** that share a word:

- **AEO topic** (this document) — categories under a project, used to group prompts.
- **Topic research** — a separate per-query keyword/content research feature (`create_topic_research`, `list_topic_researches`). Output is keyword-volume + content-gap data, **not** prompts. Cacheable for 30 days.

Don't conflate them. If the user says "topic," ask clarifying questions when ambiguous.
