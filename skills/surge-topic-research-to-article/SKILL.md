---
name: surge-topic-research-to-article
description: Research a topic deeply with SurgeGraph topic research, then turn the research into a Writer article — outline, sub-topics, entities, and intent clusters all flow into the brief. Use this skill whenever the user asks to write or draft an article about a specific topic, says "research and write," wants a long-form piece on X, asks for a deep dive, wants content on a named topic, or asks the writer to "make it thorough" or "make it rank" — even if they don't mention "research" first. Also use when the user already has a topic_research record and wants to convert it into an article without re-researching.
---

# Topic research → article

When a user names a topic and wants an article about it, this skill chains topic research (keywords, sub-topics, content gaps) into a Writer document brief, then generates the document. The result is grounded in real search data, not generic prose.

This is the user-driven counterpart to [[surge-opportunities-to-content]] (which is analytics-driven).

## Quick Start

1. Confirm the project ID and the topic to research.
2. Check for an existing topic research record for this topic; if present, reuse it (cached up to 30 days).
3. If absent, create a new topic research; wait for completion.
4. Optionally expand sub-topics for depth.
5. Derive a brief from the research output.
6. Generate the Writer document.

## Workflow

### 1. Confirm inputs

Ask if not provided:

- Project ID.
- Topic — a phrase, not a sentence ("solopreneur CRM," not "what's the best CRM for solopreneurs").
- Article angle — informational, comparison, how-to, listicle. Default: informational if user didn't say.
- Target length — short (~800 words), standard (~1,500), long-form (~3,000). Default: standard.

### 2. Check for existing topic research

List topic researches for the project, filter by topic name. Topic research is cacheable for 30 days; reuse a fresh record rather than re-researching.

If a fresh record exists, skip to step 4.

### 3. Create topic research

Trigger new topic research. This is async on the backend. Wait for completion before proceeding.

Polling: every 30 seconds, max 5 minutes. If still incomplete at 5 minutes, surface the wait and offer to resume.

### 4. Optionally expand sub-topics

If the user said "deep dive" or "make it thorough" or the target length is long-form, run topic research expansion on the top 2-3 sub-topics. This produces more granular keyword and intent data per sub-topic.

Skip for standard or short articles — adds latency and credit cost without proportional value.

### 5. Derive the brief

From the research output, extract:

- **Primary keywords** — top 5-10 from the research.
- **Sub-topics** — should become H2 sections of the article.
- **Search intent** — informational vs. commercial vs. transactional. Influences tone and CTAs.
- **Named entities** — people, companies, products to mention.
- **Content gaps** — angles competitors don't cover, surfaced by the research.

Format as a brief:

```
Title: [working title]
Target length: [N words]
Primary topic: [phrase]
Sub-topics: [H2 list]
Must-cover entities: [names]
Distinctive angle: [what this article does that others don't]
Tone: [user's brand voice — see knowledge library if attached]
```

### 6. Generate the Writer document

Create the document with the brief. If the project has a knowledge library, attach it as grounding ([[surge-knowledge-library-bootstrap]] if the project doesn't have one yet).

Return the document ID and a preview of the outline. Don't auto-publish — the user reviews first.

## Decisions

| Situation | What to do |
|---|---|
| Topic name is too broad ("marketing," "SaaS") | Surface this — research will return generic results. Ask the user to narrow ("marketing automation for SMBs," "SaaS pricing strategy"). |
| Topic name overlaps with an AEO topic the user already tracks | Mention the overlap. Suggest they may also want to run [[surge-optimize-content]] after publishing, since they'll have tracking data for related prompts. |
| User wants article in a language other than English | Confirm the project's language code matches. Topic research output is language-aware. Writer output follows the project's language. |
| Project has no knowledge library | Generation will work but quality is generic. Strongly suggest [[surge-knowledge-library-bootstrap]] before generating, especially for long-form. |
| User wants multiple articles from one research run | Use [[surge-opportunities-to-content]] for bulk generation, OR run this skill once per article. Don't try to multiplex in this skill — briefs should be article-specific. |
| Expansion would push the run past credit budget | Surface the credit cost and let the user decide. Expansion is optional. |

## Common Issues

- **Topic research returns thin keyword data** — the topic may be too niche or too new. Surface the result; the user may need to broaden the topic or write from first-hand expertise rather than search data.
- **Generated article reads generic despite research** — usually means no knowledge library was attached. Add one via [[surge-knowledge-library-bootstrap]] and regenerate.
- **Generated article duplicates an existing document** — Writer doesn't dedupe against past documents. Check the project's documents list before generating if duplication is a concern.
- **Topic research stuck in pending past 5 minutes** — the deep research backend may be overloaded. Wait the full 5 minutes; if still stuck, surface to support.
- **Sub-topic expansion produces irrelevant tangents** — expansion can drift on broad topics. For drift-prone topics, skip expansion.

## References

- [[surge-knowledge-library-bootstrap]] — strongly recommended before generating long-form articles.
- [[surge-opportunities-to-content]] — analytics-driven counterpart for bulk article production.
- [[surge-optimize-content]] — improve the article post-publication after a refresh cycle of tracking data.
- [[surge-publish-to-wordpress]] — ship the generated article to a connected WordPress site.
