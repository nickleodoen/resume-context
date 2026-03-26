<div align="center">
  <img src="resy.png" alt="Resy the jellyfish" width="120" />

  # resume-context

  **Ask OpenClaw about your coding sessions and project notes**

  Powered by [resume](https://github.com/nickleodoen/resume) · Cached by Redis

  [![OpenClaw Skill](https://img.shields.io/badge/OpenClaw-Skill-red)](https://clawhub.com)
  [![Redis](https://img.shields.io/badge/Redis-Cached-DC382D)](https://redis.io)
  [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

</div>

---

Never lose context between coding sessions. Ask OpenClaw in plain English from any channel:

"Claw give me a briefing on resume"
"Claw what was I working on for my-app?"
"Claw show me my notes for the api project"

You get back a real, LLM-generated summary of what you were working on, what you finished,
and what to do next — sourced directly from your local session data, not a memory system.

---

## How it works

```
You
 │  "Claw give me a briefing on resume"
 ▼
OpenClaw
 │  skill triggered by message pattern
 ▼
resume-context
 │  cache key: resume:show:/path/to/project
 ▼
Redis
 ├── HIT  → return instantly (<100ms)
 └── MISS → run `resume show`
              │  calls Anthropic LLM (~12s)
              └─ write to Redis, TTL 5 min
 ▼
OpenClaw
 │  "Resume Project Briefing:
 │   You're updating docs to showcase the TUI dashboard.
 │   Progress: Updated README, replaced VSCode screenshot.
 │   Next: Verify ClawHub integration end-to-end."
 ▼
You
```

First request: ~12 seconds (LLM call via `resume`). Every repeat in the next 5 minutes: under 100ms (Redis cache hit).

---

## Install
```bash
openclaw skills install resume-context
```

Add to `~/.openclaw/openclaw.json`:
```json
{
  "skills": {
    "entries": {
      "resume-context": {
        "enabled": true,
        "env": {
          "REDIS_URL": "redis://default:password@host:port",
          "RESUME_CACHE_TTL": "300"
        }
      }
    }
  }
}
```

Restart the gateway:
```bash
openclaw gateway restart
```

---

## Requirements

| Requirement | Setup |
|---|---|
| [resume CLI](https://github.com/nickleodoen/resume) | `cargo install --git https://github.com/nickleodoen/resume` |
| Redis | [Redis Cloud free tier](https://cloud.redis.io/) — takes 2 minutes |
| `ANTHROPIC_API_KEY` | Used by `resume` to generate briefings |
| Node.js 18+ | For the bridge script |

---

## Usage

Start a resume session in your project before asking:
```bash
cd ~/your-project
resume          # starts watching your session
```

Then in OpenClaw:

| You say | What happens |
|---|---|
| `Claw give me a briefing on resume` | Session briefing for the resume project |
| `Claw what was I working on for resume?` | Same |
| `Claw show me my notes for resume` | Project notes for resume |
| `Claw catch me up on my-app` | Session briefing for my-app |
| `Claw what notes do I have on resume?` | Notes only |

---

## Files

resume-context/
├── SKILL.md        — OpenClaw skill definition: trigger phrases + agent instructions
├── resume-mcp.js   — Node.js bridge: Redis cache layer + resume CLI runner
└── package.json    — Single dependency: redis ^4.7.0

**SKILL.md** teaches OpenClaw when to trigger this skill and exactly how to run it.
The `description` frontmatter is injected into the model's system prompt — that's what
makes "Claw give me a briefing on resume" route here instead of a generic memory search.

**resume-mcp.js** is the Redis caching bridge. It checks the cache first, falls back to
the live `resume` CLI on a miss, writes the result back with a configurable TTL, and
returns structured JSON. If Redis is unreachable it degrades gracefully to live execution.

---

## Environment variables

| Variable | Required | Default | Description |
|---|---|---|---|
| `REDIS_URL` | ✅ | — | Redis connection string |
| `RESUME_CACHE_TTL` | ❌ | `300` | Cache TTL in seconds |
| `RESUME_BIN` | ❌ | `~/.cargo/bin/resume` | Path to resume binary |

---

## Built at

OpenClaw Agent Toolkit Hack Day · March 25, 2026 · San Francisco

Sponsors: **Redis** · OpenAI Codex · HackerSquad

---

## License

MIT
