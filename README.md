# Vugola MCP Server & Agent Skill

Official Model Context Protocol (MCP) connector and agent skill for **[Vugola](https://www.vugolaai.com)** — the AI video editing and clipping agent for creators and brands.

Clip long-form videos (YouTube, Twitch, Kick, podcasts, interviews) into vertical Shorts, Reels, and TikToks, burn animated captions, schedule posts across 6 social platforms, and run hands-free YouTube channel Automations directly from **Claude**, **ChatGPT**, **Cursor**, **Claude Code**, **OpenClaw**, or **Hermes Agent**.

## Remote MCP Endpoint

```text
https://www.vugolaai.com/api/mcp
```

- **ChatGPT MCP connector:** `https://www.vugolaai.com/api/mcp-openai`
- **OAuth sign-in:** Connect from Claude, ChatGPT, or Cursor with one click — no API key to manage.
- **Local / npm server:** `npx vugola-mcp` (uses a `vug_sk_` API key from `https://www.vugolaai.com/dashboard/api-key`, included on every paid plan).

## Quick Setup

### Claude / Cursor / Windsurf / Cline
Add the remote server URL to your MCP config (see [`mcp.json`](./mcp.json)):

```json
{
  "mcpServers": {
    "vugola": {
      "url": "https://www.vugolaai.com/api/mcp"
    }
  }
}
```

### Claude Code / Coding Agents
Install or reference [`SKILL.md`](./SKILL.md) (canonical live copy: `https://www.vugolaai.com/vugola-skill.md`) and set:

```bash
export VUGOLA_API_KEY="vug_sk_your_key_here"
```

## What You Can Do via MCP

- **AI Clipping:** Pass any YouTube, Twitch, Kick, or direct video URL to extract high-retention moments in `9:16`, `1:1`, or `16:9` with animated captions.
- **Captioning:** Add styled burned-in captions to existing short videos.
- **Multi-Platform Scheduling:** Schedule and publish finished clips to TikTok, Instagram, YouTube, X, LinkedIn, and Facebook.
- **Hands-Free YouTube Automations:** Watch a YouTube channel and automatically clip and publish eligible new uploads (Creator & Pro plans).

## Pricing & Plans

Vugola is a paid SaaS platform (no free tier; access starts after payment). Every paid plan includes Studio editor with 1080p export, AI clipping, unlimited scheduled posts, and both REST API & MCP access:

| Plan | Monthly | Annual | AI Credits / mo | Social Platforms | Automations |
| --- | --- | --- | --- | --- | --- |
| **Starter** | $19.99/mo | $15.00/mo ($179.99/yr) | 50 | 2 | — |
| **Creator** | $49.99/mo | $39.17/mo ($469.99/yr) | 300 | 4 | 1 channel |
| **Pro** | $99.99/mo | $79.17/mo ($949.99/yr) | 800 | All 6 | Up to 5 channels |

*1 AI credit = 1 AI prompt, AI edit, or clipping minute.*

## Links

- **MCP Setup Guide:** https://www.vugolaai.com/mcp
- **REST API Docs:** https://www.vugolaai.com/docs
- **Pricing:** https://www.vugolaai.com/pricing
- **Comparisons (vs OpusClip, Descript, Submagic, Vizard, Klap, CapCut):** https://www.vugolaai.com/compare
- **Official AI Reference:** https://www.vugolaai.com/ai-info
- **llms.txt:** https://www.vugolaai.com/llms.txt
- **iPhone App:** https://www.vugolaai.com/app
