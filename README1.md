Keep signal-engine in Python (FastAPI worker)

UI + desktop still TS

Downside: sharing types/config validation is harder (but doable via JSON Schema)

4) The signal-engine contract (the “truth source”)
Inputs (normalized)

Your engine should accept only normalized structures, never raw exchange payloads.

Minimum engine inputs per symbol/timeframe

OHLCV candles (with timestamps)

mark / index / last price (optional per frame, but recommended)

orderbook snapshot (optional)

funding rate + next funding time (optional)

open interest (optional)

buy/sell ratio proxy (if you have a reliable definition)

Output (strict)

signal: LONG | SHORT | FLAT

confidence: 0–100

score_breakdown: per-feature contributions

risk: suggested SL/TP zones and position sizing hints (optional, but useful)

explain: human-readable rationale for UI

fingerprint: deterministic hash (symbol+tf+configVersion+timestamp) to dedupe

5) Strategy configuration system (deployable + editable in-app)
Requirements

Human-editable JSON/YAML

Validated with JSON Schema

Versioned (configVersion)

Supports per-symbol overrides & timeframe overrides

Supports feature toggles and weights

Config layout (conceptual)
{
  "meta": { "name": "KuCoin Perp Scalp v1", "version": "1.0.0" },
  "universe": {
    "symbols": ["BTCUSDTM", "ETHUSDTM"],
    "timeframes": ["5m", "15m", "1h"]
  },
  "features": {
    "emaDeviation": { "enabled": true, "period": 30, "weight": 1.2 },
    "markIndexBasis": { "enabled": true, "weight": 1.0 },
    "fundingBias": { "enabled": true, "weight": 0.6 },
    "orderbookImbalance": { "enabled": false, "weight": 0.4 }
  },
  "decision": {
    "longThreshold": 2.2,
    "shortThreshold": -2.2,
    "cooldownSeconds": 300,
    "confirmations": { "minTimeframesAligned": 2 }
  },
  "risk": {
    "atrPeriod": 14,
    "slATR": 1.8,
    "tpATR": 2.4,
    "maxLeverage": 8
  }
}

6) Runtime orchestration (worker)
Scheduling model

Per timeframe cadence: 5m runs every 5m, 15m every 15m, etc.

A “symbol-timeframe job” produces one signal for the latest closed candle

Optional “intra-candle” mode for tick-driven signals (more complex; do later)

Worker responsibilities

Fetch/refresh market data (candles, mark/index, funding, OB snapshot)

Cache (Redis or in-memory) to reduce API calls

Call engine

Store results

Publish events

7) API endpoints (minimal but app-complete)

REST

GET /health

GET /configs / POST /configs (store + validate)

GET /signals/latest?symbol=...&tf=...

GET /signals/history?symbol=...&tf=...&from=...&to=...

GET /metrics/strategy?configVersion=...

Streaming

WS /stream/signals (or GET /stream/signals SSE)

UI subscribes to watchlist, receives live signal events

8) Storage (simple default)

Start with PostgreSQL (or SQLite for local desktop mode).

Tables:

configs (versioned JSON + schema version)

signals (timestamp, symbol, tf, signal, confidence, breakdown json, fingerprint)

runs (audit trail: duration, data freshness, errors)

market_snapshots (optional, for replay + debugging)

Desktop mode can run with:

embedded SQLite + local worker

OR connect to a remote API

9) Deployment modes (this is what you asked for: “replayable as an app too”)
Mode A — Cloud / server

api + worker on VPS/K8s

web hosted (Vercel or same box)

Any device can open UI

Mode B — Local “single box” app

Tauri desktop app launches:

local api + local worker (child processes)

UI is embedded webview

Uses SQLite for storage

Great for personal trading workstation

Mode C — Hybrid

Desktop app UI connects to cloud API

Worker runs server-side; desktop is just the console

10) The template you can copy/paste into your build plan
“Packages contract checklist”

packages/signal-engine must not import:

HTTP clients

DB clients

filesystem

It may import:

math utils

indicator implementations

schema-validated config types

“Adapters only”

packages/market-data is the only place that knows KuCoin’s payload shape

It outputs normalized:

candles

price frames

funding frames

orderbook frames

“Single source of symbols/timeframes”

Put symbols/timeframes/config in packages/config (and/or data/)

