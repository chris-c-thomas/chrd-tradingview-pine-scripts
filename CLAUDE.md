# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pine Script v6 indicator suite for TradingView. Scripts live in `scripts/`, organized into six families, each with matching documentation in `docs/`.

| Family | Purpose | Scripts |
|--------|---------|--------|
| `trend-compass/` | Strategic trend assessment (ticker-agnostic) | Daily, 4H, 1H, 15min |
| `bottom-level-finder/` | Multi-horizon correction / re-entry framework | Unified (7 horizon modes) |
| `market-monitor/` | Watchlist bias overlays for multi-chart grids | 1min, 5min, 15min |
| `intraday-analyst/` | Intraday signal generation + context for equities, ETFs, and cash indices | equity, index |
| `spy-0dte-scalper/` | Intraday signal generation for SPY 0DTE options | 1min, 5min, 15min |
| `pocket-scout/` | Mobile-first intraday bias tool for iPhone viewing | Unified (adaptive 1m/5m/15m) |

## Development Workflow

There is no build system, linter, or test framework. Pine Script v6 is validated exclusively by pasting into the TradingView Pine Editor and adding the indicator to a chart. The edit cycle is: modify locally → paste into TradingView → verify on chart → commit.

## Naming Conventions

- **Inputs**: `i_` prefix (`i_emaFastLen`, `i_showDash`)
- **Colors**: `C_` prefix, defined as `var color` constants (`C_BULL`, `C_BEAR`, `C_NEUTRAL`)
- **Input groups**: `GP_` prefix (`GP_EMA`, `GP_VWAP`, `GP_DASH`)
- **Files**: `{family-name}-{timeframe}.pine` — no version suffixes in filenames. Exceptions:
  - `intraday-analyst/` uses `{variant}-intraday-analyst.pine` (e.g., `equity-intraday-analyst.pine`, `index-intraday-analyst.pine`)
  - `bottom-level-finder/` uses `bottom-level-finder.pine` (single unified script; timeframe controlled via `Horizon Mode` input)
  - `pocket-scout/` uses `pocket-scout.pine` (single unified script; chart TF auto-detected)
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

Every script uses a 3-EMA ribbon (Fast/Mid/Slow) with toggleable cloud fill, with the exception of Bottom Level Finder (which uses a 4-MA mode-adaptive suite) and Pocket Scout (3 EMAs, no cloud fill by mobile clutter discipline). Lengths vary by timeframe. Trend Compass scripts add a 200 EMA anchor.

### Scoring Systems

Each family uses a different scoring model:

- **SPY Scalper**: No score — AND-gate logic. All conditions must be true simultaneously (base + VWAP + EMA + RSI + candle pattern):

  ```pinescript
  bool callsSignal = callsBase and callsVwapOk and callsEmaOk and callsRsiOk and callsCandle
  ```

- **Market Monitor**: Weighted bias score. Structural conditions (VWAP, EMA, Anchor) at 2x weight, confirmation conditions (RSI, DI, TICK, Squeeze) at 1x. Max score varies by variant.
- **Trend Compass**: Composite weighted trend score (-11 to +11). 8 conditions across structural (2x) and confirmation (1x) tiers. Includes trend phase classifier with 6 states (Emerging → Accelerating → Mature → Exhausting → Consolidating → Reversing).
- **Intraday Analyst**: Weighted bias score with signal generation. Equity variant: 3 structural (2x) + 8 confirmation (1x), max +/-15. Index variant: 3 structural + 7 confirmation, max +/-13. Index variant substitutes TWAP for VWAP and a configurable volatility index (VIX/VXN/RVX) for OBV. Both variants are timeframe-adaptive (1m/5m/10m/15m).
- **Bottom Level Finder**: No composite score. Uses a 6-factor bottom signal scorecard (RSI oversold, BB %B breach, momentum turn, MA support test, VIX elevation, volume capitulation), each factor fires or doesn't fire. Bottom Proximity classification rolls the scorecard into NONE / LOW / MODERATE / ELEVATED / EXTREME.
- **Pocket Scout**: Weighted bias score with 6 conditions (vs 8-11 in desktop indicators). Structural 2x (EMA stack, VWAP, HTF Anchor) + Confirmation 1x (RSI, MACD, Vol index slope). Max +/-9 weighted, +/-6 equal. Deliberately minimal to preserve mobile information budget.

### Higher Timeframe Context

Uses `request.security()` with `lookahead=barmerge.lookahead_off` for live HTF EMAs, `lookahead_on` only for historical reference levels (prior day/week/month H/L/C).

### Tuple Consolidation

Same-timeframe `request.security()` calls should be collapsed into single tuple-returning calls to minimize CPU load. This is the primary CPU optimization lever in the suite. Example from Pocket Scout:

```pinescript
[pdh, pdl, pdc, dayOpen] = request.security(
     syminfo.tickerid, "D",
     [high[1], low[1], close[1], open],
     lookahead=barmerge.lookahead_on)
```

Each family has a documented `request.security()` budget in its docs. Per-family approximate budgets:

| Family | Budget |
|--------|--------|
| Market Monitor (1m/5m/15m) | 3-4 calls |
| Trend Compass (15m/1H/4H/Daily) | 5-7 calls |
| Intraday Analyst (equity/index) | 6-10 calls |
| SPY 0DTE Scalper (1m/5m/15m) | 5-8 calls (+ `request.footprint()`) |
| Bottom Level Finder | 3 calls |
| Pocket Scout | 4 calls |

### Alert Dual System

