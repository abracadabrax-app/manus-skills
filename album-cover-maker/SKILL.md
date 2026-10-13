---
name: abracadabrax-album-cover-maker
description: Create an album cover with Abracadabrax and deliver the file link once the render finishes. Trigger on intents like "album cover for…", "single artwork", "a playlist cover", "做专辑封面", "单曲封面", "歌单封面". Not for putting the user's own photo or band logo on the cover, editing an existing cover, making the music itself.
---

# Abracadabrax Album Cover Art Maker

Give Manus the title of your album, single, EP, mixtape or playlist, the artist name, the genre and the mood, and get square cover art that still reads as a small thumbnail on streaming platforms, with the title (and the artist) set into the artwork or no text at all. Abracadabrax renders it with the credits of your own Abracadabrax account. This connector is text-only in Manus: every step answers with plain fields, and the finished file is handed over as an absolute link on abracadabrax.app.

## Connect in Manus

1. In Manus open Settings, then Connectors, and choose Create, then "Add MCP by URL".
2. Server URL: `https://abracadabrax.app/api/manus/album-cover-maker/mcp`. Or choose "Import MCP by JSON" with:

```json
{"mcpServers": {"abracadabrax-album-cover-maker": {"url": "https://abracadabrax.app/api/manus/album-cover-maker/mcp"}}}
```

3. Save (not "Save and Publish"), then press Connect on the connector page and sign in to Abracadabrax; a new user can register there.
4. "Try it out" opens a new task with the connector on; in other tasks, turn it on from the connector icons in the chat input.

## Fits

- Square cover art for an album, a single, an EP, a mixtape or a playlist.
- The title (and the artist) set into the artwork, or no text at all.
- A look chosen from the genre, the mood and a visual style.

## Does not fit

- Putting the user's own photo or band logo on the cover, editing an existing cover, making the music itself. Answer that this connector only creates new files from text and that uploads and edits live in the Abracadabrax workspace at https://abracadabrax.app.

## The three tools (server `abracadabrax-album-cover-maker`)

- `prepare_album_cover` — free, spends no credits; in Manus it runs on the connected account. Input: `title` (required): Title of the album, single or playlist; `artist`: Artist, band or curator name; `kind`: album (default), single, ep, mixtape or playlist; `genre`: Music genre, e.g. synthwave, indie folk, trap, jazz; `mood`: Mood of the music, e.g. melancholic, euphoric, dark, dreamy; `visual_style`: Visual style, e.g. 35mm film photo, abstract gradients, vintage collage, hand-drawn illustration; `text_on_cover`: title-and-artist (default), title-only or none; `model_id`: Optional image model id: nano-banana-pro (default, sharp text), gpt-image-2, nano-banana-2, flux-2-pro. Output: `project` with `project.id` and the stored settings, `credits_required`, `dashboard_url`, `balance` (`credits`, `plan`) if signed in, `state` `ready`.
- `generate_media` — costs credits, needs the account. Input: `project_id`. Output: `state` plus `job` with `job.id`.
- `get_media_job` — free, needs the account, read-only. Input: `job_id`. Output: `state`, `job.result.images` (list of links), `job.error`, `dashboard_url`.

## Playbook

1. Collect the title, the artist name, the genre, the mood, the visual style and whether the title is printed on the cover.
2. Save the draft with `prepare_album_cover`.
3. Read the draft back: the settings, the model name and the credit cost from `credits_required`; add the remaining credits when `balance` is there.
4. Stop and wait. Rendering starts only after the user clearly agrees ("ok", "go ahead", "make it"). Spending the credits of an Abracadabrax account without that agreement is never acceptable. Any change after the draft means a new `prepare_album_cover` call.
5. Run `generate_media` with `project_id`. A healthy answer is `state` `processing` (or `queued`) and a `job.id`.
6. Check `get_media_job` with that `job_id` every few seconds, without promising how long the render takes. Continue until `state` becomes `completed` or `failed`.
7. When `completed`, hand over the absolute link from `job.result.images` (list of links) and `dashboard_url` to reopen it on Abracadabrax. Never substitute search results or other files for it.

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
