# YouTube Music Bot — Bot specification

**Archetype:** custom

**Voice:** playful and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that plays YouTube music in voice chats with play/pause/skip/volume controls and queue management.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- group admins
- music lovers
- voice chat participants

## Success criteria

- user can play a YouTube URL in a voice chat
- user can pause/resume/skip tracks
- user can adjust volume
- bot leaves voice chat on /stop or timeout

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/play** (command, actor: user, command: /play) — Play a YouTube video in the current voice chat
  - inputs: YouTube URL or search query
  - outputs: Playing confirmation, Error message if invalid URL
- **/pause** (command, actor: user, command: /pause) — Pause current playback
  - outputs: Paused confirmation
- **/resume** (command, actor: user, command: /resume) — Resume paused playback
  - outputs: Resumed confirmation
- **/skip** (command, actor: user, command: /skip) — Skip to next track in queue
  - outputs: Skipped confirmation, Queue empty message
- **/stop** (command, actor: user, command: /stop) — Stop playback and leave voice chat
  - outputs: Stopped confirmation, Voice chat leave confirmation
- **/volume** (command, actor: user, command: /volume) — Adjust playback volume (0-100)
  - inputs: Volume level 0-100
  - outputs: Volume set confirmation, Error message if invalid volume
- **/queue** (command, actor: user, command: /queue) — Show current queue
  - outputs: Queue listing, Empty queue message
- **/nowplaying** (command, actor: user, command: /nowplaying) — Show currently playing track
  - outputs: Current track info, No track playing message

## Flows

### Play video
_Trigger:_ /play <url or search>

1. Extract video info using yt-dlp
2. Add to queue
3. Join voice chat if not already connected
4. Play video
5. Send playing confirmation

_Data touched:_ queue, current_track, voice_chat_session

### Pause playback
_Trigger:_ /pause

1. Pause current playback
2. Send paused confirmation

_Data touched:_ current_track

### Resume playback
_Trigger:_ /resume

1. Resume current playback
2. Send resumed confirmation

_Data touched:_ current_track

### Skip track
_Trigger:_ /skip

1. Remove current track from queue
2. Play next track if available
3. Send skipped confirmation

_Data touched:_ queue, current_track

### Stop playback
_Trigger:_ /stop

1. Stop current playback
2. Clear queue
3. Leave voice chat
4. Send stopped confirmation

_Data touched:_ queue, current_track, voice_chat_session

### Adjust volume
_Trigger:_ /volume <0-100>

1. Validate volume level
2. Set playback volume
3. Send volume confirmation

_Data touched:_ current_track

### Show queue
_Trigger:_ /queue

1. Get current queue
2. Format queue listing
3. Send queue message

_Data touched:_ queue

### Show current track
_Trigger:_ /nowplaying

1. Get current track info
2. Format track info
3. Send track info message

_Data touched:_ current_track

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **queue** _(retention: session)_ — List of queued YouTube videos
  - fields: video_id, title, url, added_by, added_at
- **current_track** _(retention: session)_ — Currently playing YouTube video
  - fields: video_id, title, url, duration, is_paused, volume
- **voice_chat_session** _(retention: session)_ — Active voice chat connection
  - fields: chat_id, voice_chat_id, joined_at, bot_user_id

## Integrations

- **YouTube** (required) — Video extraction via yt-dlp
- **Telegram** (required) — Bot API messaging and voice chat integration
- **PyTgCalls** (required) — Telegram voice chat audio streaming
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Start bot
- Stop bot
- View logs
- Clear queue
- Force leave voice chat

## Notifications

- Playing confirmation
- Paused/resumed confirmation
- Skipped confirmation
- Volume set confirmation
- Error messages for invalid URLs or commands

## Permissions & privacy

- Read voice chat status
- Send messages to chat
- Manage voice chats (join/leave)
- None (ephemeral data, no personal data stored)

## Edge cases

- Invalid YouTube URL
- Video not found
- Voice chat not available
- Bot already in voice chat
- Queue empty on skip
- Invalid volume level
- Network timeout during playback
- YouTube video unavailable

## Required tests

- Play valid YouTube URL in voice chat
- Pause and resume playback
- Skip to next track
- Stop playback and leave voice chat
- Adjust volume
- Show queue
- Show current track
- Handle invalid YouTube URL
- Handle voice chat not available
- Handle queue empty on skip

## Assumptions

- yt-dlp is available and functional for YouTube extraction
- PyTgCalls is available and functional for voice chat integration
- Default volume is 50%
- Single track plays at a time
- Bot leaves voice chat automatically on /stop or timeout