Worker, API, UI read the same config store

11) If you want the fastest “MVP app” path

Recommended stack:

Engine: TypeScript (pure library)

API: Fastify

Worker: Node + BullMQ (Redis) or simple cron scheduler first

UI: Next.js

Desktop: Tauri

This gets you a real installable app with a modern dashboard and no duplicated logic.


Below is a deployable Signal Generator architecture + repo template that works as:

a headless service (API + worker) for bots / automation

a full app (web UI, or desktop app via Tauri/Electron)

a reusable library (so the trading bot, screener, and UI all call the same signal logic)

This is the pattern that prevents “UI logic drift” and keeps one truth source for signals.

1) Target architecture (clean, deployable, app-ready)
Core principle

Signal logic is a pure, deterministic engine (no network calls, no DB, no UI). Everything else plugs into it.

Services

signal-engine (library)

Pure functions: indicators → features → scoring → signal decision

Deterministic, testable, backtestable

Used by: API service, worker, backtester, CLI, UI preview mode

market-data (adapter layer)

Normalizes exchange feeds (KuCoin futures, etc.)

Produces consistent candles, orderbook snapshots, funding, mark/index price

signal-api (app backend)

REST/WS API for UI + external consumers

Auth, rate limiting, caching

Reads signals from DB or publishes live signals via WebSocket/SSE

signal-worker (runtime)

Scheduler + orchestrator

Pulls market data, runs engine, writes signals, pushes notifications

ui-app

Dashboard: watchlists, signal stream, per-symbol drilldown, indicator overlays

Strategy configurator: YAML/JSON config editor, validation, versioning

“Replay mode”: load historical candles & run engine frame-by-frame

2) Dataflow

Live

Worker schedules symbols/timeframes → fetches normalized market data

Worker calls signal-engine.generateSignal()

Worker persists results (DB) + publishes event (Redis/NATS optional)

API serves latest signals + streams updates to UI

Replay / backtest

UI or CLI loads historical dataset


Evaluating AI Trading Bots With High Win Rates and Profit Factors (2026)
Overview of AI Trading Bots

Artificial‑intelligence (AI) trading bots are software programs that connect to brokerage or exchange APIs and execute trades based on pre‑defined rules or machine‑learning models. They attempt to remove human emotion and execute strategies 24/7, but they do not guarantee profits. Research from 2025 notes that the global AI trading market has grown quickly (estimated at $24.53 billion in 2025) as retail and institutional investors adopt automation. These bots may analyze price data, news sentiment and order‑book depth to generate signals. Performance is typically measured using win rate (percentage of profitable trades) and profit factor (ratio of total profits to total losses). In professional trading, profit factors above 2.0 are considered good and above 4.0 exceptional. However, reliability and profit claims vary widely; independent research urges caution, noting that AI systems inherit biases and may perform poorly when many agents interact, while articles from professional traders highlight that success may often be due to luck.

