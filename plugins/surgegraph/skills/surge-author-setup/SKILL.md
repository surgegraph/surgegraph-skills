---
name: surge-author-setup
description: Create a SurgeGraph author (a per-project writer persona) and synthesize their voice from real writing samples, so the Writer attributes content to a consistent byline and writes in that person's voice. Use this skill whenever the user wants to set up an author or writer persona, give the Writer a specific voice, make content "sound like" a named person, synthesize an author's tone from their existing articles, onboard a byline for a client, attribute content to a particular author, or link a WordPress author for publishing — even if they don't say "author synthesis." Pairs with [[surge-brand-setup]] (project-level brand profile) and [[surge-setup-aeo-tracking]] (project creation).
---

# Set up an author with a synthesized voice

An author (Author Synthesis) is a **per-project writer persona**: a name, bio, voice calibration, worldview, and a corpus of representative writing that the Writer uses to produce content in a consistent voice. A project can have several authors (one per byline).

This skill creates an author and synthesizes their voice from real writing samples, optionally linking them to a WordPress author for publishing attribution. It replaces the in-app flow of hand-creating an author and pasting samples one screen at a time.

Voice synthesis is one async-ish, LLM-backed step: it crawls the sample URLs, analyzes them, and writes back a bio, a three-axis voice calibration, a worldview summary, and up to 20 corpus fragments. It is **quota-gated** (counts against the project's author-synthesis allowance) and **destructive on re-run** (a second synthesis overwrites the prior bio, voice, and corpus).

## Quick Start

1. Confirm the project ID and the author's display name.
2. (Optional) pick a WordPress author to link for the publishing byline.
3. Create the author.
4. Collect writing samples — public URLs and/or pasted text.
5. Run voice synthesis.
6. Review the synthesized voice + corpus; refine if needed.

## Workflow

### 1. Confirm scope

Ask the user for:

- **Project ID** — the author is scoped to one project. If no project exists yet, set one up first via [[surge-setup-aeo-tracking]].
- **Author name** — the display name (e.g. "Jane Doe").
- **Bio** — optional; synthesis will generate one, so a manual bio is only needed if the user wants to pin it.
- **CMS link?** — whether this author should map to a WordPress author so published content carries that byline.

### 2. (Optional) resolve the WordPress author

If the user wants a publishing byline, list the authors on their connected WordPress site and let them pick. Capture that author's `id`, `name`, `slug`, `bio`, and `avatarUrl`.

These are stored as a **snapshot** on the SurgeGraph author — the link is not resolved later from the id alone, so the fields must be passed together as a set. (If the user isn't publishing to WordPress, skip this entirely.)

### 3. Create the author

Create the author with the name (plus bio and the CMS snapshot fields if linking). The new author starts in **`pending`** status — it exists, but has no synthesized voice yet.

If the user already described a voice in plain terms ("formal and concise," "upbeat"), you may set the voice calibration up front, but synthesis will refine it from real samples anyway.

### 4. Collect writing samples

Voice synthesis needs real writing to analyze. Gather:

- **URLs** — up to 10 public pages of the author's writing (articles, blog posts, LinkedIn posts with public URLs). Pages behind a login or heavily JavaScript-rendered may yield no extractable text.
- **Pasted content** — up to 50,000 characters, for samples that aren't on public URLs.

At least one of the two is required. More varied samples produce a more accurate voice.

### 5. Run voice synthesis

Run synthesis for the author with the collected URLs and/or pasted content. This:

- crawls the URLs and converts them to text,
- runs an LLM analysis, and
- saves the resulting **bio**, **voice calibration** (three 1–5 axes: formal↔casual, verbose↔concise, optimistic↔cynical), **worldview & biases**, and up to **20 corpus fragments**.

It can take a while (crawling several pages plus the LLM call), and it **counts against the project's author-synthesis quota**. While it runs the author's status is `analyzing`; on success it becomes `completed`. If it fails the status reverts to `pending` and nothing is saved.

Surface progress to the user — this is not instant.

### 6. Review and refine

Show the user the synthesized result: the bio, the three voice-calibration values, the worldview summary, and how many corpus fragments were captured.

- If the voice is close but a knob is off (e.g. too formal), adjust the author's voice calibration directly rather than re-synthesizing.
- If the samples were poor and the whole result is weak, gather better samples and re-run synthesis (this overwrites the previous result — tell the user before doing it).

## Decisions

| Situation                                            | What to do                                                                                                                                                                                                   |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Multiple bylines for the project                     | Repeat the whole flow once per author. Each is independent.                                                                                                                                                  |
| Author has no public URLs                            | Use pasted content only. Synthesis works from either source.                                                                                                                                                 |
| User wants to re-run synthesis on an existing author | Allowed, but warn first: it **overwrites** the existing bio, voice, and corpus, and consumes another synthesis from the quota.                                                                               |
| Voice calibration is slightly off after synthesis    | Update the author's calibration directly. Don't re-synthesize just to nudge one axis.                                                                                                                        |
| Linking a WordPress author                           | Source the snapshot (id, name, slug, bio, avatarUrl) from the connected WordPress site and pass them together with `cmsType: "wordpress"`. The SurgeGraph author name then tracks the WordPress author name. |
| User only wants a byline, no voice work              | Create the author (optionally CMS-linked) and stop — synthesis is optional. The author will use a neutral default voice.                                                                                     |
| Author-synthesis quota reached                       | Surface the exact quota message from the tool. Suggest removing an unused author or upgrading.                                                                                                               |

## Common Issues

- **Synthesis result looks generic or empty** — the samples were thin, behind a login, or JavaScript-rendered with no server-side text. Provide pasted content instead of (or in addition to) URLs.
- **A sample URL returns 403** — the source site blocks automated fetches. Paste that article's text directly.
- **Author stuck in `analyzing`** — a synthesis run failed partway. The status reverts to `pending`; just run synthesis again with the same or better samples.
- **`Insufficient permission`** — the user's role doesn't allow updating authors. They need a role with author-management access from an org owner or manager.
- **Published content shows the wrong byline** — the author isn't CMS-linked, or is linked to the wrong WordPress author. Re-link with the correct WordPress author snapshot.

## References

- [[surge-brand-setup]] — the **project-level** brand profile (identity values, mandatory/prohibited vocabulary, target audience) that complements each author's individual voice.
- [[surge-setup-aeo-tracking]] — create the project first if one doesn't exist.
- [[surge-knowledge-library-bootstrap]] — give the Writer grounding context to pair with the author's voice.
- [[surge-publish-to-wordpress]] — publish content under the linked author's byline.
