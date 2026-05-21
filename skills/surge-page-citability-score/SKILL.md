---
name: surge-page-citability-score
description: Score a web page on how extractable its content is for AI search engines, with concrete rewrite suggestions per content block. Use this skill whenever the user wants to know how citable a page is, asks why AI engines aren't quoting their content, wants to improve a page for AI extractability, asks for a citability or extractability score, mentions making content more quotable, wants concrete rewrites to land in AI answers, or asks "is this content AI-ready" — even if they don't say "citability." Also use as a fast pre-publish check on any article before it goes live.
allowed-tools:
  - Read
  - Bash
  - WebFetch
  - Write
---

# Page citability score

This skill produces a 0-100 score for how extractable a page's content is to AI search engines, broken into sub-scores across five dimensions, with specific rewrite suggestions targeting the lowest-scoring blocks.

It's a content-quality skill that complements [[surge-optimize-content]]: that skill uses tracking data for already-tracked topics; this skill works on **any** page or document, even before tracking starts.

This skill is **self-contained**: fetch the page yourself via `WebFetch` and apply the rubric below. Do not call any external service or MCP tool. Your judgment as an LLM is the scoring mechanism — that's a feature, not a bug. Subjective dimensions like "uniqueness" and "answer quality" are evaluated more accurately by an LLM reading the content than by regex heuristics.

## The five-dimension citability rubric

Score each dimension 0-100, then compute a weighted composite.

### 1. Answer-block quality — 30% weight

Do sections open with direct, quotable answers?

| Score | Criteria |
|---|---|
| 90-100 | Every major section opens with a 1-2 sentence direct answer. Uses "X is..." or "X refers to..." patterns. First 40-60 words can stand alone as a complete answer. |
| 70-89 | Most sections have clear answer openings. Some definition patterns. |
| 50-69 | Some sections have answer-like openings but many bury the answer mid-paragraph. |
| 30-49 | Answers are generally buried. No consistent definition patterns. Narrative-driven. |
| 0-29 | No identifiable answer blocks. Entirely narrative or conversational. |

Signals to look for: definition patterns ("X is..."), quantified answers ("The average is $4,500"), comparison openings, declarative statements within the first sentence of each section.

### 2. Passage self-containment — 25% weight

Can paragraphs be extracted in isolation and still make sense?

| Score | Criteria |
|---|---|
| 90-100 | 80%+ of paragraphs are self-contained. Each names its subject explicitly. No reliance on pronouns referencing earlier content. Contains specific facts within the passage. |
| 70-89 | 60-79% are self-contained. Occasional pronoun references requiring context. |
| 50-69 | 40-59% are self-contained. Mixed pronoun use. |
| 30-49 | 20-39% are self-contained. Heavy pronoun reliance. |
| 0-29 | Under 20%. Continuous narrative where any extracted paragraph loses meaning. |

Per-paragraph checklist: names the subject (not "it," "this," "they"); can be understood reading only this passage; contains a specific fact, statistic, or named entity; avoids opening with "But," "However," "And."

### 3. Structural readability — 20% weight

Headings, paragraphs, lists, tables — the formatting AI systems extract from.

| Score | Criteria |
|---|---|
| 90-100 | Clean H1>H2>H3 hierarchy. Question-based headings. Short paragraphs (2-4 sentences). Tables for comparisons. Ordered lists for processes. |
| 70-89 | Good hierarchy with minor skips. Some question-based headings. Mostly short paragraphs. |
| 50-69 | Hierarchy inconsistent. Few question-based headings. Mix of short and long paragraphs. |
| 30-49 | Minimal headings. No question-based headings. Long paragraphs dominate. |
| 0-29 | No heading structure. Wall-of-text paragraphs. |

### 4. Statistical density — 15% weight

Specific numbers, dates, named studies vs. vague quantifiers.

| Score | Criteria |
|---|---|
| 90-100 | 5+ specific statistics per 500 words. Claims backed by named sources. Exact numbers (percentages, dollar amounts, timeframes, named studies). |
| 70-89 | 3-4 statistics per 500 words. Most claims sourced. |
| 50-69 | 1-2 statistics per 500 words. Mix of specific and vague. |
| 30-49 | <1 statistic per 500 words. Predominantly vague ("many," "most," "some"). |
| 0-29 | No statistics. All vague quantifiers. |

Counts: specific percentages, dollar amounts, named studies, year references, specific counts. Doesn't count: "many companies," "a significant percentage," "experts agree."

### 5. Uniqueness — 10% weight

Original data, first-hand insights, methodology vs. derivative restatement.

| Score | Criteria |
|---|---|
| 90-100 | Original research, proprietary data, surveys, or unique datasets. Analysis not found elsewhere. Methodology described. |
| 70-89 | Some original insights or unique analysis. Distinct perspective with original examples. |
| 50-69 | Mostly synthesizes existing info but adds unique commentary or examples. |
| 30-49 | Largely derivative. Restates common knowledge. |
| 0-29 | Entirely derivative. All info available verbatim on higher-authority sources. |

