---
name: surge-setup-aeo-tracking
description: End-to-end onboarding for SurgeGraph AI Visibility tracking — creates a project, picks topics, generates tracking prompts, and applies tracking in bulk. Use this skill whenever the user mentions tracking a brand on AI search engines (ChatGPT, Google AI Overviews, Perplexity, Gemini, Bing Copilot), bootstrapping AEO, getting started with SurgeGraph, monitoring AI visibility, setting up AI citation tracking, or onboarding a new project — even if they don't say "SurgeGraph" or "AEO" explicitly. Also use when the user wants to automate steps they currently do manually in the SurgeGraph dashboard (selecting topics, generating prompts, choosing prompts to track).
---

# Set up AI Visibility tracking from scratch

This skill takes a brand and a domain and produces a project with a set of AI Visibility prompts actively tracked across answer engines. It replaces the in-app flow that requires manual topic selection, manual prompt generation, and manual prompt tracking.

The flow is gated on three async steps the SurgeGraph backend runs in the background:

1. **Domain research** — kicks off automatically on project creation. Cached for 14 days per domain.
2. **Topic generation** — derived from research output, runs lazily the first time topics are listed.
3. **Prompt generation** — an OpenAI batch that produces 10-50 candidate prompts per topic.

Steps 1 and 3 are async; the skill polls until each completes before proceeding.

## Quick Start

