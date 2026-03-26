<div align="center">

  <a href="https://github.com/openclaw/openclaw">
    <picture>
      <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text-dark.svg">
      <img alt="OpenClaw" src="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text.svg" width="320" />
    </picture>
  </a>

  <br/><br/>
  <img src="resy.png" alt="Resy" width="90" />

  <h1>resume-context</h1>

  <p><strong>Never lose context between coding sessions</strong></p>
  <p>Ask OpenClaw about any project in plain English · Powered by <a href="https://github.com/nickleodoen/resume">resume</a> + Redis</p>

  [![OpenClaw Skill](https://img.shields.io/badge/OpenClaw-Skill-CC2929)](https://clawhub.com)
  [![Redis](https://img.shields.io/badge/Redis-Cached-DC382D?logo=redis&logoColor=white)](https://redis.io)
  [![Node.js](https://img.shields.io/badge/Node.js-18+-339933?logo=node.js&logoColor=white)](https://nodejs.org)
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

## Demo

**Session briefing** — `"Claw give me a briefing on resume"`

![Session briefing demo](claw-resume-show.png)

**Project notes** — `"Claw show me my notes for resume"`

![Project notes demo](claw-resume-notes.png)

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

## License

MIT

---

<div align="center">
  <br/>
  <a href="https://github.com/openclaw/openclaw">
    <picture>
      <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text-dark.svg">
      <img alt="Built for OpenClaw" src="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text.svg" width="200" />
    </picture>
  </a>
  <br/>
  <sub>A skill for <a href="https://github.com/openclaw/openclaw">OpenClaw</a> · Built at the OpenClaw Agent Toolkit Hack Day · March 25, 2026 · San Francisco<br/>Sponsors: <strong>Redis</strong> · OpenAI Codex · HackerSquad</sub>
</div>
