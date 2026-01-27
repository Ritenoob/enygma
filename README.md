# enygma## `README.md` (Ritenoob/enygma)

````md
# ENYGMA (Miniature Enigma) — KuCoin Futures Trading System (Live + Manual + Auto)

ENYGMA is a **KuCoin USDT-M perpetuals** trading system built to reproduce the **Miniature Enigma v5.2.6** scoring + gate logic and execute with strict risk controls.

This repo is designed to be driven by **Claude Code / GitHub Copilot Agents / Codespaces / VPS** and must treat the included specification files as **canonical truth**.

---

## Canonical Truth Sources (Do Not Re-interpret)

These files define the system and override any chat text:

- `docs/MINIATURE_ENIGMA_TECHNICAL_SPECIFICATION.md` (v5.2.6, dated 2026-01-27)
- `docs/STRATEGY_CONFIG.json` (exact scoring ranges, indicator params, gates, risk math)
- `docs/SIGNAL_LOGIC.md` (entry checklist + signal definitions)
- `docs/WINNING_CONFIG_REFERENCE.md`
- `docs/COMPREHENSIVE_TEST_PLAN.md`
- `docs/MINIATURE_ENIGMA_AGI_AUDIT_REPORT.md`
- `docs/ACTION_CHECKLIST.md`
- `docs/CHANGELOG_v5.2.md`

**Rule:** if anything conflicts, **latest timestamp + spec files win**.

---

## What This Repo Must Support

### Trading Modes
- **SANDBOX / PAPER (default)**
- **LIVE trading via KuCoin API** (explicit confirmation required)
- **Manual Entry** (user must approve each trade)
- **Auto Trading** (fully autonomous) with notifications enabled

### Markets
- **Futures perpetuals only** for execution truth and risk math.
- Symbol universe must be configurable (per your Mirko universe list).

### Execution & Risk (Hard Requirements)
- ROI-based SL/TP **leverage-aware** (inverse leverage scaling) per `STRATEGY_CONFIG.json`.
- Early break-even + profit lock + staircase trailing stop; **never untrail**.
- No time-based exits; exits occur only via SL/TP/stop moves and scale/lock logic.
- **reduceOnly** on all futures exits.
- Cancel/replace stop updates must be coordinated and backed by retry queue logic.

### Data Realism
- Backtests must include fees + slippage (fill model configurable).
- DOM/microstructure logic is **LIVE forward-shadow only** (no “backtest optimized DOM”).

---

## Entry Signal Requirements (ENYGMA Gates)

Canonical gates are defined in `docs/MINIATURE_ENIGMA_TECHNICAL_SPECIFICATION.md` and `docs/STRATEGY_CONFIG.json`:
- Threshold gate: `|score| >= minScore` (config default: 75)
- Dead zone gate: `|score| >= 20`
- Confluence: `minCount >= 4` and `minPercentage >= 50%`
- Confidence gate: config default `>= 0.85`
- Trend alignment gate: must align with EMA trend (enabled)
- Drawdown gate: max drawdown `<= 3%`
- Volatility gate: threshold boost in high volatility

**User override requirement (must be implemented as a configurable stricter mode):**
- “No entry between 20–80” and “require threshold cross into actionable zone”
- “Require min 4 signals” (already canonical)
- “Require confidence 90%” (stricter than canonical 0.85)

Implementation must provide a feature flag / config switch, e.g.:
- `STRICT_ENTRY_MIN_SCORE=80` (default ON)
- `STRICT_MIN_CONFIDENCE=0.90` (default ON)
while keeping canonical values available for compatibility.

---

## Indicator Constraint (User Override)
- **Remove RSI alone**: standalone RSI must not trigger entries by itself.
- If RSI remains implemented for compatibility, it must be **disabled by default** and/or only used through **StochRSI** or explicitly gated combinations.

AO (**Awesome Oscillator**) is required and must be present in scoring when enabled by timeframe config.

