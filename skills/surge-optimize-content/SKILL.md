---
name: surge-optimize-content
description: Take an existing article and produce concrete AEO improvement recommendations grounded in SurgeGraph's response-structure, citation, and topic-gap data — then optionally generate an optimized version. Use this skill whenever the user wants to improve an article for AI search, asks why a page isn't getting cited by ChatGPT or Google AI Overviews, wants to rewrite content to rank better in AI answers, asks for AEO suggestions on a specific document, mentions citability or extractability of a passage, wants to make content more quotable by AI, or asks what to do about a page that's losing AI traffic — even if they don't say "AEO" or "optimize." Also use when the user pastes a URL or document ID and asks "how can this do better."
---

# Optimize a document for AI Engine Optimization

This skill turns SurgeGraph's analytical surface (citations, topic gaps, response structure, sentiment) into actionable edit suggestions for one article at a time. Output is either a list of recommendations, an optimized document, or both.

## Quick Start

1. Identify the document — by ID, URL, or by listing recent documents.
2. Identify the document's primary topic and the prompts that target it.
3. Pull the diagnostic data: response structure for tracked prompts, topic-gap analysis, citation data.
4. Produce a prioritized list of edit suggestions.
5. If the user wants, generate an optimized version of the document.

## Workflow

### 1. Locate the document

- If the user gave a document ID, fetch it.
- If they gave a URL, list documents and match by URL.
- Otherwise, list recent documents and ask.

Capture: title, primary topic, current body, the prompts (if any) that the document was written for.

### 2. Find the tracked prompts that target this document's topic

Look up the project's AI Visibility prompts, filtered by the document's topic. These are the queries this article is competing on.

If the topic isn't tracked yet, surface that clearly — diagnostic data won't exist. Suggest [[surge-setup-aeo-tracking]] to add prompts for this topic first, then come back.

### 3. Pull diagnostic data

For each relevant tracked prompt:

- **Response structure** — how AI engines structure their answer for this prompt (list vs. paragraph vs. table; word counts of cited passages; named entities).
- **Citation data** — which domains AI engines cite. Note the user's own domain rank.
- **Topic gaps** — sub-topics that AI engines reference in answers but that the document doesn't cover.
- **Prompt response samples** — actual AI answers, with citations highlighted.

### 4. Produce edit suggestions

Translate the data into concrete recommendations. Prioritize by potential impact:

- **Structural fixes** — add a direct answer in the first 1-2 sentences of each H2; convert dense paragraphs to bullet lists where AI engines prefer lists for this prompt; add a table if response-structure data shows AI engines cite tabular sources.
- **Content gaps** — sub-topics the topic-gap tool flagged as missing. Each becomes a new H2 or H3 section with suggested word count.
- **Entity coverage** — named entities that competitor citations mention but this article doesn't.
- **Citability passages** — specific paragraphs to rewrite for easier extraction. Self-contained, fact-rich, answer-first.
- **Citation hooks** — internal links to authoritative pages, structured data, author bylines if missing.

Group suggestions by section of the document so the user can apply them in order.

### 5. Optionally generate an optimized version

If the user asks for an actual rewrite — not just suggestions — generate an optimized document. This creates a new SurgeGraph document linked to the original, with the changes applied. The original is untouched.

Don't auto-generate by default. Confirm with the user first; rewrites consume Writer credits.

## Decisions

| Situation | What to do |
|---|---|
| Document's topic is not tracked | Skill can't produce data-grounded suggestions. Run [[surge-setup-aeo-tracking]] to add tracking for this topic, wait one refresh cycle, then retry. Don't fabricate suggestions from intuition. |
| User wants to optimize in place vs. as a new doc | Default to a new optimized document (preserves history). Only do an in-place rewrite if the user explicitly asks. |
| Topic-gap tool returns nothing | Either the topic has few gaps (good — surface that as a positive) or the topic has insufficient tracking history. Check the refresh cadence on the user's plan. |
| User pastes a competitor URL | This skill operates on SurgeGraph documents only, not arbitrary URLs. Tell the user; suggest creating a Writer document with similar content if they want to optimize against this baseline. |
| Suggestions contradict the user's editorial voice | Surface the suggestion but flag the trade-off. The user has final say on tone; data-grounded structural advice (lists, answer-first paragraphs) is independent of voice. |

## Common Issues

- **"No prompts target this topic"** — the document was written for a topic SurgeGraph isn't tracking. See decision row above.
- **All suggestions are about structure, none about content** — usually means the article covers the topic well but isn't formatted for extraction. That's actionable on its own.
- **Optimized document is missing sections from the original** — verify the model output; the Writer may have collapsed sections it judged redundant. The user can ask for a re-run with a "preserve all sections" instruction.
- **Citation rank is 0 even after applying suggestions** — wait at least 2 refresh cycles before judging impact. Refresh cadence is plan-gated; ask the user when their plan refreshes if uncertain.

## References

- [[surge-setup-aeo-tracking]] — prerequisite for diagnostic data.
- [[surge-weekly-visibility-review]] — measure the impact of optimizations after a refresh cycle.
- [[surge-opportunities-to-content]] — for content that should exist but doesn't (vs. content that exists but underperforms).
