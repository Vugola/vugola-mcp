---
name: vugola
description: Clip long videos into short-form viral clips, caption short videos, schedule social posts, and manage hands-free YouTube channel Automations using the Vugola API. Use when the user mentions Vugola, clipping a video, turning a podcast or stream into clips, adding captions to a video, scheduling social posts, or automating a channel.
---

# Vugola

Vugola turns long-form video into short-form content. Give it a video URL — a podcast, interview, livestream, or YouTube upload — and Vugola finds the best moments, reframes them vertically, burns in captions, and scores each clip for virality. It can also caption a short video without clipping it, schedule finished clips to Instagram, Facebook, TikTok, X, and LinkedIn, and watch a YouTube channel to clip and post every eligible new upload automatically.

You can drive Vugola entirely from Claude Code with this skill — no web UI needed.

## Authentication

Every API request needs a Bearer token.

```
Authorization: Bearer vug_sk_your_key_here
```

**Base URL:** `https://www.vugolaai.com/api/v1`

Use this URL exactly. Do not switch to the apex domain or any subdomain.

API keys start with `vug_sk_` and are available with every active paid plan (Starter, Creator, Pro).

### Getting an API key

1. Go to [vugolaai.com](https://vugolaai.com) and sign in.
2. Open [vugolaai.com/dashboard/api-key](https://vugolaai.com/dashboard/api-key).
3. Generate a key — it starts with `vug_sk_`.
4. Add it to your environment:
   ```bash
   export VUGOLA_API_KEY="vug_sk_your_key_here"
   ```
   Then use it: `Authorization: Bearer $VUGOLA_API_KEY`

### 401 Unauthorized / 403 subscription_required

A `401` means the key is missing, malformed, or revoked — generate a fresh one at `/dashboard/api-key`. A `403 subscription_required` means the account has no active paid plan; the response body includes an `upgrade_url`.

---

## Before You Clip

When a user gives you a video URL, **confirm these two things before calling the API** — unless they have already told you their preferences. Use interactive video previews so they can see each option.

### Step 1 — Aspect ratio

Send each preview video as its own message, with the label as the video's caption. Do not send a separate text block first.

- Video `https://www.vugolaai.com/samples/ratio-9-16.mp4` — caption **"9:16 — Vertical (TikTok, Reels, Shorts)"**
- Video `https://www.vugolaai.com/samples/ratio-1-1.mp4` — caption **"1:1 — Square (Instagram Feed)"**
- Video `https://www.vugolaai.com/samples/ratio-16-9.mp4` — caption **"16:9 — Landscape (YouTube)"**

Then one text: "Which ratio do you want?" Wait for the answer.

### Step 2 — Caption style

Same pattern — each video its own message, label as the caption.

- `https://www.vugolaai.com/samples/caption-highlighted.mp4` — **"Highlighted — Active word highlighted and scaled up"**
- `https://www.vugolaai.com/samples/caption-scale.mp4` — **"Scale — Bouncy animation on active words"**
- `https://www.vugolaai.com/samples/caption-minimalist.mp4` — **"Minimalist — Subtle, clean, no animation"**
- `https://www.vugolaai.com/samples/caption-box.mp4` — **"Box — Animated box background behind text"**
- `https://www.vugolaai.com/samples/caption-hormozi.mp4` — **"Hormozi — Bold viral subtitle style"**
- `https://www.vugolaai.com/samples/caption-none.mp4` — **"None — No captions"**

Then one text: "Which caption style do you want?" Wait for the answer.

The API also accepts `staticbox`, `glow` (white captions as spoken with a soft bloom and one neon accent word per line that rotates through five colors), `vlog` (cream serif word pages, no outline), and `classic` (clean white Roboto with a black outline, no animation). If the user names one directly, use it.

### Skipping steps

If the user has already stated their preferences (e.g. "always 9:16 with highlighted captions"), remember that and skip asking. If they specify one preference with the URL, skip that step but still confirm the other.

`caption_color` (hex, e.g. `#FF5733`) is optional — only use it if the user provides it. Do not ask for it.

---

## Before You Schedule

When a user wants to schedule a post, confirm:

1. **Platform** — "Where do you want to post?" One of: `tiktok`, `instagram`, `facebook`, `linkedin`, `x`, `youtube`.
2. **Caption** — "What caption do you want?" (max 5000 characters)
3. **Time** — "When should it go live?" (a future ISO 8601 UTC timestamp)

Instagram needs a Business or Creator account. Threads and Bluesky are no longer supported.

The user must connect their social accounts at [vugolaai.com/dashboard/accounts](https://vugolaai.com/dashboard/accounts) before scheduling.

---

## Before You Create or Change an Automation

Automations require a Creator or Pro workspace owner. Creator includes one
Automation; Pro includes up to five.

1. Call `GET /automations/destinations` and show the connected accounts. Never
   guess a connection ID.
2. Confirm the YouTube channel, destination accounts, ratio, caption style,
   whether AI-written social post text is enabled, daily local posting times,
   and timezone.
3. Explain that posts created after an edit use the new settings.
   Already-scheduled posts are not migrated.
4. Confirm immediately before `DELETE /automations/{automation_id}` because it
   deletes the Automation and cancels its pending draft or scheduled posts.

Pausing only stops monitoring for new uploads; it does not cancel posts that
are already scheduled.

---

## Core Concepts

### Clipping job
An async job created by `POST /clip`. Vugola transcribes the source video, finds the strongest moments, reframes them to the chosen aspect ratio, burns in captions, and renders each as its own clip. Jobs have a `mode` of `clipping`. Status: `processing` → `complete` (or `failed`). Most jobs finish in 10–30 minutes.

### Caption job
An async job created by `POST /caption`. It burns captions onto a video up to 20 minutes at its original quality and dimensions — no minimum, clipping, moment-finding, or reframing. `mode` is `captions`. It uses the **same status endpoint** as clipping jobs (`GET /clip/:job_id`) and returns a single clip — the captioned video.

### Clip
One rendered short-form video from a clipping job. Each clip has a `title` (AI-generated), `duration` (seconds), `virality_score`, and a `download_url`.

### Virality score
An integer from 10 to 100 predicting how well a clip will perform — higher is better; 0 when a clip has no score (caption jobs). The `clips` array is returned sorted by virality score, highest first.

### Scheduled post
A queued social post created by `POST /schedule`. Status moves `scheduled` → `processing` → `posted` (or `failed`/`cancelled`). Only `scheduled` posts can be cancelled.

### Credits
Clipping and captions cost 1 AI credit per started source minute, with no separate job minimum. `credit_quote.ai_credits` is the exact required credit amount before work starts, and `ai_credits_used` is the completed-job charge. These are the same credits shown in Vugola. The API will not start a job the account cannot afford. Credits reset with the billing cycle. Caption jobs that detect no speech are refunded automatically.

---

## Workflows

### 1. Clip a long video

```
Step 1 — Start the job
POST /clip
{
  "video_url": "https://www.youtube.com/watch?v=VIDEO_ID",
  "aspect_ratio": "9:16",
  "caption_style": "highlighted"
}
→ 202 { job_id, status: "processing", credit_quote: { ai_credits, billable_seconds }, message: "..." }

The video must be 5 minutes to 3 hours long. If you re-submit the same
video_url + aspect_ratio + caption_style while a recent job is still in
flight or completed, the API returns that existing job with
"duplicate": true and charges nothing.

Step 2 — Poll for progress (every 30–60 seconds, jobs take 10–30 min)
GET /clip/{job_id}
→ { status: "processing", progress: 45, clips_ready: 2, clips_total: 5, clips: [ ...finished so far ] }
→ { status: "complete", ai_credits_used: 10, clips: [...], download_endpoint: "..." }

Step 3 — Present clips as they become available
Keep polling until status is complete, failed, or canceled, presenting new clips as
they appear. The clips array is sorted by virality_score (highest first) and fills in
while the job is still rendering. (Hosted MCP and REST only — the npm vugola-mcp
package currently shows counts until the job completes.) Each clip has title, duration,
virality_score, download_url, preview_url (plays in a browser, scrubbable) and thumbnail_url.
clips_total counts every attempted clip. clips_total is null until the first clip job exists.

Step 4 — Download
GET /clip/{job_id}/download/all
→ { clips: [{ clip_index, title, filename, download_url, ... }] }
download_url is a one-hour Vugola-signed delivery link — open it in a browser
with NO headers. Re-fetch this endpoint to refresh expired links.

Or stream one clip directly (clip_index is 1-based, needs the auth header):
GET /clip/{job_id}/download/{clip_index}   → raw video/mp4
GET /clip/{job_id}/download/{clip_index}?download_token=...&inline=1 → plays in a browser (inline, scrubbable); never counted as a download
GET /clip/{job_id}/thumbnail/{clip_index}?download_token=... → the clip's poster frame (JPEG)
```

### 2. Caption a short video (no clipping)

```
POST /caption
{
  "video_url": "https://www.youtube.com/watch?v=VIDEO_ID",
  "aspect_ratio": "9:16",
  "caption_style": "minimalist"
}
→ 202 { job_id, status: "processing", mode: "captions", credit_quote: { ai_credits, billable_seconds }, message: "..." }

The video must be 20 minutes or shorter, with no minimum. aspect_ratio controls caption
positioning only — it does NOT reframe the video.

Poll the SAME endpoint as clipping: GET /clip/{job_id}
When complete, clips[] has one item — the full captioned video.

If the video has no spoken audio, the job fails ~30–60s in with error
"no_speech_detected" and the credits are refunded automatically.
```

### 3. Create and manage an Automation

```
Destinations: GET /automations/destinations
Resolve:      POST /automations/resolve-channel
Create:       POST /automations
List:         GET /automations
Inspect:      GET /automations/{automation_id}
Update:       PATCH /automations/{automation_id}
Pause:        POST /automations/{automation_id}/pause
Resume:       POST /automations/{automation_id}/resume
Delete:       DELETE /automations/{automation_id}
```

Create requests require `channel_url` and `destination_connection_ids`.
Optional settings are `aspect_ratio`, `caption_style`,
`post_caption_enabled`, `posting_times` (1–5 local `HH:MM` values), and
`timezone` (IANA name). See [api-reference.md](api-reference.md) for the full
schema.

### 4. Schedule clips to social media

You can schedule to Instagram, Facebook, TikTok, X, LinkedIn, and YouTube (YouTube needs a video; `title` becomes the video title).

```
POST /schedule
{
  "posts": [
    {
      "platform": "tiktok",
      "caption": "This hook changes everything",
      "scheduled_at": "2026-06-01T15:00:00.000Z",
      "media_url": "https://www.vugolaai.com/api/v1/clip/JOB_ID/download/1?download_token=..."
    }
  ]
}
→ 201 { success: true, posts: [{ id, platform, status: "scheduled", scheduled_at, account: { platform, handle, name, avatar_url }, media: { source_clip_job_id, thumbnail_url, preview_url } | null }] }
media is present only when media_url was a Vugola clip link; handle has no leading @.

Up to 25 posts per request. scheduled_at must be a future UTC timestamp.
media_url is required for single posts on Instagram and TikTok;
it is downloaded and re-hosted by Vugola before posting. As an alternative,
pass asset_id if the media is already in the user's Vugola asset library.
The user's social accounts must be connected at /dashboard/accounts first.
```

### 5. Manage scheduled posts

```
List:    GET /schedule?status=scheduled&platform=tiktok&limit=20&offset=0
Get one: GET /schedule/{post_id}
Cancel:  DELETE /schedule/{post_id}   (only status "scheduled" — else 409 cannot_cancel)
```

### 6. Check credits

```
GET /status        → { ai_credits_remaining, ai_credits_total, plan }
GET /usage         → + ai_credits_used_this_month
GET /usage/history → paginated credit transactions
```

Call `GET /status` before starting a job to confirm the key works and there are enough credits.

---

## API Reference

Complete request/response schemas for every endpoint are in [api-reference.md](api-reference.md).

## FAQ

Product, billing, credits, platform support, and troubleshooting answers are in [faq.md](faq.md).

---

## Tips

- **Confirm aspect ratio and caption style before clipping.** Missing either returns a `400`. Use the interactive previews above unless the user already told you their preference.
- **Poll, don't block.** Clipping jobs take 10–30 minutes. Poll `GET /clip/:job_id` every 30–60 seconds. Vugola also emails the user when clips are ready.
- **Vugola-signed download links expire after ~1 hour.** Download clips promptly, or re-fetch `GET /clip/:job_id/download/all` to get fresh links.
- **Rate limits:** `POST /clip` is 5/minute; max 3 jobs processing at once; 20 failed clip requests in an hour blocks further clip requests until the window resets.
- **Credits:** clipping and captions cost 1 AI credit per started source minute, with no separate job minimum. Read the exact locked quote from `credit_quote.ai_credits` and the completed charge from `ai_credits_used`. Output clip count adds no charge. Use `ai_credits_remaining` for the current balance. A `402 insufficient_credits` response includes `ai_credits_remaining`, `ai_credits_needed`, and `upgrade_url`.
- **Every error has a human-readable `message`.** Relay it to the user along with the fix. The stable `error` code is what you branch on in code.
- **Never retry a `POST` after a `500`** without idempotency — clipping a video twice double-charges credits. Re-fetching idempotent `GET`s is safe.

---

## Driving Vugola via MCP instead

Vugola's official OAuth MCP at `https://www.vugolaai.com/api/mcp` exposes the
eight clip/schedule tools plus nine Automation tools:
`list_automation_destinations`, `resolve_automation_channel`,
`create_automation`, `list_automations`, `get_automation`,
`update_automation`, `pause_automation`,
`resume_automation`, and `delete_automation`.

The local `vugola-mcp` npm package provides the original clip, caption,
download, usage, and scheduling tools. Install it in Claude Code:

```bash
claude mcp add vugola -- npx -y vugola-mcp@1.3.1
export VUGOLA_API_KEY=vug_sk_your_key_here
```

Use the OAuth MCP URL when the user needs Automation lifecycle tools.