---

## Repository Expectations

This repo must include:
- Live trading engine (KuCoin REST + WS)
- Strategy/scoring engine reproducing `STRATEGY_CONFIG.json`
- Risk engine reproducing `riskManagement` formulas exactly
- Notifications module (console + optional webhook/telegram/email)
- Dashboard + metrics + health
- Tests (unit + integration), CI, lint/type/security scanning
- Codespaces + VPS deployment guidance

---

## Quick Start (Developer)

1) Create venv + install:
```bash
python3.11 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
````

2. Configure env:

```bash
cp .env.example .env
# Fill in KuCoin credentials (NO withdrawal perms).
```

3. Run:

```bash
./start.sh
```

---

## Safety

* No withdrawals.
* Secrets only via environment variables.
* Redact secrets in logs.
* Live mode requires explicit interactive confirmation.

---

## License / Disclaimer

For engineering and research use. Trading is risky; you are responsible for outcomes.

````

---

## `MASTER_PROMPT.md` (Claude Code / Copilot Agents / GitHub Agents / Codespaces / VPS)

```md
# ENYGMA MASTER PROMPT (Canonical-Truth Build)

## ROLE
You are a senior crypto systems engineer. You must generate and/or modify a repository to implement the ENYGMA / Miniature Enigma v5.2.6 trading system.

## ABSOLUTE TRUTH (Read First, Then Obey)
Before writing any code, you MUST read and treat these as canonical:
- docs/MINIATURE_ENIGMA_TECHNICAL_SPECIFICATION.md (v5.2.6, 2026-01-27)
- docs/STRATEGY_CONFIG.json
- docs/SIGNAL_LOGIC.md
- docs/WINNING_CONFIG_REFERENCE.md
- docs/COMPREHENSIVE_TEST_PLAN.md
- docs/MINIATURE_ENIGMA_AGI_AUDIT_REPORT.md
- docs/ACTION_CHECKLIST.md
- docs/CHANGELOG_v5.2.md

Conflict rule:
1) Latest-dated spec/config wins.
2) If user overrides are explicitly stated below, implement them as STRICT MODE flags (default ON) without breaking compatibility.

## USER OVERRIDES (Must Be Implemented)
- Remove RSI-alone: standalone RSI signals MUST NOT be an entry trigger by itself.
  - RSI may exist for compatibility but must be disabled by default OR only used through StochRSI.
- AO must be present (Awesome Oscillator).
- Strict entry rules (default ON):
  - No entry when |score| is between 20 and 80.
  - Require threshold-cross event into actionable zone:
    - LONG: prevScore < +80 and score >= +80
    - SHORT: prevScore > -80 and score <= -80
  - Require confluence >= 4 indicators.
  - Require confidence >= 0.90.
- These strict rules must be configurable so canonical (75 / 0.85) can still be used if required for backward compatibility.

## SCOPE (What To Build)
Build a LIVE-capable KuCoin futures trading bot with:
- KuCoin REST + authenticated WebSocket (token refresh, reconnect, resubscribe)
- Futures perpetual execution (primary)
- Manual entry mode (user approval) + auto-trading mode
- Notifications enabled (trade/risk/health)
- Risk management exactly as in STRATEGY_CONFIG.json (ROI + leverage-aware, early BE, staircase trailing, reduceOnly exits)
- No time-based forced exits
- Retry queue for stop update cancel/replace
- Observability: JSON logs (redaction), /metrics, /health/live, /health/ready, dashboard

