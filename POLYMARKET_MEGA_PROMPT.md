# Polymarket Paper Trading Bot — Claude Code Mega Prompt

## Instructions for Claude Code

You are building a **Polymarket Paper Trading Bot** using the **GSD (Get Shit Done)** protocol. This is a spec-driven development system optimized for solo developer + Claude workflow.

**Critical:** Run with `claude --dangerously-skip-permissions` to enable uninterrupted autonomous execution.

---

## GSD Protocol Setup

Before starting, initialize GSD in this project:

```bash
npx get-shit-done-cc --local
```

Then run `/gsd:new-project` and use the specification below to answer the discovery questions.

---

## PROJECT SPECIFICATION

### What This Is

A paper trading bot for Polymarket prediction markets that fetches opportunities via API, queues them for user approval, simulates trades with fake money, and provides a dashboard to monitor bot status, positions, and performance. Educational/research tool for learning prediction market trading without financial risk.

### Core Value

**Enable practice trading on Polymarket with zero financial risk while building intuition for prediction markets.**

### Target User

Solo developer (Jesus) learning prediction market dynamics, practicing trade execution patterns, and experimenting with automated trading strategies before graduating to real money.

---

## REQUIREMENTS

### V1 — MVP (This Milestone)

#### Backend (Python + FastAPI)
- [ ] Fetch opportunities from Polymarket Gamma API (`https://gamma-api.polymarket.com/events`)
- [ ] Filter by: categories (crypto, politics, sports, weather, entertainment), timeframe (daily/short-term/monthly/any), max entry price (≤$0.60 default)
- [ ] Paper broker: simulate trades, track portfolio (cash, positions, history)
- [ ] SQLite persistence for portfolio state, trade history, signal log
- [ ] Bot controls: start/stop bot, enable/disable fetching independently
- [ ] Background scheduler for polling loop (configurable interval, default 5 min)
- [ ] REST API endpoints for dashboard communication
- [ ] Debug/error logging system

#### Frontend (React + Tailwind)
- [ ] Portfolio summary: available cash, in positions, total value, realized P&L, withdrawn, win rate
- [ ] Bot controls: on/off toggle, fetching toggle, next fetch countdown
- [ ] Settings panel: categories multi-select, timeframe dropdown, default trade amount, max entry price
- [ ] Pending approval tab: opportunities with editable trade configuration (side, amount, preview)
- [ ] Open positions tab: current positions with Polymarket links
- [ ] Trade history tabs: All History, Winning Trades, Losing Trades
- [ ] Manage funds: deposit and withdraw paper money
- [ ] Debug logs panel: collapsible, shows INFO/WARN/ERROR with timestamps

#### Infrastructure
- [ ] Docker Compose setup (backend + frontend + volume for SQLite)
- [ ] Works on macOS (Docker Desktop or OrbStack)
- [ ] Environment configuration via .env file

### V2 — Future (Out of Scope for This Milestone)
- Real trade execution via Polymarket API
- Telegram/Slack notifications
- Advanced strategies (momentum, mean-reversion)
- Historical backtesting
- Multi-user support

### Out of Scope (Explicit Exclusions)
- Real money trading — paper only for safety
- Mobile app — web dashboard sufficient
- Complex AI/ML strategies — simple rule-based filtering only
- Authentication — single user, local deployment

---

## TECHNICAL CONSTRAINTS

### Stack (Locked)
- **Backend:** Python 3.11+, FastAPI, SQLite, APScheduler
- **Frontend:** React 18, Tailwind CSS, Vite
- **Infrastructure:** Docker, Docker Compose
- **No external databases** — SQLite file mounted as volume

### Environment
- Claude Code sandbox (Linux container)
- macOS host for deployment
- Network access to Polymarket API required

### API Reference
```
Polymarket Gamma API:
GET https://gamma-api.polymarket.com/events?closed=false&active=true&limit=10&order=volume&ascending=false

Response: Array of events with:
- title: string (question)
- volume: number
- endDate: string (ISO)
- markets: Array with outcomePrices as JSON string '["0.65", "0.35"]'
- slug: string (for URL construction)
```

---

## ARCHITECTURE

