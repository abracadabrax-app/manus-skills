---
name: abracadabrax-birthday-song-maker
description: Create a birthday song with Abracadabrax and deliver the file link once the render finishes. Trigger on intents like "a birthday song for…", "a happy birthday song with her name", "a custom birthday song", "做一首生日歌", "生日快乐歌加上名字", "定制生日歌曲". Not for covers, singing over an existing song, cloning someone's voice, editing an uploaded track.
---

# Abracadabrax Birthday Song Maker

Make a birthday song that uses the person's name and your own memories: Manus helps you write the lyrics, and Abracadabrax sings and produces the track in the style you pick. It is produced with the credits of your own Abracadabrax account. This connector is text-only in Manus: every step answers with plain fields, and the finished file is handed over as an absolute link on abracadabrax.app.

## Connect in Manus

1. In Manus open Settings, then Connectors, and choose Create, then "Add MCP by URL".
2. Server URL: `https://abracadabrax.app/api/manus/birthday-song-maker/mcp`. Or choose "Import MCP by JSON" with:

```json
{"mcpServers": {"abracadabrax-birthday-song-maker": {"url": "https://abracadabrax.app/api/manus/birthday-song-maker/mcp"}}}
```

3. Save (not "Save and Publish"), then press Connect on the connector page and sign in to Abracadabrax; a new user can register there.
4. "Try it out" opens a new task with the connector on; in other tasks, turn it on from the connector icons in the chat input.

## Fits

- A birthday song that uses the person's name and details the user shares (age, memories, inside jokes).
- Lyrics written together in the conversation, with [Verse] and [Chorus] tags.
- Choosing the music style and a female or male voice.

## Does not fit

- Covers, singing over an existing song, cloning someone's voice, editing an uploaded track. Answer that this connector only creates new tracks from text and that uploads and edits live in the Abracadabrax music studio at https://abracadabrax.app.

## The three tools (server `abracadabrax-birthday-song-maker`)

- `prepare_birthday_song` — free, spends no credits; in Manus it runs on the connected account. Input: `name` (required): Name of the person the song is for; `lyrics` (required): Lyrics written with the user for this person, using the name and personal details, with [Verse] and [Chorus] tags; `style`: Music style, e.g. upbeat pop, acoustic folk, rock, reggaeton, jazz. Default upbeat pop; `voice`: Vocal: female (default) or male; `occasion`: Occasion if more specific, e.g. 30th birthday, surprise party. Default birthday; `model_id`: Optional model id: minimax-music (default, vocals with lyrics) or acestep-create. Output: `project` with `project.id` and the stored settings, `credits_required`, `dashboard_url`, `balance` (`credits`, `plan`) if signed in, `state` `ready`.
- `generate_song` — costs credits, needs the account. Input: `project_id`. Output: `state` plus `job` with `job.id`.
- `get_song_job` — free, needs the account, read-only. Input: `job_id`. Output: `state`, `job.result.audio` (link), `job.error`, `dashboard_url`.

## Playbook

1. Collect the person's name, a few personal details (age, memories, inside jokes) to write the lyrics with the user, the music style and the voice.
2. Save the draft with `prepare_birthday_song`.
3. Read the draft back: the settings, the model name and the credit cost from `credits_required`; add the remaining credits when `balance` is there.
4. Stop and wait. Rendering starts only after the user clearly agrees ("ok", "go ahead", "make it"). Spending the credits of an Abracadabrax account without that agreement is never acceptable. Any change after the draft means a new `prepare_birthday_song` call.
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