Evidence on Bots Claiming High Win Rates and Profit Factors
AI trading bot / platform	Highlights & evidence	Limitations / cautions
Forex Fury (MT4/MT5 expert advisor)	• The Forex Fury website states that the robot has a 93 % winning track record verified by Myfxbook accounts and is ranked highly by several review sites. The FAQ explains that the EA trades 5–10 times per day and aims to grow accounts 10–20 % per month. It offers low/medium/high‑risk settings, works with any MT4/MT5 broker and provides installation guides.	• Performance claims come from the vendor; there is little independent data. Users must configure risk parameters (lot size, maximum spread, number of trades) and monitor trades; the FAQ notes that adjusting max_spread and max_orders is often needed. Academic research warns that high win rates do not ensure profitability—small gains and occasional large losses can still lead to overall losses.
Trade Ideas / Holly AI (stocks)	• StockBrokers.com (Dec 2025) names Trade Ideas’ “Holly” AI and “Money Machine” as the best AI trading bot service for stocks. These tools provide daily AI‑generated signals, backtesting, risk levels and strategy automation. Subscriptions range from a free plan to premium plans (≈$127 month for real‑time data).	• The review emphasises that the platform is comprehensive but has a learning curve and does not directly execute trades on its own; users must link to a compatible broker. It also notes that AI trading tools should be treated as a co‑pilot, not a fiduciary. No explicit win‑rate or profit‑factor metrics are provided.
StockHero (stocks/crypto)	• Reviewed by StockBrokers.com as a great AI‑powered bot‑creation wizard: users can design bots or rent strategies via a marketplace, connecting to brokers such as TradeStation, Webull and Robinhood Crypto. Subscription plans range from $29.99/month (1 bot) to $99.99/month (50 bots).	• Performance depends on the chosen strategy; the review notes that marketplace bots lacked detailed historical performance data.
TrendSpider (stocks, forex, crypto)	• Offers AI‑driven scanners, charting, backtesting and strategy bots across equities, futures, forex and crypto. The StockBrokers.com review notes that enhanced plans allow up to 30 trading bots, and TrendSpider recognizes over 148 candlestick patterns.	• Provides tools for idea generation and automation but does not promise specific win rates; user must design and backtest their own strategies.
Tickeron AI Trading Agents (stocks/ETFs)	• Tickeron is a financial‑technology platform offering AI “agents” that trade specific baskets of stocks. In December 2025, the company reported that its top agent achieved a 102 % annualized return with a profit factor of 6.2 on a 15‑minute strategy, while another agent reached a 24 % annualized return with a profit factor of 8.9. In November 2025, Tickeron highlighted agents with profit factors as high as 65.0 (ARKQ agent), 27.8 (UWM) and 17.2 (SPY). An analysis by AgentiveAIQ notes that only 34 AI trading systems on Tickeron have audited records, but some reached profit factors over 4.4.	• These metrics come from Tickeron’s proprietary agents; independent verification is limited. Profit factors above 4 are rare; such results may be specific to certain market conditions and may not persist.
GPS Forex Robot (MT4)	• Reviewers (e.g., DragonExpertFX) describe GPS Forex Robot as a MT4‑based system that uses a “spread‑insensitive” approach and claims a 98 % win rate. A one‑time payment of $149 and low minimum deposit make it accessible.	• The claimed 98 % win rate lacks independent verification; high win rates can hide rare but large losses. Specific methodology and long‑term performance data are not publicly available.
WallStreet Forex Robot 3.0	• A long‑established scalping robot; marketing materials claim an 80 % win rate with a 6.24 % maximum drawdown. Features include broker‑spy modules and automatic parameter adjustment.	• Performance heavily depends on fast execution; results may vary across brokers. The high win rate is vendor‑reported.
FXStabilizer Pro	• A forex robot focusing on low drawdown; recommended to use a low‑latency VPS near major trading hubs to minimize slippage.	• Limited publicly available documentation; traders must request performance metrics from the vendor.
Crypto trading bots (Coinrule, 3Commas, Cryptohopper, Pionex, etc.)	• The crypto‑tax platform Koinly lists Coinrule, 3Commas, Cryptohopper, Pionex and others as leading bots for 2026. It notes that Cryptohopper offers AI features, strategy marketplaces and supports 17+ exchanges, Pionex is free and beginner‑friendly with 16 built‑in bots, and 3Commas provides advanced customization for experienced traders. Prices vary from free plans to ~$160/month.	• Koinly emphasises that the “best” bot depends on the user’s strategy and exchange and that no bot guarantees profit; features like backtesting and transparent performance metrics should be prioritised.
Reality Check: Research and Expert Warnings

Several independent sources caution against assuming that high win rates or profit factors will be replicated in live trading:

Academic research on AI trading – A 2025 study by economists Gufler, Sangiorgi and Tarantino modelled deep‑reinforcement‑learning traders interacting in a realistic market. They found that while AI traders can match a rational benchmark in isolated simulations, when multiple AI agents interact their trading profits decline and market efficiency deteriorates due to learning externalities. The authors conclude that caution is needed when extrapolating AI trading results to real markets.

Reliability concerns – An article on SentiSight.ai argues that AI trading bots show limited reliability; success is often due to luck rather than algorithmic superiority. It notes that pattern‑recognition algorithms struggle to distinguish true signals from noise and that most commercial bots rely on correlations rather than causal understanding. The piece also warns about biases in training data and vulnerabilities such as model “hallucinations”.

Retail profitability myths – A 2025 analysis by Coincub explains that crypto trading bots automate execution but do not provide intelligence; profit depends on the user’s strategy and risk management. The article notes that while bots can be useful tools, most retail bots barely break even because fees, latency and competition erase any edge. Only traders with significant capital, low‑latency infrastructure and refined strategies consistently profit.