## REQUIRED REPO STRUCTURE (Minimum)
enygma/
├── docs/ (copy canonical files here)
├── src/
│   ├── config/
│   ├── exchange/          # KuCoin REST + WS clients
│   ├── data/              # OHLCV + TF aggregation
│   ├── indicators/        # per STRATEGY_CONFIG.json
│   ├── signal/            # scoring + breakdown + entry gates
│   ├── risk/              # SL/TP/BE/trailing/limits
│   ├── orders/            # idempotent clientOid, reduceOnly, stop mgmt
│   ├── accounting/        # fees/funding/PnL, reconciliation drift checks
│   ├── monitoring/        # metrics/logging/notifications
│   └── util/              # retries, decimal math, schema validation
├── futures_bot.py
├── gui.py                 # FastAPI app: /dashboard /metrics /health/*
├── start.sh               # POSIX: load .env, show SANDBOX vs LIVE, confirm LIVE, logs/, run/ PIDs
├── tests/
│   ├── unit/
│   ├── integration/
│   └── chaos/
├── deployment/
│   ├── Dockerfile
│   ├── docker-compose.yml
│   ├── devcontainer/      # Codespaces
│   └── systemd/           # VPS
├── .env.example
├── requirements.txt (pinned)
├── pyproject.toml (ruff/black/mypy/bandit)
└── .github/workflows/ci.yml

## NON-NEGOTIABLE IMPLEMENTATION RULES
- No placeholders. No TODOs. No pseudocode.
- Deterministic financial math (Decimal).
- Never untrail stops.
- All exits reduceOnly.
- Enforce drawdown gate (<= 3% from peak per config).
- Always include fees (taker/taker baseline) in break-even and PnL.

## TEST REQUIREMENTS
- Unit tests must validate:
  - indicator math + signals (esp. Williams %R, StochRSI, Stochastic, AO, EMA trend)
  - scoring clamping and classification
  - entry gates (threshold/deadzone/confluence/confidence/trend/drawdown/volatility)
  - strict-mode overrides (80/0.90/cross event)
  - ROI-based SL/TP/BE/trailing formulas
- Integration tests (opt-in) must hit KuCoin sandbox and validate:
  - signed requests
  - WS auth + reconnect
  - order idempotency with clientOid
  - reduceOnly closes
- Target >= 80% coverage and pass CI.

## SECURITY
- Never enable withdrawals.
- Secrets only from environment variables.
- Redact secrets in logs.
- Include Bandit + Safety + mypy in CI.

## OUTPUT MODE (For Codegen Agents)
If you are generating the repo from scratch, output:
1) MANIFEST (all files/dirs)
2) FILES (every file full content)
3) VERIFY (fresh venv setup/run/test commands)
End with: <<REPO_COMPLETE>>

---

# Copilot / GitHub Agents Task Prompts

## Agent Task 1 — Canonical Ingestion
“Read all docs/ canonical files listed above. Produce a machine-checkable ‘truth map’ JSON: indicator params, weights, gate thresholds, risk formulas, invariants.”

## Agent Task 2 — Signal Engine
“Implement src/signal scoring exactly per STRATEGY_CONFIG.json. Include full breakdown objects and strict-mode gates (80/0.90/cross event). Disable RSI-alone by default.”

## Agent Task 3 — Risk + Order Management
“Implement ROI-based SL/TP/BE/staircase trailing and coordinated cancel/replace stop updates with retry queue. Ensure reduceOnly on exits.”

## Agent Task 4 — KuCoin Exchange Layer
“Implement REST signing + WS auth token refresh + reconnect/resubscribe. Implement idempotent clientOid and 429 backoff with jitter.”

## Agent Task 5 — GUI + Metrics + Notifications
“FastAPI GUI at /dashboard, metrics at /metrics, health endpoints, notifications for trades/risk/halts. Redacted JSON logs.”

---

# Codespaces Prompt
“Create .devcontainer configuration for Python 3.11, install pinned deps, forward port 8000, and provide a VS Code task to run ./start.sh.”

# VPS Prompt
“Provide systemd units for futures_bot.py and gui.py, secure env file permissions, logrotate, healthcheck script, and deployment instructions for Ubuntu 24.04.”
````

