---
type: repository guide
title: Repository Quickstart
description: A concise orientation to ml4t-backtest development, its stable public API, runtime ownership boundaries, and the right wiki page for each common change.
tags: [backtesting, development, public-api, navigation]
verified:
  - by: openwiki/0.9.0
    at: 2026-11-10T03:14:31.122Z
sources:
  - id: openwiki-source-8037e2358a2c4f9b2c722a11
    resource: repo://AGENTS.md
  - id: openwiki-source-879c710378f4939b0b8f9110
    resource: repo://docs/getting-started/quickstart.md
  - id: openwiki-source-71dc8e9b2989b7a05397492b
    resource: repo://examples/offline_migration.py
  - id: openwiki-source-924573ecd08a59c275295fc4
    resource: repo://examples/stateful_strategies.py
  - id: openwiki-source-05ccef8d4cf1698187f20464
    resource: repo://pyproject.toml
  - id: openwiki-source-23775c3de52f3ab95a13cb8b
    resource: repo://README.md
  - id: openwiki-source-6561de57cb1814f9b9c27ae4
    resource: repo://src/ml4t/backtest/__init__.py
  - id: openwiki-source-4086dc0caf41408e8e8f1736
    resource: repo://src/ml4t/backtest/AGENTS.md
  - id: openwiki-source-dd952e38101b1434ebcc6932
    resource: repo://src/ml4t/backtest/engine.py
  - id: openwiki-source-ecdd6827e8e8d73db754d8be
    resource: repo://src/ml4t/backtest/strategy.py
generated: { by: "openwiki/0.9.0", at: "2026-11-10T03:14:31.122Z" }
---

# Repository Quickstart

`ml4t-backtest` is the event-driven simulation package in the ML4T ecosystem. It implements configurable execution, accounting, risk, and result capture while sharing runtime-neutral lifecycle and order contracts from `ml4t.specs`. Start with the package-level API rather than importing implementation modules:

```python
from ml4t.backtest import BacktestConfig, DataFeed, Engine, Strategy, run_backtest
```

The broader package export surface also includes `Broker`, `BacktestResult`, `RuntimeServices`, canonical order/fill/position types, risk rules, rebalancing helpers, lifecycle helpers, and artifact errors. Treat exports from `ml4t.backtest` as the user-facing compatibility surface; internal modules can evolve behind them.

## Set up a development checkout

The project requires Python 3.12 or newer and uses `uv`. For a locked development environment with documentation tools:

```bash
git clone https://github.com/ml4t/backtest.git
cd backtest
uv sync --dev --extra docs --locked
```

Run focused tests while iterating, then the applicable repository gates before handing off a change:

```bash
uv run ruff check src/ tests/ validation/
uv run ruff format --check src/ tests/ validation/
uv run ty check
uv run pytest tests/ -q
uv run mkdocs build --strict
pre-commit run --all-files
```

Pytest normally discovers `tests/test_*.py`, enforces registered markers, checks coverage, and excludes private and optional cross-framework validation suites. Release-critical validation has a separate allowlist in [`validation/release_checks.txt`](../validation/release_checks.txt); use [Testing, Parity Evidence, and Release Qualification](testing/quality-parity-and-release.md) before changing behavior, compatibility claims, or release evidence. Read the nearest nested `AGENTS.md` before editing a subsystem, beginning with the [repository guide](../AGENTS.md).

## Run and inspect a backtest

A strategy subclasses `Strategy` and implements `on_data(timestamp, data, context, broker)`. `on_start`, `on_prepare`, and `on_end` are optional lifecycle hooks. Build a `DataFeed`, pass it with the strategy and a `BacktestConfig` to `Engine`, and call `run()`:

```python
from ml4t.backtest import BacktestConfig, DataFeed, Engine, Strategy

class MyStrategy(Strategy):
    def on_data(self, timestamp, data, context, broker):
        if broker.get_position("AAPL") is None:
            broker.submit_order("AAPL", 10)

feed = DataFeed(prices_df=prices)
result = Engine(feed, MyStrategy(), BacktestConfig()).run()
```

For a complete executable example with bundled synthetic data and explained fills, use [`docs/getting-started/quickstart.md`](../docs/getting-started/quickstart.md). `run_backtest(...)` is the compact alternative: it accepts price, signal, and context frames or paths, constructs the feed, resolves a named configuration preset when supplied, runs an engine, and returns a `BacktestResult`.