- `alertcondition()`: Static, compile-time constant strings (Pine v6 requirement — no runtime string interpolation)
- `alert()`: Dynamic, runtime string interpolation for webhook pipelines

Exception: **Bottom Level Finder** is alert-free by design — it is a visual chart analysis tool, not a notification pipeline.

### State Management

All persistent variables use `var` declarations. Example: `var int barsSinceSignal = 999`.

### Dashboard Tables

Consistent dark theme: background `#1A1A2E` (desktop) / `#0D1117` (mobile), text `#E0E0E0`/`#B0BEC5`/`#ECEFF1`, header accent `#7C4DFF`. Position and text size configurable via inputs. Built with `table.cell()` calls, rendered only on `barstate.islast`.

Dashboard row counts by family (indicative):

| Family | Typical Rows | Default Size | Default Position |
|--------|------|------|------|
| Market Monitor 1m | 12 | Normal | Bottom Right |
| Market Monitor 5m/15m | 18 | Normal | Bottom Right |
| Trend Compass (all) | 17-19 | Small | Top Right |
| Intraday Analyst | 24-25 | Small | Bottom Right |
| SPY 0DTE Scalper 1m/5m/15m | 16-26 | Small | Top Right |
| Bottom Level Finder | ~30-40 (padded, 5 cols) | Small | Top Right |
| Pocket Scout | **8** (rows 0-1 merged banner) | **Huge** | **Top Right** |

Pocket Scout is the outlier: mobile-first design uses `size.huge` as the default and a merged banner row for the bias readout. See `scripts/pocket-scout/pocket-scout.pine` for reference.

### Color Palette (shared across all scripts)

- Bull: `#00E676`, Bear: `#FF1744`, Neutral: `#B0BEC5`
- Yellow: `#FFD600`, Orange: `#FF9100`, Cyan: `#00BCD4`, Purple: `#7C4DFF`
- VWAP: `#FFFFFF`, Gray: `#616161`/`#78909C`

Pocket Scout additionally uses vivid banner colors for its merged bias row: `#00C853` (strong bull), `#D50000` (strong bear), with translucent variants for moderate states.

## Pine v6 Gotchas

- `alertcondition()` messages must be compile-time string literals — no `nz()` on string series, no runtime interpolation
- `nz()` only works on numeric types (`int`, `float`). For `string` or `color` series, use `na(x) ? fallback : x` instead
- `range` is a reserved keyword — cannot be used as a variable name
- Multi-line ternary chains do NOT work with `:` at end-of-line as a continuation operator. Either collapse to a single line or wrap in parentheses (implicit continuation inside `()` is valid)
- Local function definitions (`name(...) =>`) are not allowed inside `if` blocks. Define at global scope or inline the calls
- `ta.pivotlow()` / `ta.pivothigh()` must be called at global scope for consistent execution. Calling inside `if` blocks or `for` loops produces incorrect values. Store pivot results in persistent arrays for lookback comparison
- `ta.*` functions must execute on every bar — cannot be called conditionally inside `if barstate.islast` blocks. The `if barstate.islast` block should only touch table rendering, never compute indicators.
- `ta.macd()` returns a 3-element tuple; destructure with `[macdLine, signalLine, histLine] = ta.macd(...)`. `[]` is the history-reference operator, not a tuple index operator.
- Compact single-line `switch` branches (comma-separated) are invalid; one branch per line is required
- Multi-line expressions need outer parentheses for line continuation
- Explicit `int` type annotations force `const` qualification on `input`-derived values — use type inference instead
- Helper functions can promote string parameters to `series` type unexpectedly — be deliberate about parameter types
- `request.security()` calls should be minimized and tuple-consolidated where possible; use appropriate lookahead setting based on whether data is live vs historical
- `request.security()` LTF calls (target TF lower than chart TF) multiply bar processing — primary CPU bottleneck source
- `request.footprint()` counts toward the shared `request.*()` budget (40/64-call limit) and repaints by design (no `lookahead` parameter)
- VWAP is intraday-only; guard with `timeframe.isintraday`
- All `indicator()` calls use `overlay=true` — these are chart overlays, not separate panes

## Mobile vs Desktop Design Considerations

Pocket Scout represents a distinct design philosophy from the rest of the suite. When modifying or extending mobile-targeted indicators, respect these constraints:

- **Row count is a hard budget**, not a soft guideline. On iPhone portrait, every row above ~10 at `size.huge` pushes the dashboard toward covering the chart.
- **Large text over small text**: `size.huge` is counterintuitive on desktop but mandatory on iPhone — small text is illegible at viewing distance.
- **Merged full-width banner rows** are a mobile pattern for high-priority readouts (bias score, regime classification). Uses `table.merge_cells()` across a 2-column table.
- **`Top Right` default position**: TradingView's mobile toolbar consumes ~80pt at the bottom of the iPhone screen, making bottom-anchored dashboards partially occluded.
- **Cut, don't compress**: If a feature can't earn its place in an 8-row dashboard for a 2-second glance read, it belongs in a desktop indicator instead.
- **Minimal chart overlays**: No cloud fills, no labels, no boxes. `plot.style_linebr` for session levels so they don't span the chart.

Desktop indicators should continue to prioritize analytical depth over visual simplicity; Pocket Scout is not a template to migrate the rest of the suite toward.

## Documentation

Each `.pine` has a matching `.md` in `docs/` covering components, tuning guides, dashboard layout, known limitations, and version history. Every `.pine` file change should trigger corresponding updates to its `.md` doc, `README.md`, `CLAUDE.md`, and the family-level `CHANGELOG.md` where one exists.