Prompt A: Instruction/System Prompt
You are a senior crypto-systems engineer delivering a complete, production-grade trading bot
repository. Your target is KuCoin, supporting USDT-M perpetuals, spot, and margin (cross or isolated),
defaulting to sandbox mode.
Key constraints:
• Runtime: Python 3.11 on Ubuntu (virtualenv); project should be ready for future containerization via
Docker.
• Launch: Provide a POSIX-compatible start.sh that loads the .env file, checks for API keys,
shows the run mode (SANDBOX vs LIVE), requires confirmation if LIVE, then launches the bot
( python futures_bot.py ) and a GUI ( uvicorn gui:app ) in the background. It should log
output to timestamped files in logs/ and record process PIDs in run/ .
• Strategy: Implement a mean-reversion strategy targeting ETHUSDTM on 15m timeframe by default.
Design a config-driven strategy loader that allows overriding the symbol and timeframe (to plug in
future strategies easily).
• Execution: Use KuCoin’s native REST v2 API for order execution and authenticated WebSocket for
real-time data (with HMAC SHA256 signing, token refresh, and automatic reconnect + resubscribe).
Support limit, market, and stop orders (stop-limit and stop-market), using idempotent clientOid
for all orders. Implement slippage protection and reduce-only flags for closing positions. Include
robust handling of rate limits (exponential backoff on HTTP 429 responses). The position mode
(hedge vs one-way) must be selectable via configuration.
• Control Modes: Support both fully autonomous trading and a manual-confirmation mode where the
user must approve each trade signal (toggle via config).
• Observability: Expose Prometheus metrics at /metrics and health endpoints at /health/live
and /health/ready . Provide a simple FastAPI-based GUI at /dashboard to display status (e.g.
recent logs, current positions). Use structured JSON logging with sensitive data redacted.
• Risk Controls: Enforce a max position size limit, per-trade stop-loss and take-profit thresholds, a
daily loss limit, and a max drawdown cap. Implement a position exposure breaker that halts trading
if net exposure exceeds a threshold, and a global kill switch to stop all trading immediately if
needed.
• Accounting: Track fees (taker/maker) and funding payments, and maintain real-time PnL.
Continuously reconcile the bot’s internal balance with the on-exchange account snapshot to detect
any drift, and auto-pause trading on any mismatch.
• Testing: Include comprehensive unit tests (e.g. for math calculations, signal generation) and
integration tests against the KuCoin sandbox. Provide scaffolding for replay or chaos testing. Aim for
high code coverage (≥80%) and ensure the test suite and linters pass in CI.
• Security: Do not hardcode secrets or enable withdrawals. Load all credentials from environment
variables only. Code must pass static analysis (type checks with mypy, security linting with Bandit,
dependency audit with Safety) and use pinned dependency versions.
• CI/CD: Include a GitHub Actions workflow for automated linting, typing, testing, coverage, and
security scanning.
1• Deliverables: The repository should include a full file tree with all essential components: a src/
directory (for modules like config, exchange client, strategy, risk management, order handling, PnL/
accounting, monitoring, etc.), a tests/ directory, a deployment/ directory (Dockerfile, docker-
compose, scripts), a docs/ directory for any documentation, plus configuration files like
.env.example , requirements.txt , pyproject.toml , and a detailed README.md .
Follow this output format strictly:
• MANIFEST: Begin with a bullet list of all files and directories to be created.
• FILES: For each file, provide a code block in the format:
```
# file: <path>
```<language>
<file content>
```
```
(Include every file’s full content with no omissions or placeholders.)
• VERIFY: After listing all files, provide a shell command sequence (in a code block) to set up and run
the project in a fresh virtual environment (e.g. installing requirements, running tests, starting the
bot).
• End the output with the marker <<REPO_COMPLETE>> (to signal that all parts of the repository
have been output).
• If the content is too large for one response, split it into multiple parts. Use the markers <<PART 1
of N>> , <<PART 2 of N>> , etc., at the end of each part (except the final one, which should end
with <<REPO_COMPLETE>> ), and continue seamlessly.
Important: No placeholders or “TODO” sections are allowed – the code must be fully implemented and
production-ready. Do not use ellipses or omit any details. All features described above must be properly
integrated and functional in the provided code. Ensure the repository is consistent, idiomatic, and runnable
as-is.
Prompt B: Code-Generation/User Prompt
You are to generate a complete, modular source code repository for a KuCoin cryptocurrency trading
system. The system must meet the following requirements:
• Exchange Support: Handle trading on KuCoin USDT-M Perpetual futures (primary focus), as well as
standard Spot and Margin markets (both cross & isolated margin).
• API Integration: Use KuCoin’s REST API v2 for order execution and the authenticated WebSocket
feed for market data streaming. Implement proper HMAC SHA256 signing for authentication, handle
token refresh, and ensure the WebSocket client automatically reconnects and resubscribes to feeds
on connection drops.
• Position Modes: Allow a runtime configuration toggle between Hedge Mode and One-Way Mode
for futures positions, to support both hedged and traditional position management.
• Strategy Module: Follow a config-driven strategy architecture. Provide a baseline mean-reversion
strategy (using Bollinger Bands + RSI or similar) targeting ETHUSDTM on 15m timeframe by default.
2The framework should allow plugging in different strategies, with configuration options to change
the trading symbol and timeframe.
• Start Script: Include a POSIX start.sh script to bootstrap the application. This script should load
environment variables from .env , verify that API keys are present, display the run mode (sandbox
vs live trading) and require explicit confirmation if in live mode. Upon confirmation, it should launch
the trading bot ( python futures_bot.py ) and a web GUI ( uvicorn gui:app ) in the
background. All output must be logged to files in a logs/ directory (with filenames including
timestamps), and process IDs should be recorded in a run/ directory for easy stop/cleanup.
• Trade Execution: Support placement and management of limit, market, stop-limit, and stop-
market orders on KuCoin. Use a unique clientOid for each order to ensure idempotency
(prevent duplicate orders on retries). Implement slippage safeguards: e.g., if price moves
unfavorably beyond a threshold during order execution, cancel or adjust the order. Handle order
cancellations and reduce-only flags properly for closing positions without over-shooting.
Incorporate robust retry logic with exponential backoff when API rate limits (HTTP 429) are
encountered.
• Control Mode: The bot should run in a fully automated mode by default, but also support a manual
confirmation mode where trade signals are presented for user approval before execution
(configurable toggle).
• Risk Management: Enforce comprehensive risk controls. Limit the maximum position size (e.g., as a
percentage of account equity). Implement per-trade stop-loss and take-profit thresholds, as well as
a daily cumulative loss limit. Monitor account drawdown and halt trading if a maximum drawdown
percentage is exceeded (acting as a “circuit breaker”). Include a global kill switch that can
immediately flatten all positions and stop the bot. Also check for excessive leverage or exposure and
prevent opening new trades if certain risk thresholds are breached.
• Reconciliation & PnL: Continuously track profits and losses, including trading fees and funding
payments for perpetual swaps. After each trade (or on a fixed interval), reconcile the bot’s internal
account balance and positions with KuCoin’s reported account status to detect any discrepancies. If
any mismatch or account drift is detected, the bot should alert and auto-pause trading to prevent
potential errors. Ensure the bot correctly calculates unrealized and realized PnL, and incorporates
funding rate debits/credits into PnL for futures.
• Monitoring & UI: Implement a lightweight web dashboard (FastAPI-based, served by Uvicorn)
accessible at /dashboard . This should display current bot status, recent trades or log entries, and
basic performance metrics. Provide Prometheus-compatible metrics at an endpoint (e.g. /
metrics ) for cluster monitoring, as well as health check endpoints ( /health or /health/live
and /health/ready ). Logging should be in structured JSON format with sensitive information (API
keys, etc.) redacted.
• Testing: Provide a comprehensive test suite. Include unit tests for strategy logic, indicator
calculations, order sizing, and risk checks. Add integration tests that can run against the KuCoin
sandbox environment to simulate real API interactions. Include scaffolding for end-to-end or chaos
testing (for example, the ability to replay historical data or simulate network failures). Aim for at least
80% code coverage. All tests and linters must pass in Continuous Integration.
• Security Best Practices: No secret keys or passwords should be hardcoded; use environment
variables ( .env file) for all credentials and sensitive config. Limit API key permissions to only what
is necessary (trading and balance reads, no withdrawal access). The code should pass static analysis
and security audits (e.g., type checking, linting with Bandit for security issues, etc.), and all
dependencies should be pinned to exact versions to avoid supply-chain issues.
• Project Structure: Organize the project into a professional structure:
3• A src/ directory containing the core modules: e.g. config (configuration and settings),
exchange (KuCoin API client and WebSocket handling), strategy (strategy interface and
implementations), risk (risk management logic), orders (order management and tracking),
accounting or pnl (PnL and reconciliation logic), monitoring (logging, metrics, alerts),
utils (utility functions), etc.
• A deployment/ directory for deployment-related files: include a Dockerfile, a docker-
compose.yml , the start.sh script, and any helper scripts like healthcheck.sh .
• A tests/ directory with subfolders for unit tests and integration tests (and sample configurations
or data for testing).
• Additional top-level files: a sample environment config .env.example , a requirements.txt (or
poetry pyproject.toml ) specifying all dependencies, a README.md with usage and
development instructions, and a docs/ folder for any further documentation or design notes.
• Deliverables: Provide the full repository content in a file-by-file format. Each file should be
preceded by a markdown line indicating its path (for example, “ # file: src/config/
settings.py ”) and then the file content in a fenced code block. Do not omit any files or sections of
code — include everything necessary to build, test, and run the project. There should be no
placeholders or incomplete sections; all features must be implemented as described. Ensure that the
final output is a runnable codebase that a senior engineer could directly put into production.
Deliver: Present the repository as described above, with every file’s content shown. Use multiple answer
messages if needed to accommodate all files (mark continuations with <<PART 1 of N>> ,
<<PART 2 of N>> , etc., and conclude the final part with <<REPO_COMPLETE>> ). The result should be a
ready-to-run code repository fulfilling all the above requirements, with no additional explanation or
commentary outside the scope of the repository content.
4## Repository Expectations

