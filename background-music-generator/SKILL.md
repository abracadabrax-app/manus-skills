---
name: abracadabrax-background-music-generator
description: Create a background track with Abracadabrax and deliver the file link once the render finishes. Trigger on intents like "background music for my video", "royalty-free music for…", "a lo-fi beat for studying", "生成背景音乐", "做一段纯音乐", "视频配乐". Not for songs with vocals and lyrics, covers, remixes, extending or splitting an existing recording.
---

# Abracadabrax AI Background Music Generator

Describe a mood and get an original instrumental track to sit under a video, a podcast, an ad, a game, a livestream or a study session. Abracadabrax produces it with the credits of your own Abracadabrax account. This connector is text-only in Manus: every step answers with plain fields, and the finished file is handed over as an absolute link on abracadabrax.app.

## Connect in Manus

1. In Manus open Settings, then Connectors, and choose Create, then "Add MCP by URL".
2. Server URL: `https://abracadabrax.app/api/manus/background-music-generator/mcp`. Or choose "Import MCP by JSON" with:

```json
{"mcpServers": {"abracadabrax-background-music-generator": {"url": "https://abracadabrax.app/api/manus/background-music-generator/mcp"}}}
```

3. Save (not "Save and Publish"), then press Connect on the connector page and sign in to Abracadabrax; a new user can register there.
4. "Try it out" opens a new task with the connector on; in other tasks, turn it on from the connector icons in the chat input.

## Fits

- An original instrumental to sit under a video, a podcast, an ad, a game, a livestream or a study session.
- Choosing the mood, the genre, the tempo and the length (30, 60, 120 or 180 seconds).
- Loop-friendly tracks with no vocals.

## Does not fit

- Songs with vocals and lyrics, covers, remixes, extending or splitting an existing recording. Answer that this connector only creates new tracks from text and that uploads and edits live in the Abracadabrax music studio at https://abracadabrax.app.

## The three tools (server `abracadabrax-background-music-generator`)

- `prepare_background_music` — free, spends no credits; in Manus it runs on the connected account. Input: `mood` (required): Mood and feel, e.g. calm and warm, upbeat and confident, dark and tense; `use`: What the music is for; default youtube-video; `genre`: Genre or instruments, e.g. lo-fi hip hop, cinematic strings, corporate acoustic, ambient synth; `tempo`: Tempo, e.g. slow, 90 bpm, fast; `duration_sec`: Length in seconds: 30, 60, 120 (default) or 180; `title`: Optional track title; `model_id`: Optional model id: stable-audio (default, instrumental) or elevenlabs-music (studio quality, more credits). Output: `project` with `project.id` and the stored settings, `credits_required`, `dashboard_url`, `balance` (`credits`, `plan`) if signed in, `state` `ready`.
- `generate_song` — costs credits, needs the account. Input: `project_id`. Output: `state` plus `job` with `job.id`.
- `get_song_job` — free, needs the account, read-only. Input: `job_id`. Output: `state`, `job.result.audio` (link), `job.error`, `dashboard_url`.

## Playbook

1. Collect the mood, what the music is for, the genre and the length.
2. Save the draft with `prepare_background_music`.
3. Read the draft back: the settings, the model name and the credit cost from `credits_required`; add the remaining credits when `balance` is there.
4. Stop and wait. Rendering starts only after the user clearly agrees ("ok", "go ahead", "make it"). Spending the credits of an Abracadabrax account without that agreement is never acceptable. Any change after the draft means a new `prepare_background_music` call.
5. Run `generate_song` with `project_id`. A healthy answer is `state` `processing` (or `queued`) and a `job.id`.
6. Check `get_song_job` with that `job_id` every few seconds, without promising how long the render takes. Continue until `state` becomes `completed` or `failed`.
7. When `completed`, hand over the absolute link from `job.result.audio` (link) and `dashboard_url` to reopen it on Abracadabrax. Never substitute search results or other files for it.

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
- Vocals and music are AI-generated; never present them as a human performance.