General Steps to Set Up an AI Trading Bot

Regardless of the platform, deploying an AI trading bot involves careful planning and risk management:

Define your trading strategy and goals. Decide whether you will trade forex, stocks, ETFs or crypto. Specify the timeframe (e.g., 15‑minute scalping or daily trend‑following), entry/exit signals, maximum drawdown and risk per trade. Bots are tools to automate a strategy; they cannot invent one for you.

Choose a reputable bot with verified performance. Look for platforms that provide audited results (e.g., Myfxbook for forex bots or transparent backtesting for stock/crypto bots). Evaluate cost, supported markets, and community feedback. Avoid bots with unverifiable claims.

Create an account and connect to your broker or exchange. For MT4/MT5 bots like Forex Fury or GPS Forex Robot, purchase the software, download the expert advisor file (.ex4 or .mq5) and copy it into the Experts folder of your terminal (File → Open Data Folder → MQL4 → Experts). Restart MT4/MT5, then drag the EA onto the chart of your chosen currency pair and timeframe. For platforms like Tickeron or Trade Ideas, sign up on their website, choose an AI agent or strategy, and link your brokerage account via API.

Configure risk settings. Adjust parameters such as lot size, stop‑loss, take‑profit, maximum spread and number of concurrent orders. Forex Fury’s FAQ, for example, advises increasing max_spread if the EA isn’t trading and setting max_orders to 1 for brokers with FIFO rules. Use a demo account or small capital at first.

Backtest and forward‑test. Before risking real money, run the bot on historical data and observe performance metrics (win rate, profit factor, drawdown, Sharpe ratio). Platforms like Trade Ideas and TrendSpider include backtesting tools. For crypto bots, most services offer sandbox or paper‑trading modes.

Deploy on a stable infrastructure. For forex robots, latency matters. Use a low‑latency Virtual Private Server (VPS) near your broker’s server to reduce slippage. Ensure your internet connection and computer remain online 24/7 if you host locally.

Monitor and adjust. Even after automation, monitor trades daily. Market conditions change; adjust risk settings or pause the bot during news events. Vendors often release updated settings—Forex Fury offers new set files regularly.

Stay compliant and manage risk. Keep leverage reasonable and use stop‑loss orders. Consider the regulatory environment in your jurisdiction and ensure that using automated software complies with broker and exchange policies. Never risk funds you cannot afford to lose.

Conclusion

AI trading bots can assist traders by automating execution and scanning markets faster than humans. Some platforms—such as Forex Fury, Tickeron and Trade Ideas—report high win rates and profit factors, but these claims largely come from vendors and may depend on specific strategies or favourable market conditions. Independent analyses caution that AI trading often produces inconsistent results; academic research finds that as more AI agents trade, profits decline and market efficiency worsens. Articles from SentiSight.ai and Coincub warn that automation is not a shortcut to guaranteed profits and that retail traders often underperform due to fees, latency and poor strategy.

If you wish to pursue AI‑driven trading, focus on building a robust strategy, choose transparent bots with verified performance, backtest thoroughly, and start with a demo or low risk. Treat the bot as a tool that augments your trading plan rather than a replacement for research and risk management.

In summary, while platforms such as Forex Fury and Tickeron advertise impressive win rates and profit factors, these claims largely stem from vendor-reported data and may not be independently verified. StockBrokers.com identifies Trade Ideas, StockHero, and TrendSpider as top AI tools, but these serve more as co‑pilots for signal generation and backtesting rather than guaranteed-profit engines. Independent research warns that AI agents can suffer declining profitability when many algorithms interact and that success often owes more to luck or favourable conditions than to algorithmic superiority.

The report outlines practical steps to set up a bot: defining your own strategy, choosing a transparent platform with audited performance, connecting via API, configuring risk parameters, and rigorously backtesting before going live. It stresses using demo accounts, low-latency VPS hosting, and continuous monitoring and adjustment. Importantly, it cautions that automation is not a shortcut to easy money; most retail bots barely break even once fees, latency, and competition are considered. Traders should treat AI as a tool to augment well-designed strategies, not a replacement for sound research and risk management.
Iterates candles stepwise → calls same engine

Stores outputs → metrics + charts
