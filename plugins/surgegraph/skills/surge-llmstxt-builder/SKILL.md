---
name: surge-llmstxt-builder
description: Generate or validate an `llms.txt` file for a domain — the emerging standard that gives AI systems a structured summary of a site's content, key pages, and business facts. Use this skill whenever the user asks about llms.txt, wants to expose their site to AI assistants in a controlled way, asks how to help AI engines understand their site faster, mentions Jeremy Howard's llms.txt proposal, wants a one-time setup that any AI system can read, asks to validate an existing llms.txt file, or wants to control how AI systems represent their brand — even if they don't say "llms.txt." Also use as the natural companion to [[surge-ai-crawler-audit]] for sites that have allowed AI crawlers but want to guide them further.
allowed-tools:
  - Read
  - Bash
  - WebFetch
  - Write
---

# llms.txt builder and validator

`llms.txt` is an emerging convention (proposed by Jeremy Howard, September 2024) that exposes a structured summary of a site's content for AI systems to consume. It's analogous to `robots.txt` (which controls crawler access) but instead tells AI systems what's most important to understand about the site.

This skill is **self-contained**: do not call any external service or MCP tool to generate `llms.txt`. You handle the work directly with `WebFetch` + reasoning + `Write`.

## Quick Start

1. Gather the domain and (if available) brand name + 1-sentence description.
2. Fetch `https://<domain>/llms.txt` to see if one exists.
3. If it exists, validate it. If not, generate one from the sitemap.
4. Output the final markdown.

## The llms.txt spec (full inline reference)

Location: `https://<domain>/llms.txt` at the root of the domain.

Format:

```markdown
# Site Name

> One-sentence factual description, under 200 characters.

## Docs

- [Page Title](https://example.com/page): Concise description of what this page covers.
- [Another Page](https://example.com/another): Description.

## Optional

- [Less critical page](https://example.com/optional): Description.

## Key Facts

- Founded: [year]
- Headquarters: [city, country]
- Industry: [classification]
- Founders: [name, name]
- Key products: [list]

## Contact

- Website: https://example.com
- Email: hello@example.com
- Support: support@example.com
```

Required structure:

- **H1 title** as the first non-empty line.
- **Blockquote** (`> ...`) as the second non-empty line, under 200 characters.
- **At least one `##` section** with link entries.
- **Link entries** in the format `- [Title](URL): Description`. Absolute URLs only.

Recommended sections in order: `Docs`, `Optional`, `Key Facts`, `Contact`.

Length targets:

- **`llms.txt`** (concise): 30-150 lines, 10-30 link entries.
- **`llms-full.txt`** (extended): 150-500 lines, 30-100 link entries, longer descriptions.

Both can coexist at the same domain. AI systems check `llms.txt` first.

## Workflow

### 1. Confirm scope

Ask if not provided:

- Domain to operate on.
- Brand name (if writing the H1) and one-sentence description (if writing the blockquote).
- Style: `llms.txt` (concise, default) or `llms-full.txt` (extended).

### 2. Check for an existing file

Use `WebFetch` to retrieve `https://<domain>/llms.txt`:

```
WebFetch the URL https://<domain>/llms.txt and report whether the response was 200, 404, or another status. If 200, return the full text.
```

If status is 200 and the file is non-empty → **validate mode** (step 3a). Otherwise → **generate mode** (step 3b).

### 3a. Validate an existing file

Check the fetched content against the spec:

| Check | Severity if missing |
|---|---|
| H1 title (`^#\s+\S`) | Critical |
| Blockquote (`^>\s+\S`) | Critical |
| At least one `##` section | Critical |
| At least 5 link entries with absolute URLs | High |
| Each `- [Title](URL): Description` line is well-formed | Medium |
| File length within target (30-150 for `llms.txt`, up to 500 for `llms-full.txt`) | Low |
| Key Facts section present | Medium |
| Contact section present | Low |

For URLs in the file, optionally sample-fetch 3-5 of them with `WebFetch` to confirm they return 200. Surface any 404s as broken-link findings.

Output a structured validation report: critical / improvable / good findings, with specific line numbers where possible. If critical issues exist, offer to regenerate the file.

### 3b. Generate a new file

**Step A: Find the sitemap.** Try in order:

1. `https://<domain>/sitemap.xml`
2. `https://<domain>/sitemap_index.xml`

