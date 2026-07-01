---
name: tubefield
description: Live YouTube data — video stats, transcripts, comments, search, playlists, and channel info via TubeField MCP.
version: 1.0.0
metadata:
  openclaw:
    requires:
      bins:
        - openclaw
    primaryEnv: TUBEFIELD_API_KEY
    envVars:
      - name: TUBEFIELD_API_KEY
        required: false
        description: Optional TubeField API key for headless bearer auth instead of the recommended browser OAuth flow.
    emoji: "📺"
    homepage: https://tubefield.com/setup-mcp
---

# TubeField — live YouTube data

TubeField gives OpenClaw agents live YouTube data through TubeField’s hosted MCP
server at:

```text
https://tubefield.com/mcp
```

Use this skill when a user needs current public YouTube information, including:

- Video metadata and stats
- Video transcripts
- Video comments
- YouTube search results
- Channel metadata and stats
- Channel uploads
- Channel playlists
- Playlist details
- Playlist videos

TubeField reads public YouTube data only. It does not provide access to private
YouTube accounts, private videos, creator analytics, or non-public data.

## External service notice

This skill connects OpenClaw to TubeField’s hosted MCP server. Requests may send
YouTube video ids, channel handles, channel ids, playlist ids, search queries,
and authentication tokens to TubeField.

Only use this skill when the user wants public YouTube data and accepts using
TubeField as the data provider.

## Cost model

TubeField bills per MCP tool call from a prepaid balance. There is no
subscription required.

Current pricing:

| Tool type | Price |
|---|---:|
| Standard tools | $0.003 per call |
| `search_videos` | $0.01 per call |
| `search_result` | $0.01 + $0.006 per result |
| `channel_lookup` | Free |
| `account_balance` | Free |

Before using a pricier call such as `search_result`, consider checking the
balance with `account_balance`.

Users can add balance, get API keys, and review usage at:

```text
https://tubefield.com/setup-mcp
```

## One-time setup, recommended OAuth flow

Use OAuth when a browser is available:

```bash
openclaw mcp add tubefield \
  --url https://tubefield.com/mcp \
  --transport streamable-http \
  --auth oauth

openclaw mcp login tubefield
openclaw mcp probe tubefield
```

`openclaw mcp login tubefield` opens a browser. Sign in or create a TubeField
account to bind the MCP connection to that account and its prepaid balance.

No OAuth scope is required. Tokens refresh automatically after login.

## Alternative setup, headless bearer auth

Use this flow when a browser is not available.

Create an API key in the TubeField dashboard, then set:

```bash
export TUBEFIELD_API_KEY="your_api_key_here"
```

Add the MCP server with a static bearer token:

```bash
openclaw mcp add tubefield \
  --url https://tubefield.com/mcp \
  --transport streamable-http \
  --header "Authorization: Bearer ${TUBEFIELD_API_KEY}"

openclaw mcp probe tubefield
```

## Working with YouTube ids

Use the most specific id available.

### Video ids

YouTube video ids are usually 11 characters.

Examples:

```text
https://www.youtube.com/watch?v=dQw4w9WgXcQ
https://youtu.be/dQw4w9WgXcQ
```

Extract:

```text
dQw4w9WgXcQ
```

Use video ids with:

- `video_detail`
- `video_transcript`
- `video_comments`

### Channel ids and handles

YouTube channel ids usually start with `UC`.

Example:

```text
UC_x5XG1OV2P6uZZ5FSM9Ttw
```

TubeField channel tools also accept YouTube handles.

Examples:

```text
@GoogleDevelopers
GoogleDevelopers
https://www.youtube.com/@GoogleDevelopers
```

Use channel ids or handles with:

- `channel_detail`
- `channel_videos`
- `channel_playlists`

Use `channel_lookup` only when you need to resolve a name, handle, or URL to a
bare `UC...` channel id.

### Playlist ids

YouTube playlist ids usually start with `PL`, though other playlist id formats
also exist.

Use playlist ids with:

- `playlist_detail`
- `playlist_videos`

## Tool selection guidance

Prefer the cheapest tool that answers the user’s request.

When the user gives a specific video URL or id:

1. Extract the video id.
2. Use `video_detail` for metadata and stats.
3. Use `video_transcript` when the user asks about the content of the video.
4. Use `video_comments` when the user asks about audience reaction, comments,
   sentiment, or comment examples.

When the user gives a channel URL, handle, or id:

1. Use the handle or channel id directly with `channel_detail`,
   `channel_videos`, or `channel_playlists`.
2. Use `channel_lookup` only if the channel cannot be resolved directly or the
   user specifically needs the canonical channel id.