Signals: "Our analysis of...", "We surveyed N professionals...", first-hand case studies, custom data visualizations, original frameworks, methodology descriptions.

## Workflow

### 1. Identify the target

Accept any of:

- A URL — public, no auth wall.
- Pasted HTML or markdown.

### 2. Fetch the page

Use `WebFetch` to retrieve the URL:

```
WebFetch <URL> and return the main content. Strip navigation, header, footer, and sidebar; preserve headings and paragraph structure.
```

If the page is JS-rendered (returns minimal HTML), surface this and ask the user to paste the rendered content.

### 3. Segment into blocks

Split the content into blocks at each H2 (or H3 if H2 is sparse). For each block, note:

- Heading text
- Word count
- Paragraph count
- Lists / tables present
- Number of statistics
- Whether opening sentence forms a standalone answer

### 4. Score each dimension

Apply the rubric above. Use your judgment as an LLM — that's the entire point of doing this in a skill rather than a backend tool. Be calibrated: don't give a 95 unless the content actually meets the 90-100 criteria.

### 5. Compute the composite

```
Composite = (Answer * 0.30) + (SelfContainment * 0.25) + (Readability * 0.20) + (StatDensity * 0.15) + (Uniqueness * 0.10)
```

Round to the nearest integer.

### 6. Identify the lowest-scoring blocks

For the 3-5 lowest-scoring sections, produce concrete rewrite suggestions:

- "Open with a direct answer" → propose the exact opening sentence.
- "Add named entities" → suggest 2-3 specific names/products/dates.
- "Convert paragraph to list" → produce the list version.
- "Add statistic" → suggest a verifiable stat (flag if the user needs to source it; **don't fabricate**).

### 7. Output

```markdown
# Citability score for {URL}

**Composite: {N}/100** — {interpretation}

## Sub-scores

| Dimension | Score | Weight | Notes |
|---|---|---|---|
| Answer-block quality | {N} | 30% | {1-2 line note} |
| Self-containment | {N} | 25% | ... |
| Structural readability | {N} | 20% | ... |
| Statistical density | {N} | 15% | ... |
| Uniqueness | {N} | 10% | ... |

## Lowest-scoring blocks

### Block 1: "{Heading}"
Score: {N}/100. Why: {1-2 sentence diagnosis}.

**Suggested rewrite:**
{Concrete proposed text}

[repeat for 2-3 more blocks]

## Recommended actions
- {priority 1}
- {priority 2}
- {priority 3}
```

## Composite score interpretation

| Range | Interpretation |
|---|---|
| 80-100 | Strong. Minor improvements only. |
| 60-79 | Good baseline. Targeted rewrites can lift this. |
| 40-59 | Mixed. Several blocks need restructuring. |
| 0-39 | Heavy lift. The page buries answers, lacks structure, or is too generic. |

Don't fixate on the composite. Sub-scores tell the actual story — a 65 with `uniqueness: 30` is different from a 65 with `self-containment: 30`.

## Decisions

| Situation | What to do |
|---|---|
| Page is gated / requires auth | `WebFetch` will fail. Ask the user to paste the HTML/markdown. |
| Page is JS-rendered with empty initial HTML | Surface this; ask for pasted content. |
| User wants to score competitor pages | Allowed and useful for benchmarking. Surface that this scores extractability, not authority — a high-citability competitor may still not outrank for other reasons. |
| User asks for an "exact word count" target | Be careful. The general principle (short, self-contained, fact-rich passages extract better) is sound; specific "optimal word count" numbers in industry write-ups aren't consistently reproduced. Speak qualitatively. |
| User asks for a re-score after edits | Re-run the skill on the updated content. Note: LLM scoring has some run-to-run variance — 2-3 point differences aren't material; 5+ point movement is real. |
| Composite doesn't match the user's intuition | Trust the sub-scores. If a specific finding seems off, surface the underlying signal so they can judge. |

## Common Issues

- **Score feels too low for well-written content** — well-written for humans isn't the same as extractable. Flowing paragraphs that read beautifully often score low on self-containment and answer-block quality. The score isn't a judgment of writing quality.
- **Score feels too high for thin content** — keyword stuffing or lots of statistics on a thin topic can inflate sub-scores. Cross-check uniqueness — a low uniqueness on an otherwise-high page means the content is derivative even if structurally good.
- **Suggestions repeat across blocks** — usually means a systemic issue (every section starts with backstory before the answer). Surface the systemic issue once.

## References

- [[surge-optimize-content]] — full rewrite flow grounded in tracked-prompt data (vs. this skill's URL-only scope).
- [[surge-opportunities-to-content]] — write new articles at scale; this skill is for individual pages.
- [[surge-topic-research-to-article]] — pair to score the output of a fresh research-to-article run.