This repo must include:
- Live trading engine (KuCoin REST + WS)
- Strategy/scoring engine reproducing `STRATEGY_CONFIG.json`
- Risk engine reproducing `riskManagement` formulas exactly
- Notifications module (console + optional webhook/telegram/email)
- Dashboard + metrics + health
- Tests (unit + integration), CI, lint/type/security scanning
- Codespaces + VPS deployment guidance
AO (**Awesome Oscillator**) is required and must be present in scoring when enabled by timeframe config.

What I will do next (without asking further questions)

Ingest and lock truth sources
I will treat the following as binding and non-negotiable:

SIGNAL_LOGIC.md

MINIATURE_ENIGMA_TECHNICAL_SPECIFICATION.md

WINNING_CONFIG_REFERENCE.md

COMPREHENSIVE_TEST_PLAN.md

STRATEGY_CONFIG.json

MINIATURE_ENIGMA_AGI_AUDIT_REPORT.md

ACTION_CHECKLIST.md

CHANGELOG_v5.2.md

CLAUDE.md

Resolve conflicts automatically

If any prompt, README, or instruction contradicts those files, the files win.

Earlier assumptions (e.g., RSI-alone, BB+RSI-only mean reversion) are discarded if not present in the docs.

AO inclusion, dead-zone rules, score thresholds, confluence counts, and stop logic will follow the specs exactly.

