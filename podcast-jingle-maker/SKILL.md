---
name: abracadabrax-podcast-jingle-maker
description: Create a jingle with Abracadabrax and deliver the file link once the render finishes. Trigger on intents like "podcast intro music", "a jingle for my show", "a channel intro that sings the show name", "做一段播客片头", "节目 jingle", "频道片头音乐". Not for full-length songs, long background tracks, covers, remixes, extending an existing recording.
---

# Abracadabrax Podcast Intro & Jingle Maker

Give Manus the name of your podcast, channel or stream and its feel, and get a short signature theme to open or close every episode: 30 or 60 seconds, instrumental, or with a short sung tagline that says the show's name. Abracadabrax produces it with the credits of your own Abracadabrax account. This connector is text-only in Manus: every step answers with plain fields, and the finished file is handed over as an absolute link on abracadabrax.app.

## Connect in Manus

1. In Manus open Settings, then Connectors, and choose Create, then "Add MCP by URL".
2. Server URL: `https://abracadabrax.app/api/manus/podcast-jingle-maker/mcp`. Or choose "Import MCP by JSON" with:

```json
{"mcpServers": {"abracadabrax-podcast-jingle-maker": {"url": "https://abracadabrax.app/api/manus/podcast-jingle-maker/mcp"}}}
```

3. Save (not "Save and Publish"), then press Connect on the connector page and sign in to Abracadabrax; a new user can register there.
4. "Try it out" opens a new task with the connector on; in other tasks, turn it on from the connector icons in the chat input.

## Fits

- A short signature theme to open or close every episode of a podcast, a channel or a stream.
- Instrumental, or a short sung tagline that says the show's name.
- 30 or 60 seconds, in the genre and mood of the show.

## Does not fit

- Full-length songs, long background tracks, covers, remixes, extending an existing recording. Answer that this connector only creates new tracks from text and that uploads and edits live in the Abracadabrax music studio at https://abracadabrax.app.

## The three tools (server `abracadabrax-podcast-jingle-maker`)

- `prepare_podcast_jingle` — free, spends no credits; in Manus it runs on the connected account. Input: `show_name` (required): Name of the podcast, channel or show; `mood`: Feel of the show, e.g. upbeat and friendly, serious news, cozy, high-energy gaming; `genre`: Genre or instruments, e.g. funky bass, lo-fi, orchestral, synth pop; `vocals`: none (default, instrumental), female or male for a sung tagline; `tagline`: Sung line when vocals are on, e.g. You're listening to The Morning Brew. Default: the show's name; `duration_sec`: Length in seconds: 30 (default) or 60; `model_id`: Optional model id for an instrumental jingle: stable-audio (default) or acestep-create. A sung tagline always uses acestep-create. Output: `project` with `project.id` and the stored settings, `credits_required`, `dashboard_url`, `balance` (`credits`, `plan`) if signed in, `state` `ready`.
- `generate_song` — costs credits, needs the account. Input: `project_id`. Output: `state` plus `job` with `job.id`.
- `get_song_job` — free, needs the account, read-only. Input: `job_id`. Output: `state`, `job.result.audio` (link), `job.error`, `dashboard_url`.

## Playbook

1. Collect the show's name, its mood and genre, whether the jingle is instrumental or sings a short tagline with the show's name, and the length (30 or 60 seconds).
2. Save the draft with `prepare_podcast_jingle`.
3. Read the draft back: the settings, the model name and the credit cost from `credits_required`; add the remaining credits when `balance` is there.
4. Stop and wait. Rendering starts only after the user clearly agrees ("ok", "go ahead", "make it"). Spending the credits of an Abracadabrax account without that agreement is never acceptable. Any change after the draft means a new `prepare_podcast_jingle` call.
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
