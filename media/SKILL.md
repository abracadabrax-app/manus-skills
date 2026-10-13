---
name: abracadabrax-image-video
description: Create a picture or a short clip from a prompt with Abracadabrax, choosing model, aspect ratio and clip length, and deliver the file link once the render finishes. Trigger on intents like "draw me a…", "make a 5-second clip of…", "poster of…", "illustration for my site", "用 Abracadabrax 生成图片", "做一段短视频", "画一张海报", "生成一个动画片段". Not for editing uploaded photos, background removal, upscaling, try-on or animating an existing image.
---

# Abracadabrax Image & Video

Abracadabrax creates still pictures and short clips from text with generative models (image: Nano Banana 2 by default; video: Kling 3.0 Pro by default; other ids such as `gpt-image-2`, `flux-2-pro`, `veo-3-1`, `sora-2`). This connector is text-only in Manus: every step answers with plain fields, and the finished file is handed over as an absolute link on abracadabrax.app.

## Connect in Manus

1. In Manus open Settings, then Connectors, and choose Create, then "Add MCP by URL".
2. Server URL: `https://abracadabrax.app/api/manus/media/mcp`. Or choose "Import MCP by JSON" with:

```json
{"mcpServers": {"abracadabrax-image-video": {"url": "https://abracadabrax.app/api/manus/media/mcp"}}}
```

3. Save (not "Save and Publish"), then press Connect on the connector page and sign in to Abracadabrax; a new user can register there.
4. "Try it out" opens a new task with the connector on; in other tasks, turn it on from the connector icons in the chat input.

## Fits

- A fresh image from a description: poster, illustration, concept, header, social visual.
- A fresh clip of a few seconds from a description.
- Changing model, aspect ratio or clip length of a draft already saved.

## Does not fit

- Anything that begins with a file the user already has: retouching a photo, cutting out a background, upscaling, virtual try-on, turning a picture into a clip, remixing a video. Answer that Abracadabrax creates only from text here and that the workspace at https://abracadabrax.app handles uploads.
- Songs and music: that is the Abracadabrax Song Maker connector.

## The three tools (server `abracadabrax-image-video`)

- `prepare_media_project` — free, spends no credits; in Manus it runs on the connected account. Input: `tab` (`image` or `video`) and `prompt` (up to 3000 characters), optionally `model_id`, `model_name`, `ratio` (for example 1:1, 16:9, 9:16, 4:3, 3:4; must be supported by the model, otherwise the model default applies) and `duration` (seconds, video only; snapped to a length the model supports). Output: `project` carrying `project.id` plus the options finally stored, `credits_required`, `dashboard_url`, `balance` (`credits`, `plan`) if signed in, `state` `ready`.
- `generate_media` — costs credits, needs the account. Input: `project_id`, plus any of the fields above to overwrite the draft at the last moment. Output: `state` plus `job` with `job.id`.
- `get_media_job` — free, needs the account, read-only. Input: `job_id`. Output: `state`, `job.result` (`images` list or `video` link), `job.error`, `dashboard_url`.

## Playbook

1. The moment the user describes something visual, save a draft with `prepare_media_project`. Decide `tab` from the wording ("clip", "animation", "seconds" mean video), keep the prompt faithful, and send only the options the user actually mentioned.
2. Read the draft back to the user: model, ratio, length for video, and the credit cost from `credits_required`; add the remaining credits when `balance` is there.
3. Stop and wait. Rendering starts only after the user clearly agrees ("ok", "render it", "go ahead"). Spending the credits of an Abracadabrax account without that agreement is never acceptable. A change requested at this point travels as an argument of `generate_media` next to `project_id`; no second draft is needed.
4. Run `generate_media` with `project_id`. A healthy answer is `state` `processing` (or `queued`) and a `job.id`.
5. Check `get_media_job` with that `job_id` every few seconds, without promising the user how long the render takes. Continue until `state` becomes `completed` or `failed`.
6. When `completed`, hand over every link in `job.result.images`, or the single `job.result.video` link, plus `dashboard_url` to reopen the draft in the Abracadabrax workspace. Do not decorate the reply with pictures from the web.

## Account and login

In Manus the Abracadabrax account is linked when the connector is connected: Connect in the connector settings opens the Abracadabrax login page (a new user can register there) and Manus keeps the connection. Every tool, the `prepare_` tool included, runs on that connection. If a call answers that the account is required, ask the user to open the connector in Manus settings and press Connect, then repeat the identical call with the same `project_id`. Nobody ever needs an API key.

## Answers that are not a success, and what to say

- `ready` — draft stored, no render yet.
- `queued` / `processing` — rendering; keep checking.
- `completed` — deliver the links.
- `failed` — the render did not finish; quote `job.error` when present and offer another attempt or a different model.
- `pending_payment` — the job is parked because the account could not pay; the draft is saved and continues on the site once credits are available. Share `dashboard_url`.
- `insufficient_credits` — the account lacks the credits for this render; the draft is saved and resumes on the site when credits are available. Say so in one sentence and share `dashboard_url`. No purchase prompts, no plan recommendations in the conversation.
- `account_required` — the call reached the server without an account; ask the user to press Connect on the connector in Manus settings and retry, or share `dashboard_url`, where the draft is already open.
- "Project not found" / "Job not found" — save a new draft.

## Boundaries

- Credit figures come only from `credits_required` and `balance`; never estimate prices or waiting times yourself.
- Each render needs its own agreement from the user. Different content means a new `prepare_media_project`.
- Repeat model names and settings exactly as the tools return them.