Normalize outputs

README.md will explicitly state that docs/ are the source of truth

The MASTER prompt will instruct Claude/Copilot/Agents to read and obey those files first

No duplicated or reinterpreted logic outside the docs

Proceed with implementation alignment

Prompts → aligned to specs

Strategy → aligned to screenshots + docs

Risk → aligned to audit + test plan

No further conceptual redesign

What this means operationally

We are no longer “designing” the system

We are executing an already-defined system

AI agents are constrained to implementation only

Any future changes must go through the docs first

If you want the next concrete step executed immediately, say one of the following (or just say “go”):

“Regenerate README strictly from docs”

“Regenerate MASTER_PROMPT strictly from docs”

“Produce implementation prompts per module (exchange, signal, risk, UI)”

“Prepare GitHub commit plan + file mapping”

I’m locked to the project files as truth from here forward.1) Critical Corrections (must fix before live)
A. Your ROI risk math is inconsistent (material)

You label stops as ROI-based:

STOP_LOSS_ROI=6
TAKE_PROFIT_ROI=30


But your calculator treats 6% as notional loss. In perpetual futures, ROI% is typically on margin, not notional.

Correct isolated-margin math (ignoring fees/slippage for the moment):

Margin per trade = equity * position_size%

Worst-case loss at ROI stop ≈ margin * STOP_LOSS_ROI%

