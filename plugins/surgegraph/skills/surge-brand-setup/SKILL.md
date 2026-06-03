---
name: surge-brand-setup
description: Configure a SurgeGraph project's brand identity so the Writer and image generation stay on-brand — the brand profile (core identity, mandatory/prohibited vocabulary, target audience), the Content Vision look (brand colors and featured-image style), and the brand mentions (links the Writer weaves into content). Use this skill whenever the user wants to set a brand voice or guidelines, define words the brand must use or avoid, set the target audience, set brand colors for generated images, control featured/hero image style, add promotional links or UTM-tracked mentions for the Writer to insert, or onboard a client's brand — even if they don't say "brand profile" or "Content Vision." Pairs with [[surge-author-setup]] (per-author voice) and [[surge-setup-aeo-tracking]] (project creation).
---

# Set up a project's brand identity

This skill configures the brand layer the Writer and image generation rely on, across three areas:

1. **Brand profile** (project-scoped) — core identity, mandatory/prohibited vocabulary, and target audience. Shared by every author in the project.
2. **Content Vision** (project-scoped) — brand colors and featured-image style that steer AI image generation.
3. **Brand mentions** (**organization-scoped**) — links, with optional UTM tracking, that the Writer inserts when their trigger keywords appear.

It replaces copying these settings into the dashboard by hand after they've been decided. Brand profile and Content Vision are per-project; brand mentions apply org-wide across every project — call that distinction out to the user so they aren't surprised when a mention shows up in another project.

This is distinct from [[surge-author-setup]]: the **brand profile** is the project's shared identity (vocab, audience), while an **author** is one byline's individual voice. A typical onboarding does both.

## Quick Start

1. Confirm the project ID.
2. Read the current brand profile (auto-generated from research on first read); review and refine it.
3. Set the Content Vision brand colors and featured-image style.
4. Add or update brand mentions (org-wide).

## Workflow

### 1. Brand profile (project-scoped)

Read the brand profile first. On a project that has none yet, the read **auto-generates** one from the project's completed research — the system AI-extracts identity values, mandatory/prohibited vocabulary, and target audience. (If research hasn't finished, the generated profile may be sparse or empty; note that and proceed.)

Show the user what's there, then refine it. Update only the fields that should change — each field left out is preserved:

- **Identity values** — the brand's core mission/values, as prose.
- **Mandatory vocabulary** — words/phrases the brand consistently uses.
- **Prohibited vocabulary** — words/phrases the brand avoids.
- **Target audience** — audience segments.

Updating a project that still has no brand row creates it from the values you supply.

### 2. Content Vision (project-scoped)

Content Vision settings steer AI image generation. Read the current settings first — this also returns the valid **image types and styles** (with their IDs) for the project.

- **Brand colors** — set a `primary`, a `background`, and up to 10 `secondary` colors, all as 6-digit hex (e.g. `#1A2B3C`). These bias generated imagery toward the brand palette. If the user only has a website and not exact hex values, ask them for the hex (or have them sample it) — color is set explicitly here.
- **Featured (hero) image** — choose behavior:
  - **Adaptive** (`isAdaptive: true`) — the type/style is chosen automatically per article. Good default.
  - **Fixed** — supply a specific `imageTypeId` and `styleId` (use the IDs from the settings read), plus up to 5 hex colors to bias the palette.

If no Content Vision settings exist yet, setting either colors or featured-image creates them.

### 3. Brand mentions (organization-scoped)

A brand mention is a link the Writer can weave into content when one of its **trigger keywords** appears — optionally with UTM parameters for attribution. **These are organization-wide, not per-project** — confirm with the user before adding, since a mention affects content across all their projects.

First list existing mentions so you don't create duplicates. Then for each new one, capture:

- **Name** — label for the mention.
- **URL** — the destination link.
- **Trigger keywords** — the keywords that cause the Writer to insert it.
- **Preferred anchor texts** — optional; what the link text should read as.
- **Default UTM params** — optional; e.g. `{ "utm_source": "ai-content" }`, appended to the tracking URL.
- **Active?** — whether the mention is live (defaults to active).

Use the update path to edit an existing mention (only the fields you pass change) rather than creating a second one.

## Decisions

| Situation                                                      | What to do                                                                                                                                                     |
| -------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Project research isn't finished when reading the brand profile | The auto-generated profile may be empty/sparse. Proceed with manual values, or wait for research (see [[surge-setup-aeo-tracking]]) and re-read later.         |
| User wants to regenerate the brand profile from scratch        | A plain re-read won't regenerate once a row exists. Treat the existing values as the base and refine them, or have the user clear fields explicitly.           |
| User only has a website, no exact brand colors                 | Ask for the hex values (or have them sample the site). Colors are set explicitly; there is no automatic color inference in this flow.                          |
| Featured image: user has no preference                         | Use adaptive (`isAdaptive: true`) so type/style is chosen per article.                                                                                         |
| Fixed featured image but unsure of valid IDs                   | Read Content Vision settings first — it returns the available image types and styles with their IDs.                                                           |
| User asks to add a brand mention "for this project only"       | Not possible — brand mentions are organization-scoped. Explain they apply across all projects; the trigger keywords are what scope where they actually appear. |
| Duplicate-looking brand mentions                               | List existing mentions first; update the existing one instead of creating another (the backend does not de-dupe).                                              |

## Common Issues

- **Generated brand profile is empty** — the project's research hasn't completed (or produced little). Fill the fields manually, or revisit after research finishes.
- **`Invalid primary color` / color rejected** — colors must be 6-digit hex like `#1A2B3C` (not `#123`, not a name like "blue").
- **Featured image update rejected** — the `imageTypeId`/`styleId` don't match a real type/style, or more than 5 colors were supplied. Re-read settings for valid IDs and cap colors at 5.
- **`Insufficient permission`** — the user's role doesn't allow managing brand/Content Vision settings. They need a role with the right access from an org owner or manager.
- **A brand mention isn't appearing in content** — it's inactive, or its trigger keywords don't occur in the generated content. Check `isActive` and broaden the trigger keywords.

## References

- [[surge-author-setup]] — per-author voice synthesis; the brand profile here is the project-wide complement to each author's individual voice.
- [[surge-setup-aeo-tracking]] — create the project (and run the research the brand profile is generated from) first.
- [[surge-generate-image]] — generate images that use the Content Vision colors and styles set here.
- [[surge-knowledge-library-bootstrap]] — give the Writer grounding context alongside the brand profile.
