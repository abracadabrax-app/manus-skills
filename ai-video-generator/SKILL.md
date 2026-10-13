---
name: abracadabrax-ai-video-generator
description: Create a video with Abracadabrax and deliver the file link once the render finishes. Trigger on intents like "make a video of…", "a 5-second clip of…", "text to video", "a vertical clip for TikTok or Reels", "生成一段视频", "文字生成视频", "做一个竖屏短视频". Not for animating an uploaded photo, editing or extending an existing video, lip sync, adding a soundtrack to a video.
---

# Abracadabrax AI Video Generator

Describe a scene and get a short AI video clip without leaving Manus: a landscape shot for YouTube, a vertical clip for TikTok, Reels or Shorts, or a square loop. Abracadabrax renders it with the credits of your own Abracadabrax account. This connector is text-only in Manus: every step answers with plain fields, and the finished file is handed over as an absolute link on abracadabrax.app.

## Connect in Manus

1. In Manus open Settings, then Connectors, and choose Create, then "Add MCP by URL".
2. Server URL: `https://abracadabrax.app/api/manus/ai-video-generator/mcp`. Or choose "Import MCP by JSON" with:

```json
{"mcpServers": {"abracadabrax-ai-video-generator": {"url": "https://abracadabrax.app/api/manus/ai-video-generator/mcp"}}}
```

3. Save (not "Save and Publish"), then press Connect on the connector page and sign in to Abracadabrax; a new user can register there.
4. "Try it out" opens a new task with the connector on; in other tasks, turn it on from the connector icons in the chat input.

## Fits

- A short clip from a scene description: subject, action, setting and camera.
- Landscape for YouTube, vertical for TikTok, Reels or Shorts, or a square loop.
- Choosing the visual style, the length and the video model before rendering.

## Does not fit

- Animating an uploaded photo, editing or extending an existing video, lip sync, adding a soundtrack to a video. Answer that this connector only creates new files from text and that uploads and edits live in the Abracadabrax workspace at https://abracadabrax.app.

## The three tools (server `abracadabrax-ai-video-generator`)

- `prepare_ai_video` — free, spends no credits; in Manus it runs on the connected account. Input: `scene` (required): What happens in the clip: subject, action, setting, camera movement, lighting; `style`: Visual style, e.g. cinematic, anime, 3D animation, documentary, claymation; `format`: landscape 16:9 (default), vertical 9:16 or square 1:1 (square only on Kling and Seedance); `duration`: Length in seconds; must be one the model supports (Kling 5 or 10, Seedance 5-12, Sora 4/8/12, Veo 4/6/8); `model_id`: Optional video model id: kling-3-pro (default), kling-3-standard, seedance-1-5-pro, seedance-1-lite, sora-2, veo-3-1-fast, veo-3-1. Output: `project` with `project.id` and the stored settings, `credits_required`, `dashboard_url`, `balance` (`credits`, `plan`) if signed in, `state` `ready`.
- `generate_media` — costs credits, needs the account. Input: `project_id`. Output: `state` plus `job` with `job.id`.
- `get_media_job` — free, needs the account, read-only. Input: `job_id`. Output: `state`, `job.result.video` (link), `job.error`, `dashboard_url`.

## Playbook

1. Collect what happens in the clip (subject, action, setting, camera), the visual style, the format (landscape, vertical or square) and the length.
2. Save the draft with `prepare_ai_video`.
3. Read the draft back: the settings, the model name and the credit cost from `credits_required`; add the remaining credits when `balance` is there.
4. Stop and wait. Rendering starts only after the user clearly agrees ("ok", "go ahead", "make it"). Spending the credits of an Abracadabrax account without that agreement is never acceptable. Any change after the draft means a new `prepare_ai_video` call.
5. Run `generate_media` with `project_id`. A healthy answer is `state` `processing` (or `queued`) and a `job.id`.
6. Check `get_media_job` with that `job_id` every few seconds, without promising how long the render takes. Continue until `state` becomes `completed` or `failed`.
7. When `completed`, hand over the absolute link from `job.result.video` (link) and `dashboard_url` to reopen it on Abracadabrax. Never substitute search results or other files for it.

## Account and login

In Manus the Abracadabrax account is linked when the connector is connected: Connect in the connector settings opens the Abracadabrax login page (a new user can register there) and Manus keeps the connection. Every tool, the `prepare_` tool included, runs on that connection. If a call answers that the account is required, ask the user to open the connector in Manus settings and press Connect, then repeat the identical call with the same `project_id`. Nobody ever needs an API key.

## Answers that are not a success, and what to say

- `ready` — draft stored, no render yet.
- `queued` / `processing` — rendering; keep checking.
- `completed` — deliver the link.
- `failed` — the render did not finish; quote `job.error` when present and offer another attempt.
- `pending_payment` — the job is parked because the account could not pay; the draft is saved and continues on the site once credits are available. Share `dashboard_url`.
- `insufficient_credits` — the account lacks the credits; the draft is saved and resumes on the site when credits are available. Say so in one sentence and share `dashboard_url`. No purchase prompts, no plan recommendations in the conversation.
- `account_required` — the call reached the server without an account; ask the user to press Connect on the connector in Manus settings and retry, or share `dashboard_url`, where the draft is already open.
- "Project not found" / "Job not found" — save a new draft.

## Boundaries

- Credit figures come only from `credits_required` and `balance`; never estimate prices or waiting times yourself.
- Each render needs its own agreement from the user.
- Images and videos are AI-generated; never present them as photos of real events or products.