When the user gives a playlist URL or id:

1. Extract the playlist id.
2. Use `playlist_detail` for playlist metadata.
3. Use `playlist_videos` for the videos inside the playlist.

When the user asks to find YouTube videos by topic, keyword, or title:

1. Use `search_videos` to find candidate videos.
2. Use `video_detail`, `video_transcript`, or `video_comments` on selected ids
   if deeper analysis is needed.

Use `search_result` only when enriched search results are worth the higher cost,
especially when the user needs search results with details and transcripts in
one step.

## Available tools

| Tool | Purpose | Price |
|---|---|---:|
| `video_detail` | Get video metadata and stats, including title, channel, publish date, duration, views, likes, comments, and tags. | $0.003 |
| `video_comments` | Get top-level comments ranked by relevance. Supports `limit` from 1 to 1000. | $0.003 |
| `video_transcript` | Get the full transcript text for a video id. | $0.003 |
| `channel_detail` | Get channel metadata and stats by channel id or handle. | $0.003 |
| `channel_videos` | Get a channel’s uploads by channel id or handle. Supports `limit` from 1 to 1000. | $0.003 |
| `channel_playlists` | Get a channel’s public playlists by channel id or handle. | $0.003 |
| `channel_lookup` | Resolve a channel handle, name, or URL to a bare `UC...` channel id. | Free |
| `playlist_detail` | Get playlist metadata and stats by playlist id. | $0.003 |
| `playlist_videos` | Get videos in a playlist. Supports `limit` from 1 to 1000. | $0.003 |
| `search_videos` | Search YouTube by keyword and return video ids with basic metadata. Supports optional region and language bias. | $0.01 |
| `search_result` | Search YouTube and return enriched results with detail and transcript data. | $0.01 + $0.006/result |
| `account_balance` | Check the account’s remaining prepaid balance. Takes no arguments. | Free |

Paginated tools with `limit` from 1 to 1000 page internally at a flat price.
The user does not need to handle page tokens.

If an id is unresolvable, the result may be null and the call may still be
billed.

## Recommended agent behavior

Be cost-aware and transparent.

Before making multiple paid calls, especially transcript, comments, or enriched
search calls, briefly explain what data is needed and choose the smallest set of
calls that can answer the user’s request.

For example:

- Use `video_detail` before deeper video analysis.
- Use `video_transcript` only when the actual spoken content matters.
- Use `video_comments` only when comments are relevant.
- Use `search_videos` before `search_result` unless enriched results are clearly
  necessary.
- Use `account_balance` before expensive or broad searches when balance may
  matter.

When summarizing transcripts or comments, make clear whether the answer is based
on video metadata, transcript content, comments, or search results.

Do not claim access to private YouTube data, creator Studio analytics, revenue
data, private comments, private videos, or account-specific YouTube information.

## Example workflows

### Get stats for a video

User asks:

```text
What are the stats for this YouTube video?
https://www.youtube.com/watch?v=dQw4w9WgXcQ
```

Agent should:

1. Extract `dQw4w9WgXcQ`.
2. Call `video_detail`.
3. Summarize the returned metadata and stats.

### Summarize a video

User asks:

```text
Summarize this video.
```

Agent should:

1. Extract the video id.
2. Call `video_detail`.
3. Call `video_transcript`.
4. Summarize the transcript and mention key metadata when useful.

### Analyze audience reaction

User asks:

```text
What are people saying in the comments?
```

Agent should:

1. Extract the video id.
2. Call `video_comments`.
3. Summarize themes, notable reactions, and caveats.

### Research videos on a topic

User asks:

```text
Find recent YouTube videos about AI agents.
```

Agent should:

1. Call `search_videos` with the topic.
2. Present relevant candidates.
3. Only call transcript or detail tools for selected videos when deeper analysis
   is needed.

### Check account balance

User asks:

```text
How much TubeField balance do I have left?
```

Agent should call `account_balance`.

## Troubleshooting

If `openclaw mcp probe tubefield` does not list tools:

1. Confirm the MCP server URL is exactly:

   ```text
   https://tubefield.com/mcp
   ```

2. Confirm the transport is:

   ```text
   streamable-http
   ```

3. For OAuth setup, run:

   ```bash
   openclaw mcp login tubefield
   ```

4. For headless setup, confirm `TUBEFIELD_API_KEY` is set and the Authorization
   header is passed exactly as:

   ```bash
   --header "Authorization: Bearer ${TUBEFIELD_API_KEY}"
   ```

5. Confirm the TubeField account has prepaid balance for paid tools.
