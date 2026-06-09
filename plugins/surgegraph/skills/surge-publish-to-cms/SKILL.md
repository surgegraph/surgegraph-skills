---
name: surge-publish-to-cms
description: Publish a SurgeGraph Writer document to the CMS connected to its project (WordPress, Shopify, and future platforms) — pick the document, verify the integration, discover the platform's publish options, confirm choices with the user, then send. Use this skill whenever the user wants to publish, push, ship, post, send, or move a document/article/draft to their website, store, or blog, asks how to get a SurgeGraph article live, mentions WordPress or Shopify publishing, or wants to publish AI-written content to their site — even if they don't name a CMS. Also use when the user has just finished optimizing or generating content and the implied next step is publication.
---

# Publish a document to the connected CMS

This skill takes a Writer document or optimized document from "draft on SurgeGraph" to "live on the user's site." It is platform-agnostic: the project's CMS integration declares its own **capabilities** — valid publish statuses, accepted publish options, and platform notes — so never assume what a platform needs. Discover it.

## Quick Start

1. Identify the document to publish — by ID if known, otherwise by listing documents and asking the user.
2. Verify the project has a CMS integration connected. The integration response includes the platform's capabilities.
3. Fetch the integration's publish options — the live option sets behind each capability field (e.g. categories and authors on WordPress, blogs on Shopify).
4. Confirm publish status and option values with the user. Required fields first.
5. Publish.

## Workflow

### 1. Locate the document

If the user gave a document ID, use it. Otherwise:

- List the project's Writer documents and optimized documents.
- Ask the user which one to publish. Sort by most recently updated first.
- Confirm the title and last-updated time before proceeding.

### 2. Verify CMS integration

Fetch the project's CMS integration. Three possible states:

- **Connected** — proceed. Note the platform type and the capabilities block: it lists valid statuses and the option fields the publish call accepts.
- **No integration connected, but the org has integrations available** — list the org's CMS integrations, ask the user which to bind, then connect. The platform is detected from the integration automatically.
- **No integrations exist on the org at all** — the user needs to set one up via the SurgeGraph dashboard first. Direct them there; this skill cannot create the integration itself.

### 3. Discover publish options

Fetch the publish options for the connected integration. The response pairs the capability fields with live option sets:

- Each **option field** names a key the publish call accepts, whether it is required, and which option set lists its valid values.
- Cache the response — the same integration won't change options during this skill run.

### 4. Confirm publish parameters with the user

Ask, in order:

- **Required option fields first** — e.g. Shopify requires a blog. If only one valid value exists, propose it instead of asking.
- **Optional option fields** — e.g. WordPress categories and author. If the user is unsure about a category, propose the closest match by document topic.
- **Publish status** — from the capability statuses. Default to `draft` unless the user is clear they want it live.

### 5. Publish

Send the document with the chosen status and options. The response includes the post/article ID and the final URL — surface both to the user. Publishing the same document again updates the existing post instead of creating a duplicate.

## Platform quirks

| Platform      | What to know                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| WordPress     | Byline comes from the document's SurgeGraph author if that author is linked to a WordPress author ([[surge-author-setup]]); pass an author option to override. Statuses include `pending` and `private` beyond draft/publish. Don't publish to "Uncategorized" by default — it makes the post hard to find on the site.                                                                                                                                                   |
| Shopify       | A blog is required — articles always live in a blog. `draft` maps to a hidden article, `publish` to a visible one. The byline uses the SurgeGraph author's name directly (falls back to "Admin"); Shopify authors are plain names, not linkable accounts, so name the SurgeGraph author to match an existing store author if consistency matters. Meta description and images ship automatically (images are rehosted on Shopify's CDN); schema markup is WordPress-only. |
| Anything else | If the project's integration reports a platform this skill hasn't seen, trust the capabilities — they describe everything the publish call needs. If the org has no integration for the platform the user wants, it isn't supported yet: say so and suggest publishing to a connected platform or exporting manually.                                                                                                                                                     |

## Decisions

| Situation                                         | What to do                                                                                                                                                                                                         |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Default publish status                            | Always default to `draft` unless the user explicitly says "publish live," "send live," or similar. Drafts are reversible; live posts are not.                                                                      |
| User didn't specify a value for an optional field | Pick by topic match if a clear match exists; otherwise ask.                                                                                                                                                        |
| User wants to publish many documents              | This skill handles one at a time. For batch publication, suggest looping the skill or — if the user has the SurgeGraph CLI — `surgegraph research gaps publish` for the end-to-end gap → write → publish pipeline. |
| Site is in a non-English language                 | Option sets (categories, authors, blogs) come back in the site's language. Pass them through unchanged; don't translate names.                                                                                     |
| User asks to schedule a post for later            | Scheduled publishing isn't supported. Publish as `draft` and tell the user to schedule it from their CMS.                                                                                                          |

## Common Issues

- **"No CMS integration connected"** — step 2 failed. Connect an integration or direct the user to set one up in the dashboard.
- **"X is required when publishing to ..."** — a required option field was omitted. Re-fetch the publish options and ask the user to pick a value.
- **Option value rejected or 404** — option sets are per-integration, not per-org. Always fetch options from the same integration the document will publish to.
- **WordPress post appears as draft despite `publish` status** — WordPress role permissions. The integration user account must have `publish_posts` capability on the WP site.
- **HTML formatting looks broken in WordPress** — SurgeGraph sends the document's HTML body as-is. WordPress's block editor may re-wrap blocks; classic editor preserves the HTML. If formatting matters, tell the user which editor their site uses.
- **Shopify article published but not visible** — it was sent as `draft` (hidden). Republish with `publish` status or flip visibility in Shopify admin.

## References

- [[surge-setup-aeo-tracking]] — projects need to exist before they can have a CMS integration.
- [[surge-author-setup]] — set up the author whose byline the published content carries.
- [[surge-opportunities-to-content]] — common upstream skill that produces documents ready to publish.
