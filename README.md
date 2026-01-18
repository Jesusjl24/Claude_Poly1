[PROJECT_README.md](https://github.com/user-attachments/files/24693552/PROJECT_README.md)
# Polymarket Paper Trading Bot — Starter Kit

A paper trading bot for Polymarket prediction markets built with the GSD (Get Shit Done) protocol.

## What's Included

| File | Purpose |
|------|---------|
| `POLYMARKET_MEGA_PROMPT.md` | Complete project specification for Claude Code |
| `CLAUDE.md` | Project rules and conventions |
| `DASHBOARD_REFERENCE.jsx` | Interactive React dashboard reference implementation |
| `setup.sh` | Quick start script to initialize GSD |

## Quick Start

### Prerequisites
- Node.js 18+
- Docker Desktop (or OrbStack on macOS)
- Claude Code CLI

### Setup

```bash
# 1. Make setup script executable and run it
chmod +x setup.sh
./setup.sh

# 2. Start Claude Code with permissions bypass
claude --dangerously-skip-permissions

# 3. Initialize the project using GSD
/gsd:new-project
```

When GSD asks discovery questions, reference `POLYMARKET_MEGA_PROMPT.md` for the answers.

### Development Workflow

```bash
# Check progress at any time
/gsd:progress

# For each phase (1-12):
/gsd:discuss-phase N    # Capture your preferences
/gsd:plan-phase N       # Create task plans
/gsd:execute-phase N    # Execute autonomously
/gsd:verify-work N      # Test and verify

# Pause/resume between sessions
/gsd:pause-work
/gsd:resume-work
```

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                 Docker Compose Stack                    │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────┐         ┌──────────────┐             │
│  │   Frontend   │ ──API──▶│   Backend    │             │
│  │  React/Vite  │         │   FastAPI    │             │
│  │   :3000      │         │    :8000     │             │
│  └──────────────┘         └──────┬───────┘             │
│                                  │                      │
│                                  ▼                      │
│  ┌──────────────┐         ┌──────────────┐             │
│  │  Polymarket  │◀────────│   SQLite     │             │
│  │  Gamma API   │         │   (volume)   │             │
│  └──────────────┘         └──────────────┘             │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

## Key Features

- **Paper Trading** — Simulate trades with fake money ($100 starting balance)
- **Manual Approval** — Bot fetches opportunities, you approve with custom side/amount
- **Editable Trades** — Override bot's suggestion, compare YES vs NO sides
- **Dual Controls** — Stop bot entirely OR just pause fetching while monitoring positions
- **Full History** — Track wins, losses, overall performance
- **Debug Logs** — See what the bot is doing, catch errors early

## Phases Overview

| Phase | Description |
|-------|-------------|
| 1 | Project Foundation (Docker, skeletons) |
| 2 | Data Layer (SQLite, models) |
| 3 | Polymarket Integration (API client) |
| 4 | Paper Broker (trade simulation) |
| 5 | Bot Engine (scheduler, state) |
| 6 | API Endpoints (REST interface) |
| 7 | Dashboard Layout (components) |
| 8 | Dashboard Opportunities (edit mode) |
| 9 | Dashboard Positions & History |
| 10 | Dashboard Polish (settings, logs) |
| 11 | Integration Testing |
| 12 | Documentation & Cleanup |

## Reference

- **Dashboard Design:** Open `DASHBOARD_REFERENCE.jsx` in Claude.ai to see the interactive prototype
- **Full Spec:** See `POLYMARKET_MEGA_PROMPT.md` for complete requirements
- **Code Rules:** See `CLAUDE.md` for conventions

## License

MIT — Educational/research use only.

---

*Built with GSD Protocol — "The complexity is in the system, not in your workflow."*
