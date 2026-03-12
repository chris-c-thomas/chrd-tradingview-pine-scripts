# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pine Script v6 indicator suite for TradingView. Scripts live in `scripts/`, organized into four families, each with matching documentation in `docs/`.

| Family | Purpose | Scripts |
|--------|---------|--------|
| `spy-0dte-scalper/` | Intraday signal generation for SPY 0DTE options | 1min, 5min, 15min |
| `market-monitor/` | Watchlist bias overlays for multi-chart grids | 1min, 5min, 15min |
| `trend-compass/` | Strategic trend assessment (ticker-agnostic) | Daily, 4H, 1H, 15min |
| `intraday-analyst/` | Intraday signal generation + context for equities, ETFs, and cash indices | equity, index |

## Development Workflow

There is no build system, linter, or test framework. Pine Script v6 is validated exclusively by pasting into the TradingView Pine Editor and adding the indicator to a chart. The edit cycle is: modify locally → paste into TradingView → verify on chart → commit.

## Naming Conventions

- **Inputs**: `i_` prefix (`i_emaFastLen`, `i_showDash`)
- **Colors**: `C_` prefix, defined as `var color` constants (`C_BULL`, `C_BEAR`, `C_NEUTRAL`)
- **Input groups**: `GP_` prefix (`GP_EMA`, `GP_VWAP`, `GP_DASH`)
- **Files**: `{family-name}-{timeframe}.pine` — no version suffixes in filenames. Exception: `intraday-analyst/` uses `{variant}-intraday-analyst.pine` (e.g., `equity-intraday-analyst.pine`, `index-intraday-analyst.pine`)
- **Version**: Header comment only (e.g., `// Version: 1.1.0`)

## Architecture Patterns

### Script Structure

Every script follows the same section order, delimited by `// ===` comment blocks:
1. Header (title, timeframe, version, author)
2. Constants (group strings `GP_*`, color constants `C_*`)
3. Inputs (organized by `group=GP_*`)
4. Helper functions (`f2()`, `posnegColor()`, `biasColor()`, `biasLabel()`)
5. Core indicator calculations (EMAs, VWAP, RSI, ADX, ATR)
6. Optional modules (TTM Squeeze, NYSE TICK, Ichimoku, Fibonacci)
7. Composite scoring engine
8. Plots (EMA ribbon, VWAP, key levels)
9. Dashboard table (rendered on `barstate.islast`)
10. Alerts (`alertcondition()` + `alert()`)

### Anti-Repaint Rule

All signals must be gated by `barstate.isconfirmed`. Dashboard/table updates only on `barstate.islast`.

### EMA Ribbon

Every script uses a 3-EMA ribbon (Fast/Mid/Slow) with toggleable cloud fill. Lengths vary by timeframe. Trend Compass scripts add a 200 EMA anchor.

### Scoring Systems

Each family uses a different scoring model:

- **SPY Scalper**: No score — AND-gate logic. All conditions must be true simultaneously (base + VWAP + EMA + RSI + candle pattern):
  ```pinescript
  bool callsSignal = callsBase and callsVwapOk and callsEmaOk and callsRsiOk and callsCandle
  ```
- **Market Monitor**: Weighted bias score. Structural conditions (VWAP, EMA, Anchor) at 2x weight, confirmation conditions (RSI, DI, TICK, Squeeze) at 1x. Max score varies by variant.
- **Trend Compass**: Composite weighted trend score (-11 to +11). 8 conditions across structural (2x) and confirmation (1x) tiers. Includes trend phase classifier with 6 states (Emerging → Accelerating → Mature → Exhausting → Consolidating → Reversing).
- **Intraday Analyst**: Weighted bias score with signal generation. Equity variant: 3 structural (2x) + 8 confirmation (1x), max +/-15. Index variant: 3 structural + 7 confirmation, max +/-13. Index variant substitutes TWAP for VWAP and a configurable volatility index (VIX/VXN/RVX) for OBV. Both variants are timeframe-adaptive (1m/5m/10m/15m).

### Higher Timeframe Context

Uses `request.security()` with `lookahead=barmerge.lookahead_off` for live HTF EMAs, `lookahead_on` only for historical reference levels (prior day/week/month H/L/C).

### Alert Dual System

- `alertcondition()`: Static, compile-time constant strings (Pine v6 requirement — no runtime string interpolation)
- `alert()`: Dynamic, runtime string interpolation for webhook pipelines

### State Management

All persistent variables use `var` declarations. Example: `var int barsSinceSignal = 999`.

### Dashboard Tables

Consistent dark theme: background `#1A1A2E`, text `#E0E0E0`/`#B0BEC5`, header accent `#7C4DFF`. Position and text size configurable via inputs. Built with `table.cell()` calls on 3-column grids, rendered only on `barstate.islast`.

### Color Palette (shared across all scripts)

- Bull: `#00E676`, Bear: `#FF1744`, Neutral: `#B0BEC5`
- Yellow: `#FFD600`, Orange: `#FF9100`, Cyan: `#00BCD4`, Purple: `#7C4DFF`
- VWAP: `#FFFFFF`, Gray: `#616161`/`#78909C`

## Pine v6 Gotchas

- `alertcondition()` messages must be compile-time string literals — no `nz()` on string series, no runtime interpolation
- `nz()` only works on numeric types (`int`, `float`). For `string` or `color` series, use `na(x) ? fallback : x` instead
- `range` is a reserved keyword — cannot be used as a variable name
- Multi-line ternary chains do NOT work with `: ` at end-of-line as a continuation operator. Either collapse to a single line or wrap in parentheses (implicit continuation inside `()` is valid)
- Local function definitions (`name(...) =>`) are not allowed inside `if` blocks. Define at global scope or inline the calls
- `ta.pivotlow()` / `ta.pivothigh()` must be called at global scope for consistent execution. Calling inside `if` blocks or `for` loops produces incorrect values. Store pivot results in persistent arrays for lookback comparison
- `request.security()` calls should be minimized; use appropriate lookahead setting based on whether data is live vs historical
- VWAP is intraday-only; guard with `timeframe.isintraday`
- All `indicator()` calls use `overlay=true` — these are chart overlays, not separate panes

## Documentation

Each `.pine` has a matching `.md` in `docs/` covering components, tuning guides, dashboard layout, and known limitations.