An `Engine` is single-use. Its high-level lifecycle is startup and preparation, sequential accepted bars, finalization, then result construction. The engine owns callback and per-bar ordering; the broker owns orders, fills, positions, cash, and its execution/accounting collaborators. The engine does **not** automatically liquidate open positions at end of data, so results may contain marked open positions. See [Event-Driven Runtime and Strategy Lifecycle](architecture/event-driven-runtime.md) before changing any ordering or callback semantics.

Inspect evidence, not only summary metrics:

```python
result.to_fills_dataframe()
result.to_trades_dataframe()
result.to_equity_dataframe()
result.metrics
```

Fill timing, prices, costs, calendars, account policy, and capture behavior are configuration semantics, not incidental defaults. See [Configuration, Presets, and Behavioral Semantics](concepts/configuration-and-profiles.md), and use [Results, Analytics, and Reproducible Artifacts](workflows/results-analytics-and-artifacts.md) for result schemas, persistence, and reload behavior.

## Runtime ownership at a glance

| Domain | Primary responsibility | Start here |
|---|---|---|
| Engine and lifecycle | Run-scoped orchestration, callback phases, session/bar ordering, and result assembly | [Event-Driven Runtime and Strategy Lifecycle](architecture/event-driven-runtime.md) |
| Feed and time | Input normalization, schema contracts, calendars, timestamps, and causal alignment | [Market Data, Feed Contracts, and Time Semantics](concepts/market-data-and-time.md) |
| Runtime views | Causal market, signal, feature, history, instrument, auxiliary, and scheduler services | [Causal Runtime Services and Scheduling](architecture/runtime-services.md) |
| Broker and accounting | Submission, eligibility, fills, costs, positions, cash/margin, and portfolio ledger mutation | [Broker, Order Execution, and Accounting](architecture/broker-execution-and-accounting.md) |
| Risk and allocation | Stops, limits, halts, target weights, and rebalance schedules | [Risk Controls, Stops, and Rebalancing](architecture/risk-controls-and-rebalancing.md) |
| Non-order transitions | Corporate actions, futures rolls/settlement, funding, capital flows, and instrument lifetime | [Instrument, Corporate Action, Futures, Funding, and Capital Lifecycles](concepts/instrument-and-cash-lifecycles.md) |
| Compatibility | Offline adaptation of Zipline conventions onto the native engine and broker ledger | [Zipline Compatibility and Offline Migration](integrations/zipline-compatibility.md) |
| Outputs | Metrics, fills, trades, equity, diagnostics, manifests, and versioned artifacts | [Results, Analytics, and Reproducible Artifacts](workflows/results-analytics-and-artifacts.md) |

## Route common changes

- **Change callback timing, same-bar/next-bar behavior, or cleanup:** begin with the [runtime lifecycle](architecture/event-driven-runtime.md), then test causal and ordering invariants.
- **Add a data column, quote source, calendar rule, or publication-time contract:** follow [market data and time](concepts/market-data-and-time.md); use [runtime services](architecture/runtime-services.md) for callback-visible or historical views.
- **Change order eligibility, fill pricing, commission/slippage, positions, cash, or margin:** follow [broker, execution, and accounting](architecture/broker-execution-and-accounting.md). Preserve the broker as the authoritative ledger owner.
- **Add stops, portfolio controls, forced exits, or rebalancing:** follow [risk controls and rebalancing](architecture/risk-controls-and-rebalancing.md), including cause propagation through execution.
- **Add or alter a configuration field or preset:** follow [configuration and profiles](concepts/configuration-and-profiles.md). State parity against an explicit configuration, never an assumed external-framework default.
- **Change corporate actions, futures, funding, deposits, or withdrawals:** follow [instrument and cash lifecycles](concepts/instrument-and-cash-lifecycles.md).
- **Change result fields or artifact files:** follow [results and artifacts](workflows/results-analytics-and-artifacts.md); public artifacts are versioned compatibility surfaces and need migration coverage.
- **Port Zipline code:** use [Zipline compatibility](integrations/zipline-compatibility.md) and the offline examples in [`examples/`](../examples/), rather than creating a second execution engine.
- **Qualify any behavioral change:** use [testing and release qualification](testing/quality-parity-and-release.md) for focused regressions, invariants, optional parity environments, performance evidence, and final gates.

The examples are learning and migration aids, not additional public API. In particular, [`examples/stateful_strategies.py`](../examples/stateful_strategies.py) demonstrates feedback-driven strategies, while [`examples/offline_migration.py`](../examples/offline_migration.py) demonstrates an offline data-to-features-to-backtest-to-diagnostics workflow.