```
polymarket-paper-trader/
├── docker-compose.yml
├── .env.example
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── app/
│   │   ├── main.py              # FastAPI app + CORS
│   │   ├── config.py            # Settings from env
│   │   ├── database.py          # SQLite setup
│   │   ├── models.py            # Pydantic models
│   │   ├── schemas.py           # DB schemas
│   │   ├── routers/
│   │   │   ├── portfolio.py     # GET/POST portfolio, funds
│   │   │   ├── opportunities.py # GET opportunities, POST approve/reject
│   │   │   ├── positions.py     # GET positions
│   │   │   ├── history.py       # GET trade history
│   │   │   ├── bot.py           # GET/POST bot status, settings
│   │   │   └── logs.py          # GET debug logs
│   │   ├── services/
│   │   │   ├── polymarket.py    # API client
│   │   │   ├── strategy.py      # Filtering logic
│   │   │   ├── paper_broker.py  # Trade simulation
│   │   │   └── scheduler.py     # Background jobs
│   │   └── utils/
│   │       └── logger.py        # Structured logging
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   ├── vite.config.js
│   ├── tailwind.config.js
│   ├── index.html
│   └── src/
│       ├── main.jsx
│       ├── App.jsx
│       ├── api.js               # Backend API client
│       ├── components/
│       │   ├── Header.jsx
│       │   ├── PortfolioCards.jsx
│       │   ├── BotSettings.jsx
│       │   ├── OpportunityCard.jsx
│       │   ├── PositionsTable.jsx
│       │   ├── HistoryTable.jsx
│       │   ├── DebugLogs.jsx
│       │   └── ManageFunds.jsx
│       └── hooks/
│           └── usePolling.js    # Auto-refresh hook
└── data/
    └── .gitkeep                 # SQLite volume mount point
```

---

## KEY DESIGN DECISIONS

### Opportunity Card (Editable Trade)
Each pending opportunity allows user to:
1. Toggle between YES/NO side (shows both prices)
2. Adjust trade amount with quick buttons ($5, $10, $25, Max) or custom input
3. See live preview: shares @ price = cost, potential payout, potential profit
4. "Compare both sides" expandable section
5. Reset to bot's recommendation
6. Approve or Skip

### Bot Controls (Two Independent Toggles)
1. **Bot Running** — Master switch. When off, everything stops.
2. **Fetching Enabled** — When off, bot continues monitoring positions but doesn't fetch new opportunities.

This allows pausing opportunity intake while letting existing positions resolve.

### Paper Broker Logic
- Starting balance: $100 (configurable)
- Trade execution: `shares = floor(amount / price)`, `cost = shares * price`
- Position tracking: entry price, current price (updated on fetch), unrealized P&L
- Resolution: When market resolves (check via API), move to history with WIN/LOSS

### Exit Strategy
Hold until market resolves. Polymarket binary markets pay $1 for correct outcome, $0 for incorrect.

---

## API ENDPOINTS

### Portfolio
- `GET /api/portfolio` — Returns cash, positions value, total value, realized P&L, withdrawn, win rate
- `POST /api/portfolio/deposit` — Add funds `{amount: number}`
- `POST /api/portfolio/withdraw` — Remove funds `{amount: number}`

### Opportunities
- `GET /api/opportunities` — Returns filtered opportunities based on current settings
- `POST /api/opportunities/{id}/approve` — Execute paper trade `{side: "YES"|"NO", amount: number}`
- `POST /api/opportunities/{id}/reject` — Skip opportunity

### Positions
- `GET /api/positions` — Returns open positions with current prices

### History
- `GET /api/history` — Returns all resolved trades
- `GET /api/history?result=WIN` — Filter to wins only
- `GET /api/history?result=LOSS` — Filter to losses only

### Bot
- `GET /api/bot/status` — Returns `{running: bool, fetching: bool, nextFetch: datetime, settings: {...}}`
- `POST /api/bot/start` — Start bot
- `POST /api/bot/stop` — Stop bot
- `POST /api/bot/fetch/enable` — Enable fetching
- `POST /api/bot/fetch/disable` — Disable fetching
- `PUT /api/bot/settings` — Update settings `{categories: [], timeframe: string, tradeAmount: number, maxPrice: number, interval: number}`

### Logs
- `GET /api/logs` — Returns recent log entries `[{timestamp, level, message}]`

---

## DASHBOARD REFERENCE IMPLEMENTATION

The React dashboard should match this design (see artifact for interactive version):

**Header Row:**
- Title: "Polymarket Paper Trader" with "Educational / Research Mode" subtitle
- Bot Running toggle (green when on, red when off)
- Fetching toggle (blue when enabled, gray when paused)
- Next fetch countdown (--:-- when paused)
- Settings gear icon

**Portfolio Cards (6 cards):**
1. Available Cash (green)
2. In Positions (blue)
3. Total Value (white)
4. Realized P&L (green/red based on value)
5. Withdrawn (yellow)
6. Win Rate (purple) with W/L count
7. Manage Funds (Add/Withdraw buttons)

**Bot Status Bar:**
Shows active filters: categories pills, timeframe, max price, "Fetching paused" indicator if applicable

**Tabs:**
1. Pending Approval (count)
2. Open Positions (count)
3. All History (count)
4. Winning Trades (count, green)
5. Losing Trades (count, red)

