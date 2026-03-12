# Trend Compass 4H

4-hour optimized strategic trend assessment overlay for equities, ETFs, and indices.

---

## Design Philosophy

The 4-hour variant bridges the gap between the intraday Market Monitors and the Daily Trend Compass. It answers: **What is the multi-day trend? Is it gaining or losing momentum? Are divergences forming?** — with faster update cycles than the Daily variant.

The 4H chart produces ~2 bars per RTH session (~6 with extended hours), making it granular enough to catch multi-day trend shifts within the same trading day, while slow enough to filter out intraday noise. It serves active swing traders who need trend context refreshed more frequently than once per day.

**Key architectural difference from the Daily variant**: The 4H chart has no local 200 EMA anchor (200 bars on 4H = ~33 days, which doesn't carry the same institutional weight as the daily 200 EMA). Instead, the Daily 200 EMA is pulled via `request.security()` as the structural anchor, alongside the Daily 50 EMA as an intermediate reference and the Weekly 50 EMA for macro context. This gives the 4H chart access to institutional-grade structural levels without computing them on the wrong timeframe.

---

## Key Differences from Daily Variant

| Feature | Daily Variant | 4H Variant |
|---------|--------------|------------|
| EMA Ribbon | 10/21/50 (local) | 10/21/50 (local, on 4H bars) |
| Anchor EMA | 200 (local daily) | Daily 200 EMA (via `request.security`) |
| HTF #1 | Weekly 50 EMA | Daily 50 EMA |
| HTF #2 | (Monthly optional) | Daily 200 EMA |
| HTF #3 | — | Weekly 50 EMA |
| Key Levels | Prior Week + Prior Month H/L/C | Prior Day + Prior Week H/L/C |
| 52-Week H/L | Local `ta.highest(252)` | Via `request.security("D")` for accuracy |
| Divergence Pivots | 5L/3R default | 4L/2R default (faster detection) |
| Divergence Decay | 15 bars (~15 days) | 20 bars (~3.5 days) |
| ATR/BB Percentile LB | 100 bars (~5 months) | 200 bars (~33 days) |
| EMA Compressed Threshold | 0.5% spread | 0.3% spread |
| EMA Slope Threshold | 0.01 | 0.005 |
| Cross Detection Window | Fast/Mid: 3 bars, Mid/Slow: 5 bars | Fast/Mid: 4 bars, Mid/Slow: 6 bars |
| `request.security` calls | 3 | 6 |
| Dashboard Rows | 17 | 18 |

---

## Table of Contents

- [Features](#features)
- [Role in the Indicator Suite](#role-in-the-indicator-suite)
- [Installation](#installation)
- [Indicator Components](#indicator-components)
  - [EMA Ribbon (10/21/50)](#ema-ribbon-102150)
  - [HTF EMAs (Daily 200, Daily 50, Weekly 50)](#htf-emas-daily-200-daily-50-weekly-50)
  - [Trend Phase Classifier](#trend-phase-classifier)
  - [Composite Trend Score](#composite-trend-score)
  - [ADX / DMI + Slope Analysis](#adx--dmi--slope-analysis)
  - [RSI + Divergence Detection](#rsi--divergence-detection)
  - [MACD + Divergence Detection](#macd--divergence-detection)
  - [OBV Trend Confirmation](#obv-trend-confirmation)
  - [Relative Volume](#relative-volume)
  - [ATR Regime Classification](#atr-regime-classification)
  - [Bollinger Band Width Percentile](#bollinger-band-width-percentile)
  - [Key Structural Levels](#key-structural-levels)
  - [Fibonacci Retracement (optional)](#fibonacci-retracement-optional)
  - [Ichimoku Cloud (optional)](#ichimoku-cloud-optional)
  - [52-Week High / Low](#52-week-high--low)
  - [Dashboard](#dashboard)
  - [Alerts](#alerts)
- [Visual Reference](#visual-reference)
  - [Color Map](#color-map)
  - [Chart Element Legend](#chart-element-legend)
  - [Dashboard Fields](#dashboard-fields)
- [Tuning Guide](#tuning-guide)
- [Architecture Notes](#architecture-notes)
- [Known Limitations](#known-limitations)
- [Changelog](#changelog)

---

## Features

- **EMA Ribbon** (10/21/50) with dynamic cloud fill, computed on 4H bars for multi-day swing context.
- **Daily 200 EMA** pulled via `request.security()` — the institutional anchor plotted as a thick orange line.
- **Daily 50 EMA** pulled via `request.security()` — intermediate structural trend plotted as a cyan step line.
- **Weekly 50 EMA** pulled via `request.security()` — macro trend reference plotted as a purple step line.
- **Trend Phase Classifier** — six states: EMERGING, ACCELERATING, MATURE, EXHAUSTING, CONSOLIDATING, REVERSING.
- **Composite Weighted Trend Score** (-11 to +11) from 8 conditions across structural and confirmation tiers.
- **Score Trend Tracking** (IMPROVING / STABLE / FADING) via 5-bar SMA comparison.
- **ADX Slope Analysis** — RISING / FLAT / FALLING trend strength momentum.
- **50 EMA Slope + Acceleration** — quantifies trend angle and curvature on the 4H timeframe.
- **RSI Divergence Detection** — bullish, bearish, hidden bull, hidden bear with visual line segments. Pivot detection tuned for 4H speed (4L/2R).
- **MACD Divergence Detection** — same four-state detection on the MACD histogram with 4H-tuned pivots.
- **OBV Trend Confirmation** — CONFIRMING / DIVERGING / NEUTRAL.
- **Relative Volume** (20-bar SMA baseline) with SPIKE / ABOVE / NORMAL / DRY classification.
- **ATR Percentile Ranking** — LOW / NORMAL / HIGH / EXTREME regime classification with 200-bar lookback.
- **Bollinger Band Width Percentile** — compression detection with 200-bar lookback.
- **52-Week High / Low** pulled from Daily timeframe for accuracy, with position-in-range percentage.
- **Prior Day High / Low / Close** reference levels (blue dashed).
- **Prior Week High / Low / Close** reference levels (purple dashed).
- **Fibonacci Retracement** (optional, default OFF) — 38.2%, 50%, 61.8% levels over 100-bar lookback (~17 trading days on 4H).
- **Ichimoku Cloud** (optional, default OFF) — standard 9/26/52/26 parameters.
- **18-row heads-up dashboard** with color-coded status indicators.
- **12 alert conditions** covering phase changes, score thresholds, divergences, golden/death crosses, volatility shifts, and compression breakout watches.

---

## Role in the Indicator Suite

| Indicator | Primary TF | Focus | Update Cadence |
|-----------|-----------|-------|----------------|
| 0DTE Scalper 1m | 1-min | Micro-entry timing | Every minute |
| 0DTE Scalper 5m | 5-min | Swing-scalp trades | Every 5 min |
| 0DTE Scalper 15m | 15-min | Session directional bias | Every 15 min |
| Market Monitor 1m | Any intraday | Multi-chart watchlist bias | Real-time |
| Market Monitor 5m | 5-min | Enhanced watchlist bias | Every 5 min |
| **Trend Compass 4H** | **4-Hour** | **Multi-day swing trend** | **~2x per session** |
| Trend Compass Daily | Daily | Strategic position trend | Once per day |

**Workflow**: Trend Compass Daily sets the macro bias. Trend Compass 4H provides faster feedback on whether that macro bias is strengthening or weakening within the current multi-day swing. Market Monitors scan for intraday alignment. 0DTE Scalpers execute.

---

## Installation

1. Open TradingView and navigate to any **4-Hour chart** (SPY, NVDA, AAPL, etc.).
2. Open the **Pine Editor** (bottom panel).
3. Delete any existing code in the editor.
4. Paste the entire contents of `trend_compass_4h_v1_0.pine`.
5. Click **Add to chart**.

### Data Requirements

- **Daily and Weekly data access** required for HTF EMA pulls and 52-week calculations. Standard on all TradingView plans.
- **Sufficient 4H bar history** for percentile calculations (200+ bars default).

### Chart Settings

No session-based logic. Chart timezone settings do not affect calculations. Apply to any chart timezone.

---

## Indicator Components

### EMA Ribbon (10/21/50)

Computed locally on 4H bars. On a 4H chart with ~6 bars per day (extended hours):

- **10 EMA**: ~1.7 trading days. Captures very short-term momentum shifts within a multi-day swing.
- **21 EMA**: ~3.5 trading days. The intermediate trend on the 4H timeframe.
- **50 EMA**: ~8.3 trading days. The primary structural trend line for multi-day swings.

The **EMA Stack** assessment incorporates the Daily 200 EMA as the anchor:

| State | Condition | Interpretation |
|-------|-----------|---------------|
| Full Bullish Stack | 4H Fast > Mid > Slow, all above D200 EMA | All timeframes aligned bullish |
| Partial Bull | 4H ribbon bullish, but below D200 | Developing uptrend, not yet structural |
| Full Bearish Stack | 4H Fast < Mid < Slow, all below D200 EMA | All timeframes aligned bearish |
| Partial Bear | 4H ribbon bearish, but above D200 | Developing downtrend or pullback |
| Compressed | EMA spread < 0.3% | Trend transition (tighter threshold than Daily due to 4H bar scale) |
| Mixed | Any other | No clear alignment |

### HTF EMAs (Daily 200, Daily 50, Weekly 50)

Three institutional-grade reference lines pulled from higher timeframes:

| Line | Color | Style | Purpose |
|------|-------|-------|---------|
| Daily 200 EMA | Orange (#FF6D00) | Solid, thick | The institutional dividing line. Primary structural anchor. |
| Daily 50 EMA | Cyan (#26C6DA) | Step line, thick | Intermediate structure. Swing traders key off this. |
| Weekly 50 EMA | Purple (#E040FB) | Step line, thick | Macro trend. ~1 year smoothed. |

All three are individually toggleable. The Daily 200 EMA is used as the anchor in both the EMA Stack assessment and the composite trend score.

### Trend Phase Classifier

Identical six-state system to the Daily variant with slightly adjusted crossover detection windows (4/6 bars instead of 3/5) to account for the higher bar frequency:

| Phase | Conditions | Interpretation |
|-------|-----------|----------------|
| **EMERGING** | ADX < 25, ADX rising, EMAs separating | New trend forming |
| **ACCELERATING** | ADX > 20, ADX rising | Trend gaining momentum |
| **MATURE** | ADX > 30, ADX flat | Trend intact, momentum plateauing |
| **EXHAUSTING** | ADX declining from > 30 | Trend losing steam |
| **CONSOLIDATING** | ADX < 20, EMAs compressed | Range-bound |
| **REVERSING** | Recent EMA crossovers detected | Trend change underway |

### Composite Trend Score

Same weighted architecture as the Daily variant and the broader indicator suite.

**Structural Conditions (2x weight):**

| # | Condition | Bullish (+2) | Bearish (-2) |
|---|-----------|-------------|-------------|
| 1 | 4H EMA Stack | Fast > Mid > Slow | Fast < Mid < Slow |
| 2 | Price vs Daily 200 EMA | Above | Below |
| 3 | Price vs Weekly 50 EMA | Above | Below |

**Confirmation Conditions (1x weight):**

| # | Condition | Bullish (+1) | Bearish (-1) |
|---|-----------|-------------|-------------|
| 4 | ADX + DI Direction | ADX > 20 AND DI+ > DI- | ADX > 20 AND DI- > DI+ |
| 5 | RSI Position | RSI > 55 | RSI < 45 |
| 6 | MACD vs Signal | Above signal | Below signal |
| 7 | OBV Slope | Positive | Negative |
| 8 | 4H 50 EMA Slope | Positive | Negative |

**Weighted max: +/-11.** Score interpretation and IMPROVING / STABLE / FADING tracking identical to Daily variant.

### Divergence Detection (RSI + MACD)

Same four-state system as Daily but with faster pivot parameters:

- **Default pivots**: 4 left / 2 right (vs 5/3 on Daily). Detects divergences ~1 bar sooner.
- **Decay window**: 20 bars (~3.5 days on 4H, vs 15 bars = 15 days on Daily). Divergences expire faster because the 4H timeframe produces more bars.
- **Visual**: RSI divergences = dashed lines (green/red). MACD divergences = dotted lines (cyan/purple). Same as Daily.

### Key Structural Levels

**Prior Day H/L/C** — blue dashed/dotted lines. The most immediately relevant reference for 4H swing entries. Prior day high is the first resistance target; prior day low is the first support target.

**Prior Week H/L/C** — purple dashed/dotted lines. Weekly structural pivots. More significant than daily levels for multi-day swing context.

**52-Week High / Low** — pulled from the Daily timeframe via `request.security()` for accuracy. Avoids the bar-count estimation issues that would arise from computing locally on 4H bars.

### All Other Components

ATR regime, BB Width percentile, OBV, relative volume, Fibonacci, and Ichimoku are functionally identical to the Daily variant. The only calibration differences are the percentile lookbacks (200 bars on 4H ≈ 33 trading days, covering similar calendar time to the Daily's 100-bar default).

---

## Dashboard

18-row compact table:

| Row | Field | Content |
|-----|-------|---------|
| 0 | Header | "Trend Compass - 4h" (merged, purple accent bar) |
| 1 | Phase | EMERGING / ACCELERATING / MATURE / EXHAUSTING / CONSOLIDATING / REVERSING |
| 2 | Score | Composite score with label |
| 3 | Momentum | IMPROVING / STABLE / FADING |
| 4 | EMA Stack | FULL BULL / PARTIAL BULL / COMPRESSED / PARTIAL BEAR / FULL BEAR / MIXED |
| 5 | ADX | Value + tier label |
| 6 | ADX Slope | RISING / FLAT / FALLING + slope value |
| 7 | RSI | Value + status |
| 8 | MACD | Histogram direction + signal position |
| 9 | RSI Div | BULL DIV / BEAR DIV / HIDDEN BULL / HIDDEN BEAR / NONE |
| 10 | MACD Div | Same classification |
| 11 | OBV | CONFIRMING / DIVERGING / NEUTRAL |
| 12 | Rel Vol | Multiplier + classification |
| 13 | ATR | Value + regime + percentile |
| 14 | BB Width | Percentile + classification |
| 15 | D 200 EMA | ABOVE/BELOW + distance percentage |
| 16 | W 50 EMA | Weekly trend alignment |
| 17 | 52w Pos | Position percentage + high/low values |

---

## Alerts

12 alert conditions (identical to Daily variant):

| Alert | Trigger |
|-------|---------|
| Trend Phase Changed | Any phase transition |
| Strong Bullish Trend | Score >= +7 (weighted) or +5 (equal) |
| Strong Bearish Trend | Score <= -7 or -5 |
| Score Crossed Neutral | Score crosses zero |
| RSI Bullish Divergence | New RSI bull div detected |
| RSI Bearish Divergence | New RSI bear div detected |
| MACD Bullish Divergence | New MACD bull div detected |
| MACD Bearish Divergence | New MACD bear div detected |
| Golden Cross (D50/D200) | Daily 50 EMA crosses above Daily 200 EMA |
| Death Cross (D50/D200) | Daily 50 EMA crosses below Daily 200 EMA |
| ATR Regime Change | Volatility regime shifts |
| BB Extreme Compression | BB width < 5th percentile |

Note: Golden/Death Cross alerts fire based on the Daily 50 and Daily 200 EMA values from `request.security()`, not local 4H EMAs. This is the institutional-grade crossover that matters.

---

## Visual Reference

### Color Map

| Element | Color | Hex |
|---------|-------|-----|
| Bullish | Green | #00E676 |
| Bearish | Red | #FF1744 |
| Neutral | Blue-gray | #B0BEC5 |
| Warning | Yellow | #FFD600 |
| Alert | Orange | #FF9100 |
| Daily 200 EMA | Deep orange | #FF6D00 |
| Daily 50 EMA | Cyan | #26C6DA |
| Weekly 50 EMA | Purple | #E040FB |
| Prior Day levels | Blue | #42A5F5 |
| Prior Week levels | Purple | #AB47BC |
| 52-Week levels | Salmon | #FF8A65 |
| Fib levels | Teal / Amber / Orange | Various |
| Ichimoku | Blue / Orange | Various |
| RSI div lines | Green (bull) / Red (bear) | Dashed |
| MACD div lines | Cyan (bull) / Purple (bear) | Dotted |

### Chart Element Legend

| Element | Style | When Visible |
|---------|-------|-------------|
| 4H Fast EMA (10) | Thin green line | Always (toggleable) |
| 4H Mid EMA (21) | Thin yellow line | Always (toggleable) |
| 4H Slow EMA (50) | Thin red line | Always (toggleable) |
| EMA Cloud | Semi-transparent fill | When Show Cloud enabled |
| Daily 200 EMA | Thick orange solid | When Show D200 enabled |
| Daily 50 EMA | Thick cyan step line | When Show D50 enabled |
| Weekly 50 EMA | Thick purple step line | When HTF + Show W50 enabled |
| Prior Day H/L | Blue dashed | When enabled |
| Prior Day C | Blue dotted | When enabled |
| Prior Week H/L | Purple dashed | When enabled |
| Prior Week C | Purple dotted | When enabled |
| 52-Week H/L | Salmon solid | When enabled |
| Fib levels | Colored dashed | When Fib enabled |
| Ichimoku Cloud | Filled cloud + lines | When Ichimoku enabled |
| RSI divergence | Dashed line on price | When detected |
| MACD divergence | Dotted line on price | When detected |

---

## Tuning Guide

### EMA Ribbon

The 10/21/50 defaults work well on 4H for multi-day swing context. If you want slower, more stable trends:

- **13/34/89**: Fibonacci-based. Much smoother. Fewer transitions.
- **20/50/100**: Very conservative. Catches only major trend shifts.

### Divergence Pivot Bars

Default 4L/2R provides a good balance between speed and significance on 4H. The 2-bar right delay means divergences confirm ~8 hours after the actual pivot.

- **3L/1R**: Fastest possible. More noise, but earliest detection.
- **5L/3R**: Matches the Daily variant. Fewer, more significant divergences.

### ATR / BB Width Percentile Lookback

Default 200 bars (~33 trading days). This covers roughly 6-7 weeks of volatility history on 4H. Alternatives:

- **100 bars** (~17 days): More responsive. Regime shifts detected sooner.
- **400 bars** (~67 days): Longer context spanning ~3 months.

### Fibonacci Lookback

Default 100 bars (~17 trading days). This captures the most recent 2-3 week swing range.

- **50 bars** (~8 days): Short-term swing. More responsive fibs.
- **200 bars** (~33 days): Broader structure. More stable levels.

### Ichimoku Parameters

Standard 9/26/52/26. On 4H bars, these translate to approximately:

- Tenkan (9): ~1.5 trading days
- Kijun (26): ~4.3 trading days
- Senkou B (52): ~8.7 trading days
- Displacement (26): ~4.3 trading days forward/back

---

## Architecture Notes

- **`request.security()` budget**: 6 calls total: Daily 200 EMA, Daily 50 EMA, Weekly 50 EMA, prior day H/L/C (tuple), prior week H/L/C (tuple), 52-week high/low (tuple). Well within the 40-call limit.
- **Loops**: Two loops (ATR percentile, BB width percentile) over 200-bar default lookback. Both lightweight, run once per bar.
- **Object counts**: Lines for key levels and divergences, labels for Fibonacci, one table for dashboard. Well within limits.
- **Dashboard**: Only rendered on `barstate.islast`.
- **Golden/Death Cross detection**: Performed on the Daily HTF EMA values, not local 4H EMAs. This correctly identifies the institutional-grade crossover by comparing the current Daily 50 EMA value to its previous value relative to the Daily 200 EMA.
- **52-Week H/L accuracy**: Pulled from Daily timeframe (`ta.highest(high, 252)` on daily bars) rather than estimated locally on 4H bars. This avoids bar-count discrepancies from extended hours, partial sessions, and holidays.

---

## Known Limitations

- **4H-optimized**: While it will run on other timeframes, all parameters and HTF mappings are calibrated for 4-hour bars. For daily charts, use the Daily variant.
- **6 `request.security` calls**: More than the Daily variant (3). Still well within TradingView's limit, but reduces headroom if combining with other indicators that also use `request.security()`.
- **Golden/Death Cross detection on HTF data**: The crossover is detected by comparing current and previous values of the Daily EMAs as seen through `request.security()`. On some 4H bars within the same daily candle, both values may be identical (no new daily bar has formed), which can occasionally delay detection by one 4H bar.
- **52-Week H/L via `request.security`**: Requires the Daily timeframe to have 252 bars of history. This is standard for all liquid US equities and ETFs, but may be limited for very recently listed instruments.
- **OBV on 4H bars**: 4H volume bars aggregate intraday volume differently depending on whether extended hours are enabled. OBV trend direction is generally reliable, but absolute OBV values may differ from daily OBV calculations.
- **Divergence detection inherent lag**: 2-bar right pivot = ~8 hours of confirmation delay. This is faster than the Daily variant but still not instantaneous.
