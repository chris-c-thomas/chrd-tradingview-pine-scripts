# TradingView Pine Script Indicator Suite

A collection of Pine Script v6 indicators for TradingView, organized into purpose-built families that span the full analytical workflow — from strategic trend assessment down to intraday execution signals.

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
| Strategic Context | [Trend Compass](#trend-compass) | Multi-day and multi-week trend phase, strength, and divergence detection |
| Reconnaissance | [Market Monitor](#market-monitor) | Directional bias scoring for multi-chart watchlist monitoring |
| Execution | [SPY 0DTE Scalper](#spy-0dte-scalper) | Intraday signal generation for SPY zero-days-to-expiration options |

All scripts are written in Pine Script v6 and run as chart overlays. Every indicator includes a real-time dashboard, configurable inputs, and a dual alert system (static `alertcondition` entries for TradingView alerts and dynamic `alert()` calls for webhook pipelines).

---

## SPY 0DTE Scalper

Purpose-built for scalping SPY 0DTE options. Each timeframe variant generates explicit CALLS and PUTS signal labels using AND-gate confluence logic — all required conditions must pass simultaneously before a signal fires. Signals are gated by `barstate.isconfirmed` to prevent repainting.

Core components shared across all variants:

- **EMA Ribbon** with dynamic cloud fill for trend structure
- **Session VWAP** with standard deviation bands
- **TTM Squeeze** detection (Bollinger Band compression inside Keltner Channels)
- **NYSE TICK Index** integration with six-tier breadth classification
- **Session levels** — Pre-Market High/Low, Opening Range, Session HOD/LOD
- **Prior Day VWAP** as an institutional reference level
- **Regime classifier** — BULLISH, BEARISH, RANGING, NO TRADE, TRANSITION
- **Signal filters** — time window, cooldown, bar confirmation, ADX gate, VWAP extension, optional squeeze and TICK filters

### Variants

| Timeframe | Script | Documentation | Hold Duration | Signals per Session |
|:----------|:-------|:--------------|:--------------|:--------------------|
| 1-minute | [spy_0dte_scalper_1min.pine](spy_0dte_scalper/spy_0dte_scalper_1min.pine) | [docs](docs/spy_0dte_scalper_1min.md) | 1 -- 5 min | Highest frequency |
| 5-minute | [spy_0dte_scalper_5min.pine](spy_0dte_scalper/spy_0dte_scalper_5min.pine) | [docs](docs/spy_0dte_scalper_5min.md) | 5 -- 25 min | Moderate frequency |
| 15-minute | [spy_0dte_scalper_15min.pine](spy_0dte_scalper/spy_0dte_scalper_15min.pine) | [docs](docs/spy_0dte_scalper_15min.md) | 15 -- 60 min | Highest conviction |

The 1-minute variant uses a strict AND-gate engine requiring all five core conditions. The 5-minute and 15-minute variants use a weighted scoring system that allows partial confluence while still requiring minimum thresholds, and add session phase detection (Open Drive, Morning, Midday, Power Hour, Close) with bias trend tracking.

---

## Market Monitor

Lightweight directional-bias overlays designed for multi-chart watchlist monitoring across 6-8 simultaneous charts. These indicators produce no signal labels — they exist purely to communicate trend context, momentum state, and directional bias at a glance.

Core components shared across both variants:

- **EMA Ribbon** (9/21/50) with trend cloud fill
- **Session VWAP** with configurable standard deviation bands
- **RSI and ADX** momentum engine
- **ATR** volatility measurement with regime classification
- **Relative volume** gauge
- **Higher timeframe EMA** confirmation layer
- **Previous Day High/Low/Close** reference levels
- **Compact dashboard** optimized for small chart tiles

### Variants

| Timeframe | Script | Documentation | Bias Score Range | Key Additions |
|:----------|:-------|:--------------|:-----------------|:--------------|
| Adaptive (1m -- Daily) | [market_monitor_1min.pine](market_monitor/market_monitor_1min.pine) | [docs](docs/market_monitor_1min.md) | -5 to +5 | Binary scoring, minimal object count for 8+ instances |
| 5-minute | [market_monitor_5min.pine](market_monitor/market_monitor_5min.pine) | [docs](docs/market_monitor_5min.md) | -11 to +11 | Weighted scoring, session phases, TICK, TTM Squeeze, Opening Range |

The 1-minute variant uses a simple binary scoring system where five conditions each contribute +/-1 point. The 5-minute variant introduces weighted scoring (structural conditions at 2x, confirmation conditions at 1x), NYSE TICK integration, TTM Squeeze detection, session HOD/LOD tracking, Opening Range levels, and bias trend tracking (IMPROVING / STABLE / FADING).

---

## Trend Compass

Strategic trend assessment overlays for equities, ETFs, and indices. Ticker-agnostic by design — these work on any liquid instrument. No signal generation. Pure trend context answering: What is the trend? How strong is it? Accelerating or exhausting? Are divergences forming? Where is price relative to institutional structure?

Core components shared across all variants:

- **EMA Ribbon** (10/21/50) with 200 EMA anchor as the institutional dividing line
- **Trend phase classifier** with six states — EMERGING, ACCELERATING, MATURE, EXHAUSTING, CONSOLIDATING, REVERSING
- **Composite weighted trend score** (-11 to +11) from eight conditions across structural (2x weight) and confirmation (1x weight) tiers
- **Score trend tracking** — IMPROVING, STABLE, FADING via smoothed comparison
- **RSI divergence detection** — bullish, bearish, hidden bullish, hidden bearish with visual line segments on the chart
- **MACD divergence detection** with four states and dotted line visualization
- **ADX slope analysis** — RISING, FLAT, FALLING
- **OBV trend confirmation** — CONFIRMING, DIVERGING, NEUTRAL
- **ATR percentile ranking** and **Bollinger Band width percentile** for volatility context
- **52-week High/Low** with position-in-range percentage
- **Prior period High/Low/Close** reference levels
- **Optional modules** — Fibonacci retracement and Ichimoku Cloud (togglable)

### Variants

| Timeframe | Script | Documentation | HTF Anchors | Divergence Sensitivity | Update Frequency |
|:----------|:-------|:--------------|:------------|:-----------------------|:-----------------|
| Daily | [trend_compass_daily.pine](trend_compass/trend_compass_daily.pine) | [docs](docs/trend_compass_daily.md) | Weekly 50 EMA | 5L/3R pivots, 15-bar decay | 1 bar per session |
| 4-hour | [trend_compass_4h.pine](trend_compass/trend_compass_4h.pine) | [docs](docs/trend_compass_4h.md) | Daily 50, Daily 200, Weekly 50 | 4L/2R pivots, 20-bar decay | ~2 bars per session |
| 1-hour | [trend_compass_1h.pine](trend_compass/trend_compass_1h.pine) | [docs](docs/trend_compass_1h.md) | 4H 50, Daily 50, Daily 200, Weekly 50 | 3L/2R pivots, 30-bar decay | ~6.5 bars per session |

Each variant is calibrated for its timeframe: compression thresholds, EMA slope sensitivity, crossover windows, and divergence pivot parameters all scale appropriately. The 1-hour variant provides the fastest divergence feedback and intraday trend assessment. The Daily variant provides the broadest structural picture with the least noise.

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
