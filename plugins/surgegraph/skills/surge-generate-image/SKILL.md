---
name: surge-generate-image
description: Generate brand-aligned images for an article using the project's Content Vision settings — brand palette, style, composition preferences are applied automatically so output stays on-brand. Use this skill whenever the user asks to generate, create, or make images for an article, needs a hero image, featured image, or blog header, wants on-brand visual assets, mentions illustrating a post, asks for AI-generated images that look like their brand, or wants to bulk-generate images for multiple articles — even if they don't say "content vision" or "image generation." Also use as the natural follow-up step after [[surge-topic-research-to-article]] or [[surge-opportunities-to-content]] produces an article without imagery.
---

# Generate brand-aligned images

The SurgeGraph Content Vision system stores per-project brand visual settings — palette, style, composition rules — and applies them to image generation. This skill produces images that respect those settings, vs. asking a generic image API for "a hero image."

## Quick Start

1. Confirm the project ID and what the image is for (article hero, section illustration, social card, etc.).
2. Check the project's Content Vision settings — palette, style, any restrictions.
3. Check the gallery for reusable existing assets before generating new ones.
4. Compose an image prompt that incorporates the visual settings + the user's subject.
5. Generate. Iterate if needed.

## Workflow

### 1. Confirm scope

Ask if not provided:

- Project ID.
- Use case — hero image, section illustration, social card, thumbnail. This drives aspect ratio and composition.
- Subject — what the image should depict. If tied to an article, fetch the article and derive subject from its title and topic.
- Count — 1 image, or a set (e.g. one hero + 3 section illustrations).

### 2. Check Content Vision settings

Fetch the project's Content Vision settings. Capture:

- Brand palette (primary, accent, neutral colors).
- Style (photographic, illustration, isometric, flat, painterly, etc.).
- Composition preferences (center subject, rule-of-thirds, full-bleed, etc.).
- Restrictions (e.g. "no human faces," "no text overlays," "avoid stock-photo aesthetic").

If Content Vision settings aren't configured for the project, surface this. Suggest the user configure them in the SurgeGraph dashboard for consistent output across the project. Generation can proceed without settings, but output won't be aligned.

### 3. Check the gallery

Fetch the project's Content Vision gallery — previously generated images. Two reasons:

- **Reuse**: an existing image may already fit the use case. Reusing saves credits and produces consistency.
- **Consistency reference**: even when generating new images, recent gallery items establish the visual language to match.

If a good gallery match exists, surface it and ask the user before generating new.

### 4. Compose the prompt

Build an image prompt with three layers:

- **Subject** — what the image shows. From the user or derived from the article.
- **Style** — from Content Vision settings. Translated into image-model vocabulary (e.g. style="isometric" → "clean isometric illustration, soft shadows").
- **Palette + composition** — from settings. Stated as constraints in the prompt.

Surface the composed prompt to the user before generating, especially for higher-credit-cost models. Let them tweak before committing.

### 5. Generate

Call the image generation tool with the composed prompt. The output lands in the project's Content Vision gallery automatically.

### 6. Iterate if needed

If the user wants variations:

- Same prompt, different seed — multiple takes on the same idea.
- Modified prompt — adjust subject, style, or composition.

Don't iterate silently — surface the variations and ask the user which to keep. Generation consumes credits.

## Decisions

| Situation                                | What to do                                                                                                                                                                                                     |
| ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Content Vision settings not configured   | Generation works but won't be brand-aligned. Strongly suggest configuring settings via the dashboard. Don't proceed with multi-image batches without settings — wasted credits.                                |
| User wants images for an entire article  | A hero + 2-3 section illustrations is the standard set. Don't generate one image per section unless the user asks — over-imagery hurts readability.                                                            |
| User wants a logo or brand mark          | Image generation is for editorial imagery, not logo design. Surface this; suggest the user use dedicated logo tools.                                                                                           |
| Aspect ratio for use case                | Hero: 16:9 or 21:9. Social card: 1.91:1 (Open Graph) or 1:1 (Instagram). Section illustration: 4:3 or 16:9. Thumbnail: 1:1.                                                                                    |
| User wants human faces                   | Check Content Vision restrictions first — many brands explicitly avoid faces. If unrestricted, surface that AI-generated faces can be uncanny and slow consideration; consider style="illustration" to soften. |
| User mentions a competitor's image style | Don't replicate. Surface the request, ask the user to describe the style in their own terms (which goes into Content Vision settings going forward).                                                           |

## Common Issues

- **Output looks off-brand despite settings configured** — the image model may have ignored part of the prompt. Try restating the style constraints earlier in the prompt; some models weight prompt prefixes more heavily.
- **Colors are close but not exact to the brand palette** — image models approximate colors. Exact-color matching requires post-processing (recoloring) or a different model. Surface the limitation.
- **Faces look distorted** — common AI image artifact. Either use a model better tuned for faces, or shift the style to illustration where stylization hides distortion.
- **Generated image has text on it** — most image models can't reliably render text. Plan for text overlays in the layout tool (e.g. WordPress block editor) rather than in the image.
- **Credit cost higher than expected** — high-resolution or many-variation generation multiplies cost. Surface the cost before bulk generation.

## References

- [[surge-topic-research-to-article]] — natural upstream skill that produces an article needing imagery.
- [[surge-opportunities-to-content]] — bulk article generation that often needs accompanying imagery.
- [[surge-publish-to-cms]] — final step where the generated image is attached to the post.
