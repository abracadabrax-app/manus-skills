---
name: abracadabrax-song-maker
description: Write lyrics with the user and have Abracadabrax sing and produce the track, or build an instrumental, then deliver the audio link. Trigger on intents like "write me a song about…", "turn these lyrics into a song", "make a jingle", "background music for my video", "写一首歌并生成", "把这段歌词做成歌", "做一段纯音乐", "生成一首生日歌". Not for covers, remixes, extending or splitting an existing recording, voice cloning or transcription.
---

# Abracadabrax Song Maker

Abracadabrax Song Maker turns a style brief and optional lyrics into a finished audio track with text-to-music models. Available model ids: `minimax-music` (default, vocals with lyrics), `acestep-create` (fast, lyrics, chosen length), `elevenlabs-music` (studio quality, no lyrics), `stable-audio` (instrumental). Lengths of 30, 60, 120, 180 or 300 seconds are accepted by the models that support a chosen length. This connector is text-only in Manus: the tools answer with plain fields and the finished track is handed over as an absolute audio link on abracadabrax.app.

## Connect in Manus

1. In Manus open Settings, then Connectors, and choose Create, then "Add MCP by URL".
2. Server URL: `https://abracadabrax.app/api/manus/song/mcp`. Or choose "Import MCP by JSON" with:

```json
{"mcpServers": {"abracadabrax-song-maker": {"url": "https://abracadabrax.app/api/manus/song/mcp"}}}
```

3. Save (not "Save and Publish"), then press Connect on the connector page and sign in to Abracadabrax; a new user can register there.
4. "Try it out" opens a new task with the connector on; in other tasks, turn it on from the connector icons in the chat input.

## Fits

- A new song: anthem, ballad, lullaby, jingle, birthday song, rap verse, theme for a video.
- An instrumental or background track with no vocals.
- Revising the lyrics, style, model or length of a song draft that is not rendered yet.

## Does not fit

- Anything built on an existing recording: covers, remixes, extending a track, stem separation, replacing a section, cloning a voice, transcribing audio. Answer that Song Maker only creates new tracks from text and that uploads live in the Abracadabrax music studio at https://abracadabrax.app.
- Pictures or clips: that is the Abracadabrax Image & Video connector.

## The three tools (server `abracadabrax-song-maker`)

- `prepare_song_project` — free, spends no credits; in Manus it runs on the connected account. Input: `style` (required, up to 600 characters: genre, mood, tempo, instruments, vocal style), optionally `lyrics` (up to 3000 characters, with section tags such as [Verse], [Chorus], [Bridge]), `title` (up to 120 characters), `instrumental` (true for no vocals), `duration_sec`, `model_id`, `model_name`. If lyrics are given on a model that cannot sing, the server switches to a lyrics-capable model by itself. Output: `project` with `project.id`, `title`, `style`, `lyrics`, `instrumental`, `duration_sec`, `model_id`, `model_name`; `credits_required`; `dashboard_url`; `balance` (`credits`, `plan`) if signed in; `state` `ready`.
- `generate_song` — costs credits, needs the account. Input: `project_id`, plus any of the fields above to overwrite the draft at the last moment. Output: `state` plus `job` with `job.id`.
- `get_song_job` — free, needs the account, read-only. Input: `job_id`. Output: `state`, `job.result.audio` (link), `job.error`, `dashboard_url`.

## Playbook

1. Write the lyrics together with the user in the conversation, structured with [Verse], [Chorus] and similar tags, unless the user wants an instrumental. Agree on genre, mood, tempo and the kind of voice.
2. Save the draft with `prepare_song_project`: the style brief, the lyrics (or `instrumental: true`), a title if there is one, and a model or length only when the user asked for them.
3. Read the draft back: title, model name, length, whether it is instrumental, and the credit cost from `credits_required`; add the remaining credits when `balance` is there.
4. Stop and wait. Rendering starts only after the user clearly agrees ("ok", "produce it", "go ahead"). Spending the credits of an Abracadabrax account without that agreement is never acceptable. A late change (one more verse, a slower tempo) travels as an argument of `generate_song` next to `project_id`.
5. Run `generate_song` with `project_id`. A healthy answer is `state` `processing` (or `queued`) and a `job.id`.
6. Check `get_song_job` with that `job_id` every few seconds, without promising the user how long the render takes. Continue until `state` becomes `completed` or `failed`.
7. When `completed`, hand over the `job.result.audio` link and `dashboard_url` to reopen the song in the Abracadabrax music studio. Do not substitute search results or other music for the track.

## Account and login

In Manus the Abracadabrax account is linked when the connector is connected: Connect in the connector settings opens the Abracadabrax login page (a new user can register there) and Manus keeps the connection. Every tool, the `prepare_` tool included, runs on that connection. If a call answers that the account is required, ask the user to open the connector in Manus settings and press Connect, then repeat the identical call with the same `project_id`. Nobody ever needs an API key.

## Answers that are not a success, and what to say

- `ready` — draft stored, no render yet.
- `queued` / `processing` — rendering; keep checking.
- `completed` — deliver the link.
- `failed` — the render did not finish; quote `job.error` when present and offer another attempt or a different model.
- `pending_payment` — the job is parked because the account could not pay; the song is saved and continues on the site once credits are available. Share `dashboard_url`.
- `insufficient_credits` — the account lacks the credits for this track; the song is saved and resumes on the site when credits are available. Say so in one sentence and share `dashboard_url`. No purchase prompts, no plan recommendations in the conversation.
- `account_required` — the call reached the server without an account; ask the user to press Connect on the connector in Manus settings and retry, or share `dashboard_url`, where the song is already open.
- "Project not found" / "Job not found" — save a new draft.
- Error from a cover, extend, replace or stems model id — explain that those need an uploaded track in the music studio.

## Boundaries

- Credit figures come only from `credits_required` and `balance`; never estimate prices or waiting times yourself.
- Each render needs its own agreement from the user. New lyrics or a new style mean a new `prepare_song_project`.
- Vocals and music are AI-generated; never present them as a human performance.
