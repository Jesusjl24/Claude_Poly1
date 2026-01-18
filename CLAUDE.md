# CLAUDE.md — Project Rules for Claude Code

## Project Context

This is a **Polymarket Paper Trading Bot** — an educational tool for practicing prediction market trading without financial risk.

**Owner:** Jesus (solo developer, Project Support Officer at EnergyCo NSW)
**Purpose:** Learn prediction market dynamics, practice trade patterns, experiment with automated trading

---

## GSD Protocol Active

This project uses the **Get Shit Done (GSD)** protocol for spec-driven development.

**Key commands:**
- `/gsd:progress` — Check current status
- `/gsd:discuss-phase N` — Capture implementation preferences
- `/gsd:plan-phase N` — Create atomic task plans
- `/gsd:execute-phase N` — Execute with fresh subagent contexts
- `/gsd:verify-work N` — Manual testing and fixes
- `/gsd:pause-work` — Create handoff for session end
- `/gsd:resume-work` — Continue from last session

**Run Claude Code with:** `claude --dangerously-skip-permissions`

---

## Code Style

### Python (Backend)
- Python 3.11+ with type hints
- FastAPI for REST endpoints
- Pydantic for validation
- SQLite with raw SQL (no ORM complexity)
- APScheduler for background jobs
- Black formatting, 88 char line length
- Imports: stdlib → third-party → local (separated by blank lines)

### JavaScript/React (Frontend)
- React 18 with hooks (no class components)
- Tailwind CSS for styling (no CSS files)
- Vite for bundling
- ESM imports only
- Prettier formatting

### General
- Descriptive variable names (no single letters except loop indices)
- Comments explain WHY, not WHAT
- No console.log/print in production code — use proper logging
- Error messages should be actionable

---

## File Organization

```
polymarket-paper-trader/
├── .planning/              # GSD artifacts (auto-generated)
├── backend/app/            # FastAPI application
├── frontend/src/           # React application
├── data/                   # SQLite database (Docker volume)
└── docker-compose.yml      # Container orchestration
```

---

## Testing Approach

- **Backend:** Manual API testing with curl/httpie during development
- **Frontend:** Visual testing in browser
- **Integration:** End-to-end flow verification after each phase
- **No unit test framework required** — keep it simple for MVP

---

## API Design

- REST endpoints under `/api/` prefix
- JSON request/response bodies
- HTTP status codes: 200 (success), 201 (created), 400 (bad request), 404 (not found), 500 (server error)
- Timestamps in ISO 8601 format
- Money amounts as floats (paper trading, precision not critical)

---

## Git Workflow

**Commit format:** `{type}({phase}-{plan}): {description}`

Types:
- `feat` — New feature
- `fix` — Bug fix
- `docs` — Documentation
- `chore` — Config, dependencies
- `refactor` — Code cleanup

**Never use:** `git add .` — stage files individually

---

## Environment Variables

```bash
# Backend
POLYMARKET_API_URL=https://gamma-api.polymarket.com
FETCH_INTERVAL_SECONDS=300
STARTING_BALANCE=100.0
DATABASE_PATH=/app/data/trades.db

# Frontend
VITE_API_URL=http://localhost:8000/api
```

---

## Docker Configuration

- Backend: Python 3.11-slim base image, port 8000
- Frontend: Node 20-alpine for build, nginx for serve, port 3000
- Data volume: `./data:/app/data` for SQLite persistence

---

## Key Decisions Log

| Decision | Rationale |
|----------|-----------|
| SQLite over PostgreSQL | Simpler setup, no separate container, sufficient for single-user |
| APScheduler over Celery | Lightweight, no Redis/RabbitMQ dependency |
| No authentication | Single user, local deployment only |
| Hold until resolution | Simpler than active exit strategies for MVP |
| React over Vue | User preference, existing experience |

---

## Deviation Rules (GSD)

When executing tasks, apply automatically:

1. **Bug?** → Fix immediately, document in summary
2. **Missing critical functionality?** → Add immediately (validation, error handling, security)
3. **Blocking issue?** → Fix to unblock (missing deps, config errors)
4. **Architectural change needed?** → STOP, return checkpoint for user decision

---

## Success Metrics

- Bot successfully fetches from Polymarket API
- Paper trades execute with correct calculations
- Portfolio metrics update accurately
- Dashboard renders all components
- Docker Compose brings up entire stack
- State persists across restarts

---

## Reference Implementation

See `POLYMARKET_MEGA_PROMPT.md` for full specification including:
- API endpoint definitions
- Dashboard component specifications
- Phase-by-phase execution plan
- Architecture diagram

---

*GSD Protocol: "The complexity is in the system, not in your workflow."*