1. Gather: brand name, primary domain URL, location code, language code.
2. Create the project.
3. Poll setup status until domain research is complete.
4. List the auto-generated topic suggestions.
5. With the user (or "use top N" default), select topics to track.
6. Trigger prompt generation for the selected topics.
7. Poll setup status until prompt generation is complete.
8. List the generated prompts.
9. With the user (or "track top M per topic" default), pick which prompts to enable.
10. Apply tracking in bulk.
11. Activate tracking — always pass `schedule` and `engines` (use defaults if the user didn't state a preference). Dispatches the first run.

## Workflow

### 1. Gather inputs

Ask the user for:

- **Brand name** — how the brand should appear in AI-engine prompts.
- **Domain URL** — primary website domain.
- **Location** — country/region. If the user gives a name like "United States," look up the location code.
- **Language** — language code (e.g. `en`). Look up if unsure.

### 2. Create the project

Create the SurgeGraph project. Side effects happen automatically: domain research is kicked off, the four default answer engines are enabled, and project access is granted to org owners and managers.

The response includes `domainResearchStatus`:

- `cached_complete` — research was already done for this domain in the last 14 days. **Skip step 3.**
- `queued` — research is running asynchronously. Proceed to step 3.

### 3. Wait for research

Poll the project setup status. Read `isResearchCompleted` — proceed when it's `true`.

Polling cadence: every 30 seconds, with a 10-minute ceiling. If still incomplete at 10 minutes, tell the user research is taking longer than usual and offer to come back to it later.

### 4. List topic suggestions

Fetch the project's topic suggestions. Each topic has `id`, `topicName`, `topicDescription`, and `source` (auto-generated vs. user-added).

If the tool returns `generationStatus: research_pending`, research output is still being processed — back to step 3.

### 5. Select topics

Two modes:

- **Interactive**: show the topics to the user, let them pick.
- **Auto top-N**: if the user said something like "track the top 5 topics" or "use defaults," select the first N topics by `source` (auto-generated first, then user-added).

Default for "no preference stated": top 5 topics.

### 6. Trigger prompt generation

Pass the **topic names** of the selected topics. The tool will create the topic records on the fly if any names don't yet exist.

This kicks off a background OpenAI batch (GPT-5-mini) that produces 10-50 prompts per topic. The tool returns immediately with a queued acknowledgment.

The operation is **idempotent**: if prompt generation was already triggered for this project, the tool returns without re-running. Don't retry on the same project.

### 7. Wait for prompt generation

Poll the project setup status. Read `isPromptGenerationCompleted` — proceed when it's `true`.

Polling cadence: every 60 seconds, with a 15-minute ceiling.

### 8. List the generated prompts

Fetch the project's AI Visibility prompts. All newly generated prompts start with `isTracking: false`; nothing is tracked until the user opts in.

### 9. Select prompts to track

Two modes:

- **Interactive**: show prompts grouped by topic, let the user pick.
- **Auto top-M per topic**: if the user said "track top 10 per topic" or "use defaults," select the first M prompts per `topicId`.

Default for "no preference stated": top 10 prompts per topic.

### 10. Apply tracking in bulk

Pass an array of `{ promptId, isTracking: true }` pairs to the bulk tracking update tool. The update is atomic — all updates apply or none do.

Confirm to the user the total count enabled, grouped by topic.

### 11. Activate tracking

Call `activate_ai_visibility_tracking` with `projectId`, `schedule`, and `engines`. This single call configures **and** activates:

- Marks the project as activated (sets the `tracking_activated` flag — without this, the in-app UI keeps showing the "set up tracking" CTA).
- Dispatches the **first tracking run immediately** across all `isTracking: true` prompts on every enabled engine.
- Schedules the next run using the provided cadence.

**Both `schedule` and `engines` are required.** If the user didn't state a preference, pass these defaults verbatim:

```jsonc
{
  "schedule": {
    "interval": 1,
    "intervalUnit": "day",
    "time": "09:00",
    "timezone": "America/New_York"
  },
  "engines": [
    { "modelId": "chatgpt", "isEnabled": true },
    { "modelId": "google-ai-overview", "isEnabled": true },
    { "modelId": "google-ai-mode", "isEnabled": true },
    { "modelId": "gemini", "isEnabled": true },
    { "modelId": "perplexity", "isEnabled": false }
  ]
}
```

Activation always implies `active: true` — there's no way to set up a paused project via this tool. If the user wants to pause after setup, call `update_ai_visibility_schedule` separately.

If the user *did* state a preference (e.g. *"weekly on Mondays at 6am UTC"*, *"only ChatGPT and Perplexity"*, *"don't track Bing Copilot"*), overlay it onto the defaults. For weekly schedules add `daysOfWeek: [1]` (0=Sun … 6=Sat); for monthly add `daysOfMonth: [1]`.

The tool returns one of:

- `alreadyActive: true` — tracking was already activated; no new run was dispatched and any provided `schedule` / `engines` were **ignored**. If the user wanted config changes, call `update_ai_visibility_schedule` or `update_ai_visibility_engines` separately.
- `dispatched: true` — first run was queued. Report `promptCount`, `engineCount`, and `nextRunAt` to the user.
- `dispatched: false` (with `alreadyActive: false`) — activation flag was set but no run fired. Surface the returned `message` — usually means no prompts have `isTracking: true`, no engines are enabled, or the org is out of credits/plan access.

**Plan-gated errors** (returned as `{ error, blockedEngines? }` with no side effects):

- `blockedEngines: [...]` present → at least one engine in the `engines` arg isn't granted by the plan. Tell the user which engines were blocked and ask whether to (a) retry without those engines (or omit the `engines` arg to use defaults), or (b) pause setup until they upgrade.
- Schedule message *"...plan can run tracking at most every X days..."* → the requested cadence is faster than the plan allows. Ask the user for a less-frequent cadence, or omit the `schedule` arg to use defaults.

## Decisions

| Situation | What to do |
|---|---|
| User gave no preference for topic count | Default to top 5 topics. |
| User gave no preference for prompt count | Default to top 10 prompts per selected topic. |
| Research is taking longer than 10 minutes | Surface the wait; offer to resume later. Don't keep polling silently. |
| User is on Trial | Note that AI engine coverage is limited to ChatGPT and Google AI Overview regardless of how many prompts are tracked. |
| User asks about refresh cadence | Refresh cadence is **plan-gated** — depends on subscription. Do not claim a universal "daily" cadence. Point them to their plan details. |
| Quota error during generate or bulk-update | Surface the exact quota message from the tool. Suggest removing existing prompts or upgrading. |
| User wants alerts/notifications when visibility changes | Alerts are **not a SurgeGraph feature** — refresh happens on a plan-gated cadence and results land on the dashboard. Do not promise pings, emails, or webhooks. |

## Common Issues

- **`Insufficient permission`** — the user's role doesn't allow project creation. They need an org owner or manager to invite them with the right role.
- **`No active plan`** — prompt generation requires a paid subscription. Direct them to surgegraph.io.
- **Stuck on `isPromptGenerationTriggered: true` but never completes** — the OpenAI batch may have failed silently. Wait the full 15 minutes; if still stuck, contact `hello@surgegraph.io` with the project ID.
- **No topics appear after research completes** — research may have produced no extractable topics (e.g. a single-page domain with thin content). The user can add topics manually via the in-app flow.
- **User wants to re-run prompt generation with different topics** — not possible; generation is one-shot per project. Add more topics manually and create prompts one at a time, or contact support.

## References

- [AEO concepts: topics, prompts, engines](references/aeo-concepts.md) — 60-second primer on what each entity represents and how they relate.
