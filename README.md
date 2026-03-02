# CHRD Indicator Suite of TradingView Pine Scripts

A collection of Pine Script v6 indicators for TradingView, organized into purpose-built families that span the full analytical workflow including strategic trend assessment as well as intraday execution signals.

---

## Table of Contents

- [Overview](#overview)
- [SPY 0DTE Scalper](#spy-0dte-scalper)
- [Market Monitor](#market-monitor)
- [Trend Compass](#trend-compass)
- [Installation](#installation)
- [Disclaimer](#disclaimer)
- [License](#license)

---

## Overview

Each family targets a distinct layer of the analysis process. Together, they form a top-down framework: establish macro context, assess directional bias across a watchlist, then execute with precision timing.

| Layer | Family | Role |
|:------|:-------|:-----|
| Strategic Context | [Trend Compass](#trend-compass) | Multi-day, multi-week, and intraday structural trend phase, strength, and divergence detection |
| Reconnaissance | [Market Monitor](#market-monitor) | Directional bias scoring for multi-chart watchlist monitoring |
| Execution | [SPY 0DTE Scalper](#spy-0dte-scalper) | Intraday signal generation for SPY zero-days-to-expiration options |

All scripts are written in Pine Script v6 and run as chart overlays. Every indicator includes a real-time dashboard, configurable inputs, and a dual alert system (static `alertcondition` entries for TradingView alerts and dynamic `alert()` calls for webhook pipelines).

---

## SPY 0DTE Scalper

Purpose-built for scalping SPY 0DTE options. Each variant is tightly coupled to its specific timeframe, employing strict AND-gate logic across price structure, volume, momentum, and candlestick patterns to filter out low-probability setups.

| Variant | Script | Docs | Base Signal | Key Filters | Target Environment |
|:--------|:-------|:-----|:------------|:------------|:-------------------|
| **1-Minute** | `spy_0dte_scalper_1min.pine` | `spy_0dte_scalper_1min.md` | 9/21 EMA Cross | VWAP, RSI, Engulfing/Pinbar | High-frequency momentum scalping |
| **5-Minute** | `spy_0dte_scalper_5min.pine` | `spy_0dte_scalper_5min.md` | MACD Cross | VWAP, RSI, 9/21 EMA | Standard intraday trend following |
| **15-Minute** | `spy_0dte_scalper_15min.pine` | `spy_0dte_scalper_15min.md` | MACD Cross | VWAP, RSI, 9/21 EMA | Session-level structural swings |

---

## Market Monitor

Designed for multi-chart grid layouts (e.g., 6-8 charts per screen). The Market Monitor replaces cluttered signal arrows with a clean, glanceable directional bias score, allowing you to quickly assess the market breadth and structural alignment of your entire watchlist.

| Variant | Script | Docs | Anchor | Session Tracking | Unique Features |
|:--------|:-------|:-----|:-------|:-----------------|:----------------|
| **1-Minute** | `market_monitor_1min.pine` | `market_monitor_1min.md` | Session VWAP | N/A | Lightweight, adaptive, equal-weight scoring |
| **5-Minute** | `market_monitor_5min.pine` | `market_monitor_5min.md` | 15m EMA | 5m Phase Intervals | TTM Squeeze, TICK, Weighted Scoring (-11 to +11) |
| **15-Minute** | `market_monitor_15min.pine` | `market_monitor_15min.md` | 1H EMA | 15m Phase Intervals | TTM Squeeze, TICK, Session-level stability bias |

---

## Trend Compass

The strategic context layer. Evaluates the lifecycle of a trend (Emerging → Accelerating → Mature → Exhausting → Consolidating → Reversing) while plotting institutional-grade higher-timeframe reference levels directly on the local chart.

| Variant | Script | Docs | HTF References | Divergence Config | Cadence |
|:--------|:-------|:-----|:---------------|:------------------|:--------|
| **Daily** | `trend_compass_daily.pine` | `trend_compass_daily.md` | Weekly 50, Prior M/W H/L/C | 5L/5R pivots, 60-bar decay | 1 bar per session |
| **4-Hour** | `trend_compass_4h.pine` | `trend_compass_4h.md` | Daily 50, Daily 200, Weekly 50 | 4L/2R pivots, 20-bar decay | ~2 bars per session |
| **1-Hour** | `trend_compass_1h.pine` | `trend_compass_1h.md` | 4H 50, Daily 50, Daily 200, Weekly 50 | 3L/2R pivots, 30-bar decay | ~6.5 bars per session |
| **15-Minute** | `trend_compass_15m.pine` | `trend_compass_15m.md` | 1H 50, 4H 50, Daily 50, Daily 200 | 3L/2R pivots, 40-bar decay | 26 bars per session |

Each variant is calibrated for its timeframe: compression thresholds, EMA slope sensitivity, crossover windows, and divergence pivot parameters all scale appropriately. The 15-minute and 1-hour variants provide the fastest divergence feedback and intraday trend assessment, while the Daily variant provides the broadest structural picture with the least noise.

---

## Installation

1. Open a chart in [TradingView](https://www.tradingview.com/).
2. Open the Pine Editor (bottom panel).
3. Copy the contents of the desired `.pine` file and paste it into the editor.
4. Click **Add to chart**.
5. Configure inputs through the indicator settings dialog as needed.

Each script's documentation file covers all available inputs, dashboard layout, alert configuration, and tuning recommendations for its intended timeframe.

---

## Disclaimer

These indicators are analytical tools for educational and informational purposes only. They do not constitute financial advice, trading recommendations, or solicitations to buy or sell any financial instrument. Past performance of any indicator or signal does not guarantee future results. Trading involves substantial risk of loss. Always conduct your own analysis and consult with a qualified financial professional before making trading decisions.

---

## License

[MIT](LICENSE) -- Copyright (c) 2026 Chris Thomas