Example (your numbers):

Equity: $1,000

Position size: 3% → margin = $30

SL: 6% ROI → loss ≈ $30 * 0.06 = $1.80 (not $21.60)

Also, the approximate underlying price move that corresponds to an ROI stop is:

price_move% ≈ ROI% / leverage (linear contract approximation)

So at 12×, a 6% ROI stop corresponds to about 0.5% adverse price movement (plus fees/slippage).

Action: either:

keep ROI-based stops and fix the calculator + dashboards, or

rename and convert to price-based stops (e.g., STOP_LOSS_PCT_PRICE) to match your existing math.

B. Your “Cancel All Orders” emergency endpoint uses an abandoned V1

Your emergency script uses:

DELETE /api/v1/orders

That endpoint is explicitly listed as “Cancel All Orders - V1” and has very high weight (800) and is under Abandoned Endpoints.

Use the current endpoint:

DELETE /api/v3/orders (weight 10)
and separately:

DELETE /api/v1/stopOrders (Cancel All Stop orders, weight 15)

C. Shadow mode should validate requests without placing orders

Your current shadow mode returns early, so it does not validate signing, symbol constraints, order schema, etc.

KuCoin Futures provides a proper “order test” endpoint:

POST /api/v1/orders/test (weight 2)

Action: In shadow mode, call /api/v1/orders/test with the exact body you would send to /api/v1/orders.

2) API Signing & Connectivity: harden and standardize

Your signing logic is directionally correct, but you should centralize it and enforce canonical rules:

Prehash: {timestamp + method + endpoint + body}

Query string must be included in endpoint for GET/DELETE

Body must be exact JSON string (no extra spaces)
KuCoin’s rules are explicit.

Updated connectivity test (recommended)

Use the documented Futures endpoint:

GET https://api-futures.kucoin.com/api/v1/account-overview (weight 5)

Also include currency=USDT so the signature includes the query string deterministically.

3) Execution safety interlocks (non-negotiable for live funds)

Add two independent “go-live gates”:

BOT_MODE=live

LIVE_TRADING_ARMED=true (or a stronger acknowledgement string)

And add a file-based kill switch:

If /tmp/BOT_EMERGENCY_STOP exists → do not place orders, only manage/cancel exits.

This prevents accidental live trading from a mis-set .env, CI run, or developer shell.

4) Risk & order-management patches aligned with your “execution/risk truth”
A. Exits must be reduceOnly always

You already state this requirement; enforce it at the request-building layer. KuCoin Futures order payload supports reduceOnly.

B. Stop/TP should use exchange-native conditional orders where possible