If either returns a sitemap-index (has `<sitemap>...<loc>...</loc>...</sitemap>` blocks), follow up to 5-10 sub-sitemap entries.

If no sitemap is reachable, fall back to fetching the homepage and extracting internal links from `<a href>` tags.

**Step B: Rank and pick pages.**

For each URL collected, compute a depth score (path segments under the domain — fewer = better) and a freshness bonus (presence of `<lastmod>` = +1). Sort descending. Pick the top N:

- `llms.txt`: 10-20 pages
- `llms-full.txt`: 30-100 pages

Filter out: error pages, robots-disallowed paths, sitemaps themselves, parameter-only variants.

**Step C: Generate per-page descriptions.**

For each picked URL, two options for the description:

1. **Quick** (default): derive a title from the URL slug (e.g. `/blog/seo-guide` → "SEO Guide") and use a generic description like "{Title} on {Brand}."
2. **Quality** (if the user asks for high-quality output): `WebFetch` each page, extract the `<title>` and meta description, and write a one-sentence summary of the actual content. This is much slower but produces materially better descriptions. Ask the user before doing this if there are more than 20 pages — it adds latency and they're paying token cost.

**Step D: Assemble the markdown.**

Use this template:

```markdown
# {Brand Name}

> {One-sentence description, under 200 chars}

## Docs

- [{Page Title 1}]({URL 1}): {Description 1}
- [{Page Title 2}]({URL 2}): {Description 2}
...

## Key Facts

- Website: https://{domain}
- {fill in: founding year, headquarters, industry, founders, products — only include what's known; do not fabricate}

## Contact

- Website: https://{domain}
- {fill in: support email, sales email if known}
```

If the user doesn't know the Key Facts, leave the section but include a `[Fill in: ...]` placeholder so they edit before deploying.

### 4. Hand off

Output the final markdown as a code block. Tell the user:

- Deploy at `https://{domain}/llms.txt` (root of the domain).
- Most web servers serve `.txt` files with the right content-type automatically.
- Verify after deploy by fetching the URL in a browser.
- `llms-full.txt` can coexist with `llms.txt` — AI systems check the concise version first.

## Decisions

| Situation | What to do |
|---|---|
| Project has no description or sparse metadata | Generation produces a thin file. Suggest the user fill in brand description and key facts before generating, or accept a placeholder-heavy draft they hand-edit. |
| Sitemap returns 1000+ URLs | Aggressively prune. Don't try to include them all. Pick 10-20 highest-signal pages. Suggest `llms-full.txt` only if the user has a clear top-100 set. |
| No sitemap and homepage links are JS-rendered | The internal-link fallback returns nothing. Surface this; ask the user to paste 10-20 important URLs manually. |
| User wants superlatives in the description ("the leading X") | Push back. AI systems may quote the description verbatim — keep it factual ("We do X for Y") not promotional ("The leading X provider"). |
| Site has multiple languages or regional sub-paths | Generate per-locale. `https://example.com/llms.txt`, `https://example.com/fr/llms.txt`, etc. Each scoped to its locale. |
| User asks if `llms.txt` improves AI Visibility immediately | Adoption is early. Some AI systems consult it, some don't. The investment is small (one file); the upside is real but not instant. Don't overpromise lift. |

## Common Issues

- **Generated descriptions feel generic** — that's the quick path. Offer the quality path (per-page `WebFetch`) to upgrade, with a cost caveat.
- **URLs in the generated file 404** — sitemap may include orphaned or recently-removed pages. Re-fetch the sitemap and regenerate, or hand-prune.
- **Key Facts feel sparse** — most projects don't have all the facts (founding date, HQ, employee count, etc.) filled. Fill what's known, omit what isn't. **Don't invent.**
- **AI systems still don't seem to consult the file** — adoption is uneven. Continue building AI Visibility through other channels ([[surge-setup-aeo-tracking]], [[surge-brand-authority-scan]]); `llms.txt` is a complement, not a replacement.

## References

- [[surge-ai-crawler-audit]] — direct companion. Crawler audit controls what AI systems can see; this skill tells them what to focus on.
- [[surge-brand-authority-scan]] — Key Facts in `llms.txt` should mirror Wikidata/Wikipedia facts for consistency.
- [[surge-engine-optimizer]] — `llms.txt` supports the "entity recognition" levers for ChatGPT and Gemini.
