---
name: surge-knowledge-library-bootstrap
description: Create a SurgeGraph knowledge library and ingest a set of source documents (URLs, files, brand guidelines, internal docs) so the Writer can ground future articles in real brand context instead of generic prose. Use this skill whenever the user mentions setting up a knowledge base, wants to feed brand context to Writer, says the AI outputs sound generic or off-brand, wants to ingest their docs for the writer to reference, asks how to make Writer use their brand voice, wants to upload reference material, or mentions Writer needing context — even if they don't say "knowledge library." Also use as a prerequisite step before generating long-form articles via [[surge-topic-research-to-article]] or [[surge-opportunities-to-content]].
---

# Bootstrap a knowledge library

A knowledge library is a project-scoped collection of source documents that Writer uses as grounding context. Without one, Writer falls back to generic LLM prose. With one, Writer can cite the user's own documentation, brand voice, product specs, customer research, etc.

This skill creates a library and ingests N source documents in one pass.

## Quick Start

1. Confirm the project ID and the library's purpose (brand voice, product docs, customer research, etc.).
2. Collect the source documents — URLs, file paths, or pasted text.
3. Create the library.
4. Add each source as a knowledge library document.
5. Verify ingestion succeeded.

## Workflow

### 1. Confirm scope

Ask:

- Project ID.
- Library name — short, descriptive. Examples: "Brand voice," "Product specs Q2 2026," "Customer research interviews."
- Library description — one sentence explaining what's inside. This goes into Writer's context when the library is attached, so it influences how Writer uses the material.
- Sources — list of URLs, file paths, or pasted content.

### 2. Validate sources

For each source:

- **URLs** — confirm they're publicly accessible (not behind auth). The ingestion fetches the page; auth walls fail silently.
- **Files** — confirm size and format. Common formats supported: markdown, HTML, plain text, PDF. Bytes per document have a plan-gated limit.
- **Pasted text** — works for ad-hoc additions; less reusable than URLs.

Drop or flag any source that fails validation before proceeding. Don't try to ingest a 404 or a binary the system doesn't support.

### 3. Create the library

Create the knowledge library with the name and description. Capture the returned `libraryId`.

### 4. Add documents one by one

For each validated source, add it as a knowledge library document. Surface the count as you progress for libraries with >5 sources.

If any document fails to add, continue with the rest — don't abort the whole batch. Surface failures at the end with a list.

### 5. Verify

List the library's documents. Confirm count matches expected. Surface to the user:

- Library ID and name.
- Document count added vs. attempted.
- Any failures with reasons.

## Decisions

| Situation | What to do |
|---|---|
| User wants one library per topic | Don't. Knowledge libraries scope to the project, and Writer attaches one library at a time. One library per *purpose* (brand voice, product, research) is the right granularity, not one per topic. |
| User wants to ingest a competitor's site | Allowed — useful for "write like X" reference material. But surface that the library will influence Writer's output direction; the user should label it clearly (e.g. "competitor reference — not our voice"). |
| Source is a PDF over the size limit | Surface the limit. Suggest splitting the PDF or extracting the most relevant sections as plain text. |
| User mentions wanting to update sources later | Adding new documents to an existing library is supported. Replacing requires deleting and re-adding (or creating a versioned library: "Brand voice v2"). |
| User asks about ingestion latency | Add is typically fast. Embedding and indexing happen in the background; documents are available to Writer once indexing completes. |
| Source is behind a login | Won't work for URL ingestion. Suggest the user paste the content directly or export to PDF/markdown first. |

## Common Issues

- **Document ingestion succeeded but content looks empty in Writer's context** — embedding may not have completed yet. Wait a few minutes; if still empty, the source may have been HTML with no extractable body text (e.g. JS-rendered single-page app).
- **Adding a document returns 403** — the URL is blocked by the source site's bot policy. Suggest pasting the content directly or hosting a markdown copy.
- **Writer still produces generic prose after attaching the library** — confirm the library is *attached* to the document generation request, not just created. Library attachment is a per-generation parameter.
- **Library exceeds plan limit** — knowledge libraries are plan-gated by document count and storage. Surface the limit and suggest pruning or upgrading.
- **Duplicate documents** — adding the same URL twice creates two records. Skill should de-dupe by source URL before adding, but the backend doesn't enforce uniqueness.

## References

- [[surge-topic-research-to-article]] — primary consumer of knowledge libraries.
- [[surge-opportunities-to-content]] — bulk Writer generation benefits significantly from a library attached.
- [[surge-optimize-content]] — rewrite quality improves with brand voice library attached.