KuCoin supports:

Take-profit/stop-loss order endpoint: POST /api/v1/st-orders (weight 2)
It supports both up/down triggers (two-sided bracket behavior) in one request: triggerStopUpPrice and triggerStopDownPrice and stopPriceType.

Also, KuCoin explicitly documents closeOrder behavior: when closeOrder=true, you can omit side/size/leverage and it closes the position.

Recommended structure per fill:

Place entry (/api/v1/orders)

Confirm fill (WS or poll)

Place TP/SL bracket (/api/v1/st-orders) as closeOrder-style exit logic (plus reduceOnly where applicable)

C. Cancel/replace stop update must be coordinated

To satisfy your “cancel/replace risk mitigated backed by retry queue” requirement:

Maintain a per-position “risk-order state machine”

Update stops via:

place new stop (or st-order)

confirm accepted

cancel old stop(s)
This avoids windows with no stop.

5) Monitoring & reconciliation improvements
A. Position truth source should be exchange, not internal state

Use:

GET /api/v1/positions to reconcile open positions (weight 2)

Your daily checklist should explicitly include:

Positions diff: exchange vs bot state

Open orders diff: exchange open orders list vs bot tracking

Stop orders list diff: stopOrders endpoint

B. Upgrade “health-monitor.sh”

Right now it assumes /performance always exists and returns JSON with .balance.

Add checks for:

last websocket heartbeat

exchange latency (timestamp skew)

last successful signed API call

order placement/cancel error rate

6) Concrete patch set (drop-in replacements)
Patch 1 — Shadow mode should hit KuCoin’s order-test endpoint

Use KuCoin Futures:

POST /api/v1/orders/test

Intent: validate signature + parameters without placing a trade.

Patch 2 — Replace emergency cancel endpoint

Replace V1 abandoned DELETE /api/v1/orders
with:

DELETE /api/v3/orders
and also:

DELETE /api/v1/stopOrders

Patch 3 — Add “close all positions” logic (real close, not just cancel)

To close positions you must submit reduceOnly market orders (or a closeOrder-style close via the TP/SL interface). Use positions list:

GET /api/v1/positions

Then, for each open position:

If LONG → send market SELL reduceOnly size = abs(qty)

If SHORT → send market BUY reduceOnly size = abs(qty)

Order placement is via:

POST /api/v1/orders

7) Plan edits you should apply immediately (summary)

Fix ROI risk calculator (or rename variables) to eliminate notional-vs-margin confusion.

Shadow mode → /api/v1/orders/test instead of “return early.”

Emergency cancel → /api/v3/orders + /api/v1/stopOrders.

Add LIVE arming gate + file kill switch.

Reconcile positions with GET /api/v1/positions as the truth source.

Prefer exchange-native TP/SL via POST /api/v1/st-orders and coordinate stop updates.

Centralize signing exactly as KuCoin specifies.Perfect. I’ll create a complete architecture and template for an enterprise-grade AGI-controlled crypto perpetuals trading system. It will cover:

* AGI agent orchestration (Claude + sub-agents)
* Signal generation (multi-source, multi-timeframe)
* Execution via manual/auto trader with exchange integrations
* Hooks + Claude code for agent control
* Machine learning + backtesting engine
* Paper/live trade with configurable SL/TP logic
* Modular, production-grade repo structure in stable languages (Python, Rust/Go if needed)

I’ll return a fully structured repo template with file layout, key code templates, and component descriptions.
Feel free to continue refining details while I work on this.
Here's your comprehensive report on automated trading bots, including architecture, code build, licensing, and regulatory/security requirements: {{file:file-7MYiFEdrFGhCwmu4iNV9xz}}.

This report covers ten real examples of algorithmic trading bots, summarises their technical foundations and regulatory/licensing contexts, and highlights key industry and cybersecurity obligations. Let me know if you need an analysis of a specific bot or help setting up a secure trading environment.