**Opportunity Card Structure:**
- Category pill + "View on Polymarket ↗" link
- Question text
- End date + Volume
- Side selector: YES @ $X.XX / NO @ $X.XX (selected has colored ring)
- Amount input with quick buttons
- Preview: "Buy N shares @ $X.XX = $X.XX" + payout + profit
- Compare both sides (expandable)
- Reset / Skip / Approve buttons

**Debug Logs Panel:**
- Collapsible at bottom
- Shows error count badge if errors exist
- Timestamp + Level + Message per line
- Color coded: ERROR (red), WARN (yellow), INFO (gray)

---

## EXECUTION PHASES (GSD Roadmap)

### Phase 1: Project Foundation
- Initialize project structure
- Docker Compose setup
- Backend skeleton (FastAPI + SQLite)
- Frontend skeleton (React + Vite + Tailwind)
- Verify containers build and communicate

### Phase 2: Data Layer
- SQLite schema (portfolio, positions, history, opportunities, logs, settings)
- Database initialization and migrations
- Pydantic models and schemas

### Phase 3: Polymarket Integration
- API client for Gamma API
- Fetch and parse events
- Strategy filtering (categories, timeframe, price)
- Error handling and logging

### Phase 4: Paper Broker
- Portfolio management (cash, deposit, withdraw)
- Trade execution simulation
- Position tracking
- Market resolution detection
- Trade history recording

### Phase 5: Bot Engine
- APScheduler background jobs
- Bot state management (running, fetching)
- Opportunity queue management
- Settings persistence

### Phase 6: API Endpoints
- All REST endpoints per specification
- Request/response validation
- Error responses

### Phase 7: Dashboard — Layout
- Component structure
- Header with controls
- Portfolio cards
- Tab navigation

### Phase 8: Dashboard — Opportunities
- Opportunity card with full edit mode
- Side selection
- Amount configuration
- Preview calculations
- Approve/reject actions

### Phase 9: Dashboard — Positions & History
- Positions table with Polymarket links
- History table with result badges
- Filtered views (wins/losses)

### Phase 10: Dashboard — Polish
- Bot settings panel
- Debug logs panel
- Manage funds modal
- Auto-refresh polling
- Loading states

### Phase 11: Integration Testing
- End-to-end flow testing
- API → Frontend integration
- Bot cycle verification

### Phase 12: Documentation & Cleanup
- README with setup instructions
- Environment variable documentation
- Code cleanup and comments

---

## GSD WORKFLOW

For each phase, follow this cycle:

```
/gsd:discuss-phase N    # Capture your implementation preferences
/gsd:plan-phase N       # Research + create atomic task plans
/gsd:execute-phase N    # Execute plans with fresh context per plan
/gsd:verify-work N      # Manual testing + fix any issues
```

Use `/gsd:progress` to check status at any time.

---

## SUCCESS CRITERIA

The project is complete when:

1. **Functional:**
   - [ ] Bot fetches opportunities from Polymarket API
   - [ ] User can approve/reject opportunities with custom side and amount
   - [ ] Paper trades execute correctly (shares, cost, P&L calculations)
   - [ ] Positions update and resolve when markets end
   - [ ] Portfolio tracks all metrics accurately
   - [ ] Bot can be started/stopped, fetching can be paused independently

2. **Technical:**
   - [ ] `docker-compose up` starts entire stack
   - [ ] Frontend communicates with backend via REST API
   - [ ] State persists across container restarts (SQLite volume)
   - [ ] No errors in debug logs during normal operation

3. **UX:**
   - [ ] Dashboard matches design specification
   - [ ] Opportunity editing is intuitive (side, amount, preview)
   - [ ] All Polymarket links work correctly
   - [ ] Debug logs help diagnose issues

---

## COMMANDS TO START

```bash
# Initialize GSD
npx get-shit-done-cc --local

# Start project
/gsd:new-project

# When asked, reference this specification document

# Begin execution
/gsd:discuss-phase 1
/gsd:plan-phase 1
/gsd:execute-phase 1
/gsd:verify-work 1

# Continue through all phases...
```

---

## NOTES FOR CLAUDE CODE

1. **Use subagents liberally.** Each phase's plans should execute in fresh context windows to avoid quality degradation.

2. **Atomic commits per task.** Format: `{type}({phase}-{plan}): {description}`

3. **Test as you go.** Each phase should be independently verifiable before moving on.

4. **Handle API failures gracefully.** Polymarket API may rate limit or be temporarily unavailable.

5. **Keep the user informed.** Use the debug logs system to surface what the bot is doing.

6. **This is educational software.** Prioritize clarity and learning over optimization.

---

*Built with GSD Protocol v2.6 — "The complexity is in the system, not in your workflow."*
