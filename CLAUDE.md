# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pine Script v6 indicator suite for TradingView. Eight `.pine` scripts organized into three families, each with matching documentation in `docs/`.

| Family | Purpose | Scripts |
|--------|---------|--------|
| `spy_0dte_scalper/` | Intraday signal generation for SPY 0DTE options | 1min, 5min, 15min |
| `market_monitor/` | Watchlist bias overlays | 1min (lightweight), 5min (weighted scoring) |
| `trend_compass/` | Strategic trend assessment | Daily, 4H, 1H |

## Development Workflow

There is no build system, linter, or test framework. Pine Script v6 is validated exclusively by pasting into the TradingView Pine Editor and adding the indicator to a chart. The edit cycle is: modify locally → paste into TradingView → verify on chart → commit.

## Naming Conventions

- **Inputs**: `i_` prefix (`i_emaFastLen`, `i_showDash`)
- **Colors**: `C_` prefix, defined as `var color` constants (`C_BULL`, `C_BEAR`, `C_NEUTRAL`)
- **Input groups**: `GP_` prefix (`GP_EMA`, `GP_VWAP`, `GP_DASH`)
- **Files**: `{script_name}_{timeframe}.pine` — no version suffixes in filenames
- **Version**: Header comment only (e.g., `// Version: 1.1.0`)

## Architecture Patterns

### Anti-Repaint Rule
All signals must be gated by `barstate.isconfirmed`. Dashboard/table updates only on `barstate.islast`.

### EMA Ribbon
Every script uses a 3-EMA ribbon (Fast/Mid/Slow) with toggleable cloud fill. Lengths vary by timeframe.

### Scoring Systems
- Market Monitor 1min: Binary bias score (-5 to +5), five ±1 conditions
- Market Monitor 5min: Weighted bias score (-11 to +11), structural conditions at 2x weight
- Trend Compass: Composite score (-11 to +11), 8 conditions across structural (2x) and confirmation (1x) tiers

### Signal AND-Gate Logic (SPY Scalpers)
Signals require all conditions true simultaneously: base + VWAP + EMA + RSI + candle pattern. Example:
```pinescript
bool callsSignal = callsBase and callsVwapOk and callsEmaOk and callsRsiOk and callsCandle
```

### Higher Timeframe Context
Uses `request.security()` with `lookahead=barmerge.lookahead_off` for live HTF EMAs, `lookahead_on` only for historical reference levels (prior day/week/month H/L/C).

### Alert Dual System
- `alertcondition()`: Static, compile-time constant strings (Pine v6 requirement — no runtime string interpolation)
- `alert()`: Dynamic, runtime string interpolation for webhook pipelines

### State Management
All persistent variables use `var` declarations. Example: `var int barsSinceSignal = 999`.

### Dashboard Tables
Consistent dark theme: background `#1A1A2E`, text `#E0E0E0`, header accent `#7C4DFF`. Position and text size configurable via inputs. Built with `table.cell()` calls, rendered only on `barstate.islast`.

## Pine v6 Gotchas

- `alertcondition()` messages must be compile-time string literals — no `nz()` on string series, no runtime interpolation
- `request.security()` calls should be minimized; use appropriate lookahead setting based on whether data is live vs historical
- VWAP is intraday-only; guard with `timeframe.isintraday`

## Documentation

Each `.pine` has a matching `.md` in `docs/` covering components, tuning guides, dashboard layout, and known limitations.
