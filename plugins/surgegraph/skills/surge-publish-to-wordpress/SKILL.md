---
name: surge-publish-to-wordpress
description: Publish a SurgeGraph Writer document to a connected WordPress site — pick the document, verify the CMS integration is connected, choose categories and an author, then send. Use this skill whenever the user wants to publish, push, ship, post, send, or move a document/article/draft to WordPress, asks how to get a SurgeGraph article live, wants to schedule a post, asks about WordPress integration with SurgeGraph, or mentions publishing AI-written content to their site — even if they don't say "WordPress" or "CMS" explicitly. Also use when the user has just finished optimizing or generating content and the implied next step is publication.
---

# Publish a document to WordPress

WordPress is the only CMS SurgeGraph publishes to today (Shopify, Webflow, and Wix are not yet integrated). Use this skill to take a Writer document or optimized document from "draft on SurgeGraph" to "live on the user's WordPress site."

## Quick Start

1. Identify the document to publish — by ID if known, otherwise by listing documents and asking the user.
2. Verify the project has a CMS integration connected. If not, connect one.
3. Fetch the available WordPress categories and authors for the integration.
4. Confirm category selection, author, and publish status (draft vs. published vs. scheduled) with the user.
5. Publish.

## Workflow

### 1. Locate the document

If the user gave a document ID, use it. Otherwise:

- List the project's Writer documents and optimized documents.
- Ask the user which one to publish. Sort by most recently updated first.
- Confirm the title and last-updated time before proceeding.

### 2. Verify CMS integration

Fetch the project's CMS integration metadata. Three possible states:

- **Connected to a WordPress integration** — proceed.
- **No integration connected, but org has WordPress integrations available** — list them, ask the user which to bind, then connect.
- **No integrations exist on the org at all** — the user needs to set up a WordPress integration first via the SurgeGraph dashboard. Direct them there; this skill cannot create the integration itself.

### 3. Fetch WordPress metadata

For the connected WordPress integration:

- List categories. Cache the response — same integration won't change categories during this skill run.
- List authors.

### 4. Confirm publish parameters with the user

Ask, in order:

- **Category** — one or more. If the user is unsure, propose the closest match by document topic.
- **Author** — if omitted, the document's mapped WordPress author is used. Surface this default explicitly so the user can override.
- **Publish status** — `draft`, `publish`, or `future` (scheduled). Default to `draft` unless the user is clear they want it live.
- **Scheduled time** — only if status is `future`. ISO 8601 in WordPress's site timezone.

### 5. Publish

Send the document. The response includes the WordPress post ID and the final URL. Surface both to the user.

## Decisions

| Situation | What to do |
|---|---|
| Default publish status | Always default to `draft` unless the user explicitly says "publish live," "send live," or similar. Drafts are reversible; live posts are not. |
| User didn't specify a category | Pick by topic match if a clear match exists; otherwise ask. Don't publish to "Uncategorized" by default — it makes the post hard to find on the site. |
| User wants to publish many documents | This skill handles one at a time. For batch publication, suggest looping the skill or — if the user has the SurgeGraph CLI — `surgegraph research gaps publish` for the end-to-end gap → write → publish pipeline. |
| WordPress site is in a non-English language | Categories and authors come back in the site's language. Pass them through unchanged; don't translate names. |
| User mentions Shopify, Webflow, or another CMS | Not supported yet. Tell the user; suggest they post the article to the WordPress site if they have one, or download the document and publish manually. |

## Common Issues

- **"No CMS integration connected"** — step 2 failed. Connect a WordPress integration or direct the user to set one up in the dashboard.
- **"Author not found" or 404 on author ID** — author lookup is per-WP-site, not per-org. Always fetch authors from the same integration the document will publish to.
- **Scheduled publish lands at the wrong time** — WordPress schedules in the site's configured timezone, not UTC. Confirm the timezone with the user before scheduling.
- **Post appears as draft despite `publish` status** — WordPress role permissions. The integration user account must have `publish_posts` capability on the WP site.
- **HTML formatting looks broken in WordPress** — SurgeGraph sends the document's HTML body as-is. WordPress's block editor may re-wrap blocks; classic editor preserves the HTML. If formatting matters, tell the user which editor their site uses.

## References

- [[surge-setup-aeo-tracking]] — projects need to exist before they can have a CMS integration.
- [[surge-opportunities-to-content]] — common upstream skill that produces documents ready to publish.
