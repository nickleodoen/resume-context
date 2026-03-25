# resume-context

> Ask OpenClaw about your coding sessions and project notes — powered by [resume](https://github.com/nickleodoen/resume) + Redis.

Built at the **OpenClaw Agent Toolkit Hack Day** · March 25, 2026

[![OpenClaw Skill](https://img.shields.io/badge/OpenClaw-Skill-red)](https://clawhub.com)
[![Redis](https://img.shields.io/badge/Redis-Cache-DC382D)](https://redis.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## What it does

Never lose context between coding sessions. Ask OpenClaw about any project in plain English:

"Claw give me a briefing on resume"
"Claw what was I working on for my-app?"
"Claw show me my notes for the api project"
"Claw catch me up on resume"

OpenClaw runs the `resume` CLI in your project directory, generates a plain-English briefing of what you worked on, what you finished, and what to do next — then caches it in Redis so repeat questions answer in under 100ms.

## Demo

![Demo](https://hel1.your-objectstorage.com/hackersquadcontent/project-recordings/project_rec_cmn6lg9oc0105nv0khkmgjd7e-2026-03-25T230229.mp4)

You: "Claw give me a briefing on resume"
Resume Project Briefing:
You're updating the resume project README to showcase the TUI dashboard
and clarify installation/usage.
Progress:

Updated README with TUI dashboard screenshot for live monitoring
Replaced VSCode screenshot with updated TUI interface image

Next step: Verify ClawHub skill integration and test end-to-end workflow

## Architecture

You → OpenClaw → resume-context skill
↓
Redis GET (cache hit → <100ms)
↓ miss
resume show / resume notes
↓
LLM briefing (Anthropic/Ollama)
↓
Redis SET (5 min TTL)
↓
Response back to you

## Install
```bash
openclaw skills install resume-context
```

Then add to `~/.openclaw/openclaw.json`:
```json
{
  "skills": {
    "entries": {
      "resume-context": {
        "enabled": true,
        "env": {
          "REDIS_URL": "redis://default:password@host:port"
        }
      }
    }
  }
}
```

## Requirements

| Requirement | Notes |
|---|---|
| [resume](https://github.com/nickleodoen/resume) | `cargo install --git https://github.com/nickleodoen/resume` |
| Redis | [Redis Cloud free tier](https://cloud.redis.io/) works great |
| Node.js 18+ | For the bridge script |
| `ANTHROPIC_API_KEY` | Used by resume for briefing generation |

## How it works

1. OpenClaw receives your message and matches it to this skill via the `description` field in `SKILL.md`
2. The skill extracts the project name and finds the directory on disk
3. Checks Redis — cache hit returns instantly with TTL remaining
4. Cache miss: runs `resume show` or `resume notes` in your project directory
5. `resume` calls your LLM to generate the briefing (~10-15s first time)
6. Result cached in Redis for 5 minutes
7. Response returned to you in the channel you asked from

## Why Redis?

Without Redis, every question re-runs the LLM — 10-15 seconds every time. With Redis, repeat questions in the same session answer in under 100ms. Redis also gives the skill **durable memory** — briefings persist across OpenClaw restarts and are available instantly when you return to a project.

## Environment variables

| Variable | Required | Default | Description |
|---|---|---|---|
| `REDIS_URL` | ✅ | — | Redis connection string |
| `RESUME_CACHE_TTL` | ❌ | `300` | Cache TTL in seconds |
| `RESUME_BIN` | ❌ | `~/.cargo/bin/resume` | Path to resume binary |

## File structure

resume-context/
├── SKILL.md        # OpenClaw skill definition — trigger phrases, instructions
├── resume-mcp.js   # Node.js bridge — shells out to resume CLI, handles Redis cache
└── package.json    # Dependencies (redis ^4.7.0)

## Usage examples

| You say | What happens |
|---|---|
| "Claw give me a briefing on resume" | `resume show` in the resume project |
| "Claw what was I working on for resume?" | Same as above |
| "Claw show me my notes for resume" | `resume notes` in the resume project |
| "Claw catch me up on my-app" | `resume show` in the my-app project |

## License

MIT
