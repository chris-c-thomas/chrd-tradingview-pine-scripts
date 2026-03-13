# 0DTE SPY Scalper - 15m

**Platform**: TradingView
**Language**: Pine Script v6
**Chart**: SPY 15-Minute (optimized)
**Theme**: Dark theme (vibrant, high-contrast palette)

---

## Overview

A single-overlay indicator purpose-built for establishing directional bias and identifying high-conviction swing entries for 0DTE (zero days to expiration) SPY options on the 15-minute chart. It combines trend structure, momentum, volatility, volatility compression detection (TTM Squeeze), market breadth (NYSE TICK with SMA smoothing), key price levels, a multi-timeframe confirmation layer (5-minute), an anchor EMA for structural context, and RSI divergence detection into a unified decision-support system with **weighted confluence scoring** and a 26-row heads-up dashboard.

v1.1 introduces a fundamentally upgraded signal engine with tiered scoring weights, RSI divergence as a new analytical capability, ATR regime classification, score trend tracking, prior day VWAP close as a key institutional level, and Bollinger Band width percentile as a continuous volatility metric.

This is the 15-minute companion to the 1-minute and 5-minute SPY 0DTE Scalpers. It targets the highest-conviction directional swings with 15-60 minute hold times. The 15-minute chart produces ~26 RTH bars per session, so the signal engine is calibrated for maximum conviction per signal with the lowest total signal count across all three variants.

**Role in the multi-chart hierarchy**:

- **1-minute**: Micro-entry timing, rapid scalps (1-5 min holds)
- **5-minute**: Swing-scalp trades, moderate conviction (5-25 min holds)
- **15-minute** (this): Directional bias, structural trend context, high-conviction swings (15-60 min holds)

The 15-minute chart is the "big picture" view. Its primary value is establishing whether the session trend is bullish, bearish, ranging, or in transition. Signals on this timeframe represent major structural inflection points. When the 15-minute and 5-minute agree on direction, the 1-minute can be used for precision entry timing.

The indicator does not auto-trade. It surfaces high-confluence setups as CALLS or PUTS labels on the chart, backed by a weighted scoring engine that evaluates up to twelve independent conditions per bar. All thresholds are configurable.

---

## Table of Contents

- [What's New in v1.1](#whats-new-in-v11)
- [Features](#features)
- [Key Differences from 1-Minute and 5-Minute Variants](#key-differences-from-1-minute-and-5-minute-variants)
- [Installation](#installation)
- [Indicator Components](#indicator-components)
  - [EMA Ribbon + Anchor EMA](#ema-ribbon--anchor-ema)
  - [VWAP and Bands](#vwap-and-bands)
  - [RSI](#rsi)
  - [ADX / DMI](#adx--dmi)
  - [ATR + ATR Regime Classification](#atr--atr-regime-classification)
  - [TTM Squeeze + BB Width Percentile](#ttm-squeeze--bb-width-percentile)
  - [RSI Divergence Detection](#rsi-divergence-detection)
  - [NYSE TICK Index (Smoothed)](#nyse-tick-index-smoothed)
  - [Volume Footprint](#volume-footprint)
  - [Multi-Timeframe Confirmation (5-Min)](#multi-timeframe-confirmation-5-min)
  - [Key Price Levels](#key-price-levels)
  - [Regime / Mode Detection](#regime--mode-detection)
  - [Signal Engine (Weighted Scoring)](#signal-engine-weighted-scoring)
  - [Score Trend Indicator](#score-trend-indicator)
  - [Directional Bias Background](#directional-bias-background)
  - [Dashboard](#dashboard)
  - [Alerts](#alerts)
- [Visual Reference](#visual-reference)
  - [Color Map](#color-map)
  - [Chart Element Legend](#chart-element-legend)
  - [Dashboard Fields](#dashboard-fields)
- [Tuning Guide](#tuning-guide)
  - [Minimum Signal Score (Weighted vs Equal)](#minimum-signal-score-weighted-vs-equal)
  - [EMA Lengths](#ema-lengths)
  - [Anchor EMA Length](#anchor-ema-length)
  - [ADX Thresholds](#adx-thresholds)
  - [VWAP Extended Distance](#vwap-extended-distance)
  - [Signal Cooldown](#signal-cooldown)
  - [Signal Time Window](#signal-time-window)
  - [Opening Range Duration](#opening-range-duration)
  - [RSI Thresholds](#rsi-thresholds)
  - [RSI Divergence Settings](#rsi-divergence-settings)
  - [TTM Squeeze Settings](#ttm-squeeze-settings)
  - [NYSE TICK Settings](#nyse-tick-settings)
  - [Multi-Timeframe Toggle](#multi-timeframe-toggle)
  - [Directional Bias Background](#directional-bias-background-1)
  - [Tuning Profiles](#tuning-profiles)
- [Architecture Notes](#architecture-notes)
- [Known Limitations](#known-limitations)
- [Changelog](#changelog)

---

## What's New in v1.1

### 1. Weighted Scoring Engine

The most significant change. Conditions are now split into two tiers:

| Tier | Weight | Conditions | Rationale |
|------|--------|-----------|-----------|
| **Structural** | 2x | VWAP position, EMA alignment, Anchor EMA | These define the directional environment. If all three agree, the structural thesis is strong. |
| **Confirmation** | 1x | RSI neutral, candle pattern, ADX, MTF trend, MTF MACD, level proximity, squeeze, TICK, divergence | These provide timing and additional conviction. |

**Weighted max score**: 3 structural (x2) + 9 confirmation (x1) = **15**. Default min score: **7** (equivalent to all structural conditions aligning, which itself yields 6 points).

A toggle (`Use Weighted Scoring`) allows reverting to equal-weight mode (max 12). The dashboard header shows `[W]` or `[EQ]` to indicate the active mode. Alerts include the scoring mode.

### 2. RSI Divergence Detection

Detects classic bullish and bearish RSI divergence over a configurable lookback window (default 5 bars = 1.25 hours on 15-min). On the 15-min timeframe, divergence represents structurally meaningful price/momentum disagreement that frequently precedes real reversals.

- **Bullish divergence**: price near its lookback low, RSI above its lookback low (momentum not confirming the price weakness)
- **Bearish divergence**: price near its lookback high, RSI below its lookback high (momentum not confirming the price strength)

Integrated as a new confirmation scoring condition (1x weight). Dashboard row shows BULL DIV / BEAR DIV / NONE. A dedicated alert fires on detection.

### 3. ATR Regime Classification

Current ATR compared to its 20-bar SMA provides context for whether volatility is elevated, normal, or compressed relative to recent history.

| Ratio | Classification | Color |
|-------|---------------|-------|
| > 1.3x | HIGH | Orange |
| 0.7-1.3x | NORMAL | Gray |
| < 0.7x | LOW | Cyan |

Displayed in two dashboard rows (ATR with regime label, and a dedicated Vol Regime row). Not directly scored — this is pure situational awareness that complements the TTM Squeeze binary state.

### 4. Score Trend Indicator

Tracks the best directional score (max of calls/puts) over the last 3 bars and classifies the trajectory: BUILDING (2 consecutive increases), RISING (1 increase), STEADY (flat), FALLING (1 decrease), FADING (2 consecutive decreases).

Dashboard shows the trend label and the 3-bar score history (e.g., "3 > 5 > 7"). On a 15-min chart where setups develop over 30-45 minutes, knowing that conditions are building toward a signal is actionable context.

### 5. Prior Day VWAP Close

The prior session's closing VWAP value is plotted as a new key level (dotted cyan line with "PD VWAP" label). This is a common institutional reference point representing prior-session fair value.

Added to the level proximity detection pool — price near the prior VWAP close now contributes to the level proximity scoring condition. Dashboard row shows distance from current price and ABOVE/BELOW status.

### 6. Bollinger Band Width Percentile

A continuous measure of BB width relative to its own 20-bar SMA, complementing the binary TTM Squeeze state with a nuanced volatility expansion/compression reading.

| Ratio | Classification | Color |
|-------|---------------|-------|
| > 1.5x | EXPANDING | Orange |
| 0.7-1.5x | NORMAL | Gray |
| < 0.7x | COMPRESSING | Cyan |

Dashboard row shows the ratio and classification. Not directly scored.

---

## Features

- **EMA Ribbon** (10/21/34) with dynamic cloud fill that shifts color based on trend alignment (bullish, bearish, mixed). Wider EMA windows tuned for 15-min bar dynamics, capturing 2.5-8.5 hour swings.
- **Anchor EMA** (50-period, default) plotted as a thicker structural reference line. 50 bars on a 15-min chart = ~12.5 hours (~2 full sessions). **Weighted as a structural scoring condition (2x)** in v1.1.
- **Session-anchored VWAP** with configurable standard deviation bands.
- **RSI, ADX, and ATR** calculated internally and displayed in the dashboard (not plotted as separate panes, preserving chart real estate).
- **ATR Regime Classification** comparing current ATR to its 20-bar SMA. Three states: HIGH (>1.3x), NORMAL (0.7-1.3x), LOW (<0.7x). Displayed in dashboard with ratio.
- **TTM Squeeze detection** identifying Bollinger Band compression inside Keltner Channels. Three states: SQUEEZE ON, FIRED, OFF. Integrated as confirmation scoring condition (1x weight).
- **Bollinger Band Width Percentile** providing a continuous volatility metric relative to recent history. Three states: EXPANDING (>1.5x), NORMAL, COMPRESSING (<0.7x). Complements the binary squeeze state.
- **RSI Divergence Detection** over configurable lookback (default 5 bars = 1.25 hrs). Classic bullish/bearish divergence scored as confirmation condition (1x). Dashboard and alert integration.
- **NYSE TICK Index with SMA smoothing** pulled via `request.security()`. Raw and 5-period SMA-smoothed TICK pulled; smoothed value used for scoring with calibrated lower thresholds (300/-300/600).
- **Volume Footprint** (v1.2) integration via `request.footprint()` for deep intrabar order flow analysis. Displays buy/sell volume, delta with strength classification, cumulative session delta, and POC/VAH/VAL as step-line levels. Delta integrated as 1x confirmation scoring condition (ON by default). Prior bar's POC enriches the support/resistance proximity pool. Requires TradingView Premium or Ultimate plan; degrades gracefully to N/A on lower plans.
- **Multi-timeframe confirmation layer (5-minute)** pulling 5-min RSI, EMA trend, and MACD histogram via consolidated `request.security()` tuple.
- **Pre-Market High/Low** automatically detected and drawn as horizontal levels once RTH begins.
- **Prior Day High/Low/Close** pulled from the daily timeframe and plotted as dotted reference levels.
- **Prior Day VWAP Close** (new in v1.1): institutional fair value reference from prior session. Plotted as a dotted cyan line. Included in level proximity pool.
- **Opening Range** (default 30 minutes / 2 bars) captured at RTH open and drawn as dashed levels.
- **Session HOD/LOD** tracked in real time with dynamically updating lines.
- **Session progress indicator** showing current bar count out of ~26 total RTH bars.
- **Regime Classifier** categorizing current market state into BULLISH, BEARISH, RANGING, NO TRADE, or TRANSITION.
- **Weighted confluence scoring engine** (v1.1, expanded in v1.2): 3 structural conditions (2x weight) + 10 confirmation conditions (1x weight, including footprint delta). Weighted max: 16. Equal-weight max: 13. Toggle between modes.
- **Score Trend Indicator** (new in v1.1): 3-bar score trajectory tracking. BUILDING / RISING / STEADY / FALLING / FADING with history display.
- **Expanded candle pattern detection** optimized for 15-min bars: engulfing, strong close (60% body), hammer/shooting star (1.6x tail), inside bar breakout, 3-bar momentum, morning star/evening star.
- **Directional bias background** (optional, off by default): subtle full-bar tint when regime is directional.
- **VWAP cross markers** (diamond shapes) for quick visual identification.
- **Real-time dashboard** (31-row table overlay) with all indicator data, scoring mode indicator, score trend, ATR regime, BB width, RSI divergence, prior VWAP close, and footprint order flow rows.
- **Dual alert system**: static `alertcondition()` entries (11 total, including RSI divergence and footprint delta/cumulative delta crossovers) plus dynamic `alert()` calls with interpolated context including scoring mode, ATR regime, divergence, and footprint status.
- **Bar confirmation gate** and **signal cooldown** (default 2 bars = 30 minutes).

---

## Key Differences from 1-Minute and 5-Minute Variants

| Aspect | 1-Minute | 5-Minute | 15-Minute v1.1 | Why (15-min) |
|--------|----------|----------|----------------|---------------|
| EMA Lengths | 8/13/21 | 9/15/21 | 10/21/34 | Wider windows for 2.5-8.5 hr swings |
| Anchor EMA | N/A | N/A | 50-period (2x structural weight) | Multi-session structure; weighted as structural condition |
| RSI Length | 14 | 10 | 9 | 15-min bars are smooth; RSI(9) responsive to intraday swings |
| RSI OB/OS | 70/30 | 65/35 | 60/40 | 15-min RSI rarely reaches 70/30 intraday |
| RSI Divergence | N/A | N/A | 5-bar lookback (1x confirmation) | NEW in v1.1: 5 bars = 1.25 hrs, structurally meaningful |
| ADX No-Trend | 15 | 18 | 20 | ADX reads higher on smooth bars |
| Scoring Model | AND-gate | 5+4 equal (max 9) | 3 structural (2x) + 9 confirm (1x), max 15 | Weighted scoring differentiates structural vs timing conditions |
| Scoring Toggle | N/A | N/A | Weighted / Equal mode switch | Equal mode (max 12) available for traders who prefer flat weighting |
| Score Trend | N/A | N/A | 3-bar trajectory tracking | BUILDING/FADING awareness for slow-forming setups |
| ATR Regime | N/A | N/A | HIGH/NORMAL/LOW vs 20-bar SMA | Contextualizes raw ATR value |
| BB Width | N/A | N/A | Ratio vs 20-bar SMA | Continuous volatility metric complementing binary squeeze |
| Prior Day VWAP | N/A | N/A | Plotted + in level proximity pool | Institutional fair value reference |
| Opening Range | 5 min (5 bars) | 15 min (3 bars) | 30 min (2 bars) | Institutional standard initial balance |
| Signal Cooldown | 5 bars (5 min) | 3 bars (15 min) | 2 bars (30 min) | ~26 bars/session; minimal cooldown maximizes opportunities |
| Signal Window Start | 09:35 | 09:45 | 10:00 | After 30-min OR completes + 1 buffer bar |
| VWAP Ext Filter | 1.5 ATR | 2.0 ATR | 2.5 ATR | Larger ATR values on 15-min |
| Candle Patterns | 4 patterns | 6 patterns | 8 patterns | Morning star/evening star meaningful on 15-min |
| Strong Close | 50% | 55% | 60% | 15-min candles have proportionally larger wicks |
| Hammer/Star | 2.0x body | 1.8x body | 1.6x body | 15-min patterns show less extreme tail ratios |
| MTF Source | N/A | 1-min (RSI + EMA) | 5-min (RSI + EMA + MACD) | 5-min is the natural lower-TF for 15-min |
| TICK Handling | Raw 1-min | Raw 1-min | Raw + SMA(5) smoothed | Noise reduction for 15-min timeframe |
| TICK Thresholds | 500/-500/800 | 500/-500/800 | 300/-300/600 | Calibrated for smoothed signal |
| Squeeze Recency | 5 bars (~5 min) | 3 bars (~15 min) | 2 bars (~30 min) | Appropriate for 15-min structural plays |
| Level Proximity | N/A | 1.0 ATR | 1.5 ATR (includes PD VWAP) | Widened radius; prior VWAP added to level pool |
| Bias Background | N/A | N/A | Optional (off default) | "Set and glance" directional context |
| Session Progress | N/A | N/A | Bar count + Early/Late labels | Phase awareness with ~26 bars/session |
| Dashboard Rows | 16 | 18 | 26 | Added score trend, RSI div, ATR regime, BB width, PD VWAP, vol regime |
| Alert Count | 4 static | 4 static | 7 static + 5 dynamic | Added RSI divergence alert |
| Line Extend | 300 bars | 60 bars | 40 bars | ~1.5 sessions |
| Arrow Size | tiny | tiny | small | Better visibility on sparser chart |

---

## Installation

1. Open TradingView and navigate to a **SPY 15-minute chart**.
2. Open the **Pine Editor** (bottom panel).
3. Delete any existing code in the editor.
4. Paste the entire contents of `spy_0dte_scalper_15min_v1_1.pine` into the editor.
5. Click **"Add to chart"** (or "Save" then "Add to chart").
6. The indicator appears as an overlay with the dashboard in the top-right corner.

**Required chart settings**:

- **Extended hours**: Enable "Extended Trading Hours" in chart settings if you want Pre-Market High/Low levels to populate.
- **Timezone**: ensure your chart timezone matches the indicator's timezone input (default: `America/New_York`).

**Upgrading from v1.0**: Remove the v1.0 indicator from your chart before adding v1.1. The `i_minScore` default has changed from 5 to 7 to reflect weighted scoring. If you previously used a custom min score, you may need to adjust it (see the [scoring calibration table](#minimum-signal-score-weighted-vs-equal)).

---

## Indicator Components

### EMA Ribbon + Anchor EMA

Three exponential moving averages form the core trend structure with a dynamic cloud fill between the fast and slow EMAs. A fourth long-period "anchor" EMA provides multi-session structural context.

| Parameter | Default | Purpose |
|-----------|---------|---------|
| Fast EMA | 10 | Short-term direction (10 bars = 2.5 hrs) |
| Mid EMA | 21 | Intermediate trend (21 bars = 5.25 hrs) |
| Slow EMA | 34 | Structural trend (34 bars = 8.5 hrs = ~1.3 sessions) |
| Anchor EMA | 50 | Multi-session structure (50 bars = 12.5 hrs = ~2 sessions) |

**Cloud fill states**: green (bullish: fast > mid > slow), red (bearish: fast < mid < slow), yellow (mixed/transitioning). The anchor EMA is plotted as a purple line with `linewidth=2`.

**Scoring**: In v1.1, both EMA alignment (fast > slow) and anchor EMA position are **structural conditions weighted 2x** in weighted scoring mode.

### VWAP and Bands

Session-anchored VWAP with manually calculated standard deviation bands using cumulative variance.

| Parameter | Default | Purpose |
|-----------|---------|---------|
| Show VWAP | true | Master toggle |
| Show Bands | true | +/- 1 StdDev bands |
| Band Multiplier | 1.0 | StdDev scaling factor |

VWAP is the primary mean-reversion anchor. Price relationship to VWAP is a **structural scoring condition (2x weight)** in v1.1.

### RSI

| Parameter | Default | Purpose |
|-----------|---------|---------|
| Length | 9 | RSI lookback (9 bars = 2.25 hrs) |
| Overbought | 60 | Upper extreme threshold |
| Oversold | 40 | Lower extreme threshold |

RSI neutral zone is a confirmation condition (1x weight). Signals are suppressed when RSI is at extremes.

### ADX / DMI

| Parameter | Default | Purpose |
|-----------|---------|---------|
| ADX Length | 14 | ADX/DI lookback period |
| No-Trend | 20 | Below this = no meaningful trend |
| Trending | 25 | Above this = confirmed trend |
| Strong Trend | 40 | Above this = strong directional move |

ADX above no-trend is a confirmation condition (1x weight) and optionally a gate prerequisite.

### ATR + ATR Regime Classification

| Parameter | Default | Purpose |
|-----------|---------|---------|
| ATR Length | 14 | ATR lookback (14 bars = 3.5 hrs) |

ATR serves as the volatility normalizer for VWAP distance filtering, level proximity detection, and the extension guard.

**New in v1.1**: ATR is compared to its 20-bar SMA to produce a **regime classification**:

| Ratio (ATR / SMA20) | Label | Interpretation |
|---------------------|-------|---------------|
| > 1.3 | HIGH | Elevated volatility — trending or volatile session |
| 0.7 - 1.3 | NORMAL | Typical volatility range |
| < 0.7 | LOW | Compressed volatility — potential squeeze setup |

Displayed in two dashboard rows (Row 8: ATR with regime label, Row 24: dedicated Vol Regime). The ratio value is shown for precision. This is a contextual indicator — not directly scored — that helps traders calibrate expectations for price movement and stop distances.

### TTM Squeeze + BB Width Percentile

**TTM Squeeze** detects volatility compression by comparing Bollinger Band width to Keltner Channel width.

| Parameter | Default | Purpose |
|-----------|---------|---------|
| BB Length | 20 | Bollinger Band SMA period |
| BB Multiplier | 2.0 | Bollinger Band width |
| KC Length | 20 | Keltner Channel EMA period |
| KC Multiplier | 1.5 | Keltner Channel width (ATR-based) |

Three states: SQUEEZE ON (BB inside KC — compression building), FIRED (BB exits KC — breakout beginning), OFF (normal volatility). Squeeze fired within 2 bars is a confirmation scoring condition (1x weight).

**New in v1.1: BB Width Percentile** provides a continuous measure of BB width relative to its own 20-bar SMA. This complements the binary squeeze with a nuanced volatility reading:

| Ratio (BBWidth / SMA20) | Label | Interpretation |
|--------------------------|-------|---------------|
| > 1.5 | EXPANDING | Volatility is actively expanding — breakout may be underway |
| 0.7 - 1.5 | NORMAL | Typical bandwidth range |
| < 0.7 | COMPRESSING | Bandwidth narrowing — squeeze likely forming or active |

Displayed as dashboard Row 21. Not scored — complements the binary squeeze state with continuous context.

### RSI Divergence Detection

**New in v1.1.** Detects classic bullish and bearish RSI divergence over a configurable lookback window.

| Parameter | Default | Purpose |
|-----------|---------|---------|
| Enable RSI Divergence | true | Master toggle |
| Divergence Lookback | 5 bars | Window for comparing price/RSI extremes (5 bars = 1.25 hrs) |

**Detection logic**:

- **Bullish divergence**: Current low is within 0.5 ATR of the lookback period's lowest low, but current RSI is at least 3 points above the lookback period's lowest RSI, and RSI is below 50. This indicates price weakness is not confirmed by momentum — a potential reversal setup.
- **Bearish divergence**: Current high is within 0.5 ATR of the lookback period's highest high, but current RSI is at least 3 points below the lookback period's highest RSI, and RSI is above 50. Price strength is not confirmed by momentum.

**Why this matters on 15-min**: A 5-bar lookback on 15-min represents 1.25 hours of price action. Divergence at this timeframe is structurally meaningful and frequently precedes real reversals, unlike on 1-min (5 min) or 5-min (25 min) where divergence is mostly noise.

**Scoring**: Integrated as a confirmation condition (1x weight). Bullish divergence scores for CALLS; bearish for PUTS.

**Dashboard**: Row 19 shows BULL DIV / BEAR DIV / NONE with the lookback period.

**Alert**: A dedicated `alertcondition()` and dynamic `alert()` fire on divergence detection.

### NYSE TICK Index (Smoothed)

| Parameter | Default | Purpose |
|-----------|---------|---------|
| TICK Symbol | USI:TICK | NYSE TICK source |
| Bullish Threshold | 300 | Smoothed TICK above this = bullish breadth |
| Bearish Threshold | -300 | Smoothed TICK below this = bearish breadth |
| Extreme Threshold | 600 | Smoothed TICK beyond this = extreme breadth |

Both raw and SMA(5)-smoothed TICK are pulled from the 1-min timeframe. Smoothed value used for scoring; both shown in dashboard. Confirmation condition (1x weight).

### Multi-Timeframe Confirmation (5-Min)

| Parameter | Default | Purpose |
|-----------|---------|---------|
| Enable MTF | true | Master toggle for all 5-min pulls |
| 5-Min RSI Length | 10 | RSI on 5-min bars |
| 5-Min Fast EMA | 9 | Fast EMA on 5-min bars |
| 5-Min Slow EMA | 21 | Slow EMA on 5-min bars |

Pulls three categories of 5-minute data:

1. **5-Min EMA Trend**: confirmation condition (1x weight)
2. **5-Min RSI**: displayed in dashboard, not directly scored
3. **5-Min MACD Histogram**: MACD(12,26,9). Confirmation condition (1x weight)

### Key Price Levels

| Level | Source | Style | Color | Purpose |
|-------|--------|-------|-------|---------|
| Pre-Market High/Low | Session tracking | Dashed | Red/Green | PM range boundaries |
| Prior Day High/Low/Close | Daily `request.security()` | Dotted | Orange/Teal/Gray | Prior session reference |
| **Prior Day VWAP Close** | **Daily `request.security()`** | **Dotted, width 2** | **Cyan (#40C4FF)** | **Institutional fair value (NEW v1.1)** |
| Opening Range High/Low | First 2 bars (30 min) | Dashed | Purple shades | Session initial balance |
| Session HOD/LOD | Real-time tracking | Solid | Green/Red | Current session extremes |

Line extend distances are set to 40 bars. Labels are offset 6 bars to the right.

**Level proximity** is a confirmation condition (1x weight). "Near support" for CALLS and "near resistance" for PUTS, using 1.5 ATR proximity radius. **v1.1 adds PD VWAP to the proximity pool** — price near the prior day's closing VWAP contributes to the level proximity score, with directional logic (PD VWAP acts as support from above, resistance from below).

### Regime / Mode Detection

Five-state classifier based on ADX trend strength, EMA alignment, and VWAP position:

| Mode | Conditions | Dashboard Color | Advice |
|------|-----------|-----------------|--------|
| BULLISH | ADX >= no-trend, EMA bullish, close > VWAP | Green | ACTIVE |
| BEARISH | ADX >= no-trend, EMA bearish, close < VWAP | Red | ACTIVE |
| RANGING | ADX < no-trend, not near VWAP | Yellow | CAUTION |
| TRANSITION | ADX >= trending, EMAs not aligned | Orange | ACTIVE |
| NO TRADE | ADX < no-trend, close near VWAP (< 0.5 ATR) | Gray | AVOID |

### Signal Engine (Weighted Scoring)

The v1.1 signal engine uses a **tiered weighted confluence scoring model** — 3 structural conditions (2x weight) and 9 confirmation conditions (1x weight).

**Gate conditions** (must all pass — AND-gated prerequisites, not scored):

- Bar is confirmed (closed)
- Within signal time window (default 10:00-15:45 ET)
- ADX above no-trend threshold (when enabled)
- Price not VWAP-extended (within 2.5 ATR)
- Cooldown elapsed (2+ bars since last signal)

**Structural conditions** (2x weight in weighted mode, 1x in equal mode):

| # | Condition | CALLS Logic | PUTS Logic | Weight |
|---|-----------|-------------|------------|--------|
| 1 | VWAP position | Close > VWAP or reclaiming | Close < VWAP or rejecting | 2x |
| 2 | EMA alignment | Fast > Slow or crossing above | Fast < Slow or crossing below | 2x |
| 3 | Anchor EMA | Close + fast EMA above anchor | Close + fast EMA below anchor | 2x |

**Confirmation conditions** (always 1x weight):

| # | Condition | CALLS Logic | PUTS Logic | Weight |
|---|-----------|-------------|------------|--------|
| 4 | RSI neutral zone | Between OS and OB (40-60) | Between OS and OB (40-60) | 1x |
| 5 | Candle pattern | Bullish engulfing, strong close, hammer, IB breakout, 3-bar up, morning star | Bearish engulfing, strong close, shooting star, IB breakout, 3-bar down, evening star | 1x |
| 6 | ADX trend present | ADX >= no-trend | ADX >= no-trend | 1x |
| 7 | 5m EMA trend | 5-min fast EMA > slow EMA | 5-min fast EMA < slow EMA | 1x |
| 8 | 5m MACD momentum | 5-min MACD histogram > 0 | 5-min MACD histogram < 0 | 1x |
| 9 | Level proximity | Near support (PM Low, OR Low, PDL, LOD, PD VWAP) | Near resistance (PM High, OR High, PDH, HOD, PD VWAP) | 1x |
| 10 | Squeeze fired | Squeeze fired within 2 bars (~30 min) | Squeeze fired within 2 bars (~30 min) | 1x |
| 11 | TICK breadth | Smoothed TICK > bullish threshold | Smoothed TICK < bearish threshold | 1x |
| 12 | RSI Divergence | Bullish divergence detected | Bearish divergence detected | 1x |

**Score summary**:

| Mode | Structural Max | Confirm Max | Total Max |
|------|---------------|-------------|-----------|
| Weighted (default) | 6 (3 x 2) | 9 (9 x 1) | 15 |
| Equal | 3 (3 x 1) | 9 (9 x 1) | 12 |

A signal fires when: gate passes AND total score >= `i_minScore` (default: 7).

**Why weighted scoring matters**: In equal-weight mode, a signal with 7 confirmation conditions but no structural alignment scores the same as one with all 3 structural conditions plus 1 confirmation. In weighted mode, the structural signal scores higher (7 vs 7), and signals without structural alignment are penalized. This produces higher-quality signals with fewer false positives.

**Candle pattern details** (unchanged from v1.0):

| Pattern | Key Threshold | Notes |
|---------|---------------|-------|
| Engulfing | Full body engulf | Standard |
| Strong close | Body > 60% of range | Raised for 15-min wicks |
| Hammer / Shooting star | Tail > 1.6x body | Relaxed for 15-min |
| Inside bar breakout | Prior bar inside bar[2] | Standard |
| 3-bar momentum | 3 consecutive directional closes | Standard |
| Morning star / Evening star | 3-bar reversal, doji middle | 15-min specific |

### Score Trend Indicator

**New in v1.1.** Tracks the best directional score (max of calls/puts scores) over the last 3 bars to show whether conditions are building toward or fading away from a signal.

| Trajectory | Condition | Color | Interpretation |
|-----------|-----------|-------|---------------|
| BUILDING | Score increased 2 consecutive bars | Green | Setup developing — signal may fire soon |
| RISING | Score increased 1 bar | Green | Conditions improving |
| STEADY | Score unchanged | Gray | Neutral |
| FALLING | Score decreased 1 bar | Red | Conditions deteriorating |
| FADING | Score decreased 2 consecutive bars | Red | Setup dissipating |

Dashboard Row 12 shows the trend label and the 3-bar score history (e.g., "3 > 5 > 7"). This is not a scoring condition — it is purely informational context for anticipating signal timing.

### Directional Bias Background

Optional (off by default) subtle background tint when regime is BULLISH or BEARISH. Signal background flashes take priority.

### Dashboard

26-row table positioned top-right with three columns: label, value, context.

| Row | Field | Value | Context/Label |
|-----|-------|-------|---------------|
| 0 | Header | "SPY 0DTE Scalper (15m) v1.1 [W]/[EQ]" | Scoring mode indicator |
| 1 | Price | Current close | Green Day / Red Day |
| 2 | Mode | Regime state | ACTIVE / CAUTION / AVOID |
| 3 | RSI | RSI value | OVERBOUGHT / OVERSOLD / NEUTRAL |
| 4 | ADX | ADX value | NO TREND / WEAK / TRENDING / STRONG |
| 5 | VWAP Dist | Distance from VWAP | EXTENDED / NEAR VWAP |
| 6 | PM High | Distance to PM High | RESISTANCE / CLEARED |
| 7 | PM Low | Distance to PM Low | SUPPORT / BROKEN |
| 8 | ATR | ATR value + regime | HIGH/NORMAL/LOW (ratio) |
| 9 | EMA Trend | Ribbon alignment | BULLISH / BEARISH / MIXED |
| 10 | Anchor EMA | Position vs anchor | ABOVE / BELOW / NEUTRAL + price |
| 11 | Signal | Last signal direction | Score (e.g., "9/15 (W)") |
| 12 | Score Trend | Trajectory label | 3-bar history (e.g., "3 > 5 > 7") |
| 13 | Day Chg | Intraday % change | — |
| 14 | Rel Vol | Volume / 20-bar SMA | SPIKE / ABOVE AVG / BELOW AVG |
| 15 | DI Spread | DI+ / DI- values | BULLS / BEARS |
| 16 | 5m Trend | 5-min EMA alignment | BULLISH / BEARISH / MIXED |
| 17 | 5m RSI | 5-min RSI value | OVERBOUGHT / OVERSOLD / NEUTRAL |
| 18 | 5m MACD | 5-min MACD histogram | BULLISH / BEARISH / FLAT |
| 19 | RSI Div | Divergence state | BULL DIV / BEAR DIV / NONE + lookback |
| 20 | Squeeze | TTM state | BUILDING / BREAKOUT / n bars ago / NORMAL |
| 21 | BB Width | Width ratio | EXPANDING / NORMAL / COMPRESSING |
| 22 | TICK | Raw (Smoothed) | EXTREME BULL/BEAR, BULLISH/BEARISH, MILD, FLAT |
| 23 | PD VWAP | Distance from PD VWAP | ABOVE/BELOW + price |
| 24 | Vol Regime | ATR classification | HIGH/NORMAL/LOW + "ATR Nx avg" |
| 25 | Session | Bar count / total | EARLY SESSION / LATE SESSION |

### Alerts

**Static alerts** (`alertcondition`) — 7 total:

- CALLS Signal
- PUTS Signal
- VWAP Cross Bullish
- VWAP Cross Bearish
- Regime Mode Change
- Squeeze Fired
- **RSI Divergence** (new in v1.1)

**Dynamic alerts** (`alert`) — 5 total:

- CALLS signal with interpolated context (score/max, scoring mode, RSI, ADX, VWAP dist, mode, anchor, vol regime, MTF, squeeze, TICK, divergence)
- PUTS signal with same context
- Mode shift
- Squeeze fired
- **RSI divergence detected** (new in v1.1)

---

## Visual Reference

### Color Map

| Element | Color | Hex |
|---------|-------|-----|
| VWAP | White | #FFFFFF |
| VWAP Bands | Cyan | #00BCD4 |
| Fast EMA | Yellow | #FFD600 |
| Mid EMA | Orange | #FF9100 |
| Slow EMA | Red | #FF1744 |
| Anchor EMA | Purple | #7C4DFF |
| Bullish Cloud | Green (85% transparent) | #00E676 |
| Bearish Cloud | Red (85% transparent) | #FF1744 |
| Mixed Cloud | Yellow (90% transparent) | #FFD600 |
| PM High | Bright Red | #FF5252 |
| PM Low | Bright Green | #69F0AE |
| PDH | Deep Orange | #FF6E40 |
| PDL | Teal | #64FFDA |
| PDC | Blue Gray | #B0BEC5 |
| **PD VWAP** | **Light Blue** | **#40C4FF** |
| OR High | Purple | #E040FB |
| OR Low | Light Purple | #B388FF |
| CALLS | Green | #00E676 |
| PUTS | Red | #FF1744 |
| Dashboard Header | Purple | #7C4DFF |
| **ATR HIGH / BB EXPANDING** | **Orange** | **#FF9100** |
| **ATR LOW / BB COMPRESSING** | **Cyan** | **#40C4FF** |

### Chart Element Legend

| Element | Style | When Visible |
|---------|-------|--------------|
| EMA Ribbon | 3 solid lines + cloud fill | Always |
| Anchor EMA | Thick purple line | Always |
| VWAP | White line, width 2 | When enabled |
| VWAP Bands | Cyan circles | When enabled |
| PM High/Low | Dashed lines + labels | RTH, when enabled |
| PDH/PDL/PDC | Dotted lines + labels | When enabled |
| **PD VWAP** | **Dotted line (width 2) + label** | **When enabled** |
| OR High/Low | Dashed lines + labels | After OR complete |
| HOD/LOD | Solid lines + labels | RTH, when enabled |
| CALLS Signal | Green triangle (small) + label + bg flash | On signal |
| PUTS Signal | Red triangle (small) + label + bg flash | On signal |
| VWAP Cross | Diamonds above/below bar | On confirmed cross |
| Bias Background | Very subtle green/red tint | When enabled + directional mode |

### Dashboard Fields

All dashboard fields update on the last bar only (`barstate.islast`) for performance. The header row displays "SPY 0DTE Scalper - 15m" with a purple accent bar.

---

## Tuning Guide

### Minimum Signal Score (Weighted vs Equal)

The most impactful setting. Score behavior differs between weighted and equal modes.

**Weighted mode** (max 15):

| Score Threshold | Expected Behavior |
|-----------------|-------------------|
| 4-5 | Permissive — fires with 2 structural + some confirmations. Many signals, lower average conviction. |
| 6 | All structural conditions met OR mixed structural + several confirmations. |
| 7 (default) | All structural (6 pts) + 1 confirmation, or 2 structural + 5 confirmations. Balanced. |
| 8-9 | High conviction — requires structural alignment plus multiple confirmations. |
| 10-12 | Very selective — may go sessions without firing. |
| 13-15 | Near-perfect alignment. Extremely rare signals. |

**Equal mode** (max 12):

| Score Threshold | Expected Behavior |
|-----------------|-------------------|
| 4 | Permissive. |
| 5-6 | Balanced — most core conditions pass. |
| 7-8 | Core + 2-3 enhancements. Higher conviction. |
| 9+ | Very high bar. 0-3 signals per session. |
| 11-12 | Near-perfect. May go sessions without firing. |

**Calibration from v1.0**: If you used `minScore=5` in v1.0 (equal, max 11), the equivalent in v1.1 weighted mode is approximately `minScore=7`.

### EMA Lengths

| Setting | Fast / Mid / Slow | Character |
|---------|-------------------|-----------|
| Responsive | 8 / 13 / 21 | Faster reactions, more cloud transitions |
| Default | 10 / 21 / 34 | Balanced structural context |
| Smooth | 13 / 26 / 50 | Very slow, highest conviction cloud state |

### Anchor EMA Length

| Setting | Effective Lookback | Character |
|---------|-------------------|-----------|
| 34 | ~8.5 hrs (1.3 sessions) | More responsive |
| 50 (default) | ~12.5 hrs (2 sessions) | Standard multi-session |
| 100 | ~25 hrs (4 sessions) | Weekly structural trend |

### ADX Thresholds

| Sensitivity | No-Trend | Trending | Strong |
|-------------|----------|----------|--------|
| Sensitive | 16 | 22 | 35 |
| Default | 20 | 25 | 40 |
| Conservative | 24 | 30 | 45 |

### VWAP Extended Distance

| Setting | Behavior |
|---------|----------|
| 1.5 ATR | Conservative — suppresses early |
| 2.5 ATR (default) | Balanced |
| 4.0 ATR | Permissive — only extreme extensions |

### Signal Cooldown

| Setting | Effective Time | Behavior |
|---------|---------------|----------|
| 1 bar | 15 min | Maximum frequency |
| 2 bars (default) | 30 min | Balanced |
| 3 bars | 45 min | Conservative |

### Signal Time Window

Default: 10:00-15:45 ET. First two bars establish the OR; signals begin after completion.

| Profile | Window | Rationale |
|---------|--------|-----------|
| Conservative | 10:15-15:30 | Extra buffer, avoid final 30 min |
| Default | 10:00-15:45 | Standard |
| Extended | 09:45-15:55 | Maximum window |

### Opening Range Duration

| Setting | Bars | Effective Time |
|---------|------|---------------|
| 15 min | 1 bar | Minimal |
| 30 min (default) | 2 bars | Institutional standard |
| 45 min | 3 bars | Extended balance |
| 60 min | 4 bars | Full first hour |

### RSI Thresholds

| Profile | OB / OS | Behavior |
|---------|---------|----------|
| Tight | 55 / 45 | Very narrow neutral zone |
| Default | 60 / 40 | Catches intraday exhaustion |
| Wide | 70 / 30 | Effectively disables RSI gating on 15-min |

### RSI Divergence Settings

| Parameter | Range | Notes |
|-----------|-------|-------|
| Lookback 3 bars | 45 min | More sensitive, more false positives |
| Lookback 5 bars (default) | 1.25 hrs | Balanced for 15-min |
| Lookback 8-10 bars | 2-2.5 hrs | Very structural, fewer signals |

The divergence detection uses a price proximity threshold of 0.5 ATR and an RSI minimum gap of 3 points. These are hardcoded to avoid over-parameterization but can be modified in the source.

### TTM Squeeze Settings

Default BB(20, 2.0) vs KC(20, 1.5) works well across timeframes. Squeeze recency window is 2 bars (~30 min).

### NYSE TICK Settings

| Profile | Bull / Bear / Extreme |
|---------|----------------------|
| Sensitive | 200 / -200 / 400 |
| Default | 300 / -300 / 600 |
| Conservative | 400 / -400 / 800 |

### Multi-Timeframe Toggle

When disabled, removes 1 `request.security()` tuple call (5-min RSI, EMA fast, EMA slow, MACD histogram). Conditions #7 and #8 will never score, reducing effective max score to 13 (weighted) or 10 (equal).

### Directional Bias Background

Off by default. Enable for constant visual context when using 15-min as a "set and glance" monitor.

### Tuning Profiles

#### Profile: Conservative / Trend-Following

Highest-conviction structural signals only. Expect 0-3 signals per session.

```
Scoring Mode:          Weighted
EMA Lengths:           13 / 26 / 50
Anchor EMA:            50
ADX No Trend:          24
Cooldown:              3 bars (45 min)
Signal Window:         10:15-15:30
RSI OB/OS:             55 / 45
RSI Divergence:        ON (lookback 7)
MTF Confirmation:      ON
Squeeze Detection:     ON
TICK Index:            ON
Bias Background:       ON
Min Score:             9
```

#### Profile: Balanced (Default)

Targeting 2-6 signals per session with solid conviction.

```
Scoring Mode:          Weighted
EMA Lengths:           10 / 21 / 34
Anchor EMA:            50
ADX No Trend:          20
Cooldown:              2 bars (30 min)
Signal Window:         10:00-15:45
RSI OB/OS:             60 / 40
RSI Divergence:        ON (lookback 5)
MTF Confirmation:      ON
Squeeze Detection:     ON
TICK Index:            ON
Bias Background:       OFF
Min Score:             7
```

#### Profile: Active Swing Trader

Experienced traders using 15-min for swing entries with 5-min/1-min for risk management.

```
Scoring Mode:          Equal
EMA Lengths:           8 / 15 / 26
Anchor EMA:            34
ADX No Trend:          16
Cooldown:              1 bar (15 min)
Signal Window:         09:45-15:55
RSI OB/OS:             65 / 35
RSI Divergence:        ON (lookback 3)
MTF Confirmation:      ON
Squeeze Detection:     OFF
TICK Index:            OFF
Bias Background:       OFF
Min Score:             5
```

---

## Architecture Notes

**Performance**: the script uses `var` declarations for persistent state (lines, labels, level values, cooldown counters). All dashboard updates occur only on `barstate.islast`. RSI divergence detection uses native `ta.lowest()` and `ta.highest()` with no loops. ATR regime and BB width percentile use simple `ta.sma()` comparisons. Score trend tracking uses bar indexing (`[1]`, `[2]`) with no arrays.

**`request.*()` budget**: the script makes 4 `request.*()` calls total, consolidated via tuple returns (v1.2.1):

- 1 tuple on `"1"` (NYSE TICK raw close + SMA(5) smoothed)
- 1 tuple on `"5"` (MTF RSI, EMA fast, EMA slow, MACD histogram)
- 1 tuple on `"D"` (prior day high, low, close, VWAP close, current day open)
- 1 for `request.footprint()` (volume footprint data)

This is well within TradingView's limit of 40 calls. Disabling MTF removes 1 call; disabling TICK removes 1; disabling footprint removes 1.

**Repainting**: all signal logic is gated by `barstate.isconfirmed`. MTF data uses `lookahead=barmerge.lookahead_off`. Prior day data uses `lookahead=barmerge.lookahead_on` with `[1]` offset for correct historical behavior. **Footprint data repaints by design** — `request.footprint()` has no `lookahead` parameter. Real-time uses 1-tick granularity (Premium+) while historical bars recalculate with coarser intervals. The footprint delta scoring condition (1x weight) is documented as repainting-capable; at 1 point on a 16-point scale (6.25%), the impact is within the scoring engine's noise floor.

**Footprint level proximity**: the prior bar's POC (`fpPocPrice[1]`) is injected into the `nearSupport` / `nearResistance` OR chains. The `[1]` offset ensures the POC reference is from a finalized bar, reducing repainting impact in the proximity calculation. Cumulative session delta (`fpCumDelta`) uses a `var` variable that resets at RTH open and accumulates each bar's delta — note that per-bar delta shifts from recalculation propagate into the cumulative total.

**Object limits**: Pine Script v6 allows 500 labels, 500 lines, 200 boxes. The addition of PD VWAP adds 1 line + 1 label. On a 15-min chart (~26 bars/session), object limits are never approached.

**Weighted scoring implementation**: uses two integer variables (`wStructural` and `wConfirm`) that are set based on the `i_useWeighted` toggle. When weighted, structural conditions add 2 to the score; when equal, they add 1. This is a minimal-overhead approach that avoids arrays or complex data structures.

**RSI divergence approach**: uses `ta.lowest()`/`ta.highest()` over the lookback window rather than pivot-based detection. This is more computationally efficient and avoids the lag inherent in waiting for pivot confirmation. The tradeoff is slightly higher sensitivity (more detections, including some that don't fully resolve), mitigated by the ATR proximity filter and RSI threshold gates.

**Score trend**: computed from `math.max(callsScore, putsScore)` on the current and prior 2 bars. The "best directional score" approach avoids the complexity of tracking calls and puts trends separately while providing the most actionable information (whether any directional setup is forming).

**Prior day VWAP**: uses `ta.vwap(hlc3)[1]` inside `request.security()` on the daily timeframe with `lookahead_on` to correctly fetch the prior day's closing VWAP value without repainting.

**MACD tuple destructuring**: `ta.macd()` returns `[macdLine, signalLine, histogram]` at global scope. The histogram is passed into the consolidated 5-min `request.security()` tuple alongside RSI and EMA values.

**Opening range bar math**: `orBarsNeeded = math.ceil(i_orMinutes / 15)`.

**TICK smoothing**: raw close and SMA(5) pulled in a single `request.security()` tuple on the 1-min timeframe for true 5-minute average breadth.

**Timezone**: session detection uses configurable `i_tz`, defaulting to `America/New_York`.

---

## Known Limitations

1. **Footprint data repaints by design**: Volume footprint values (delta, POC, VAH, VAL, buy/sell volume, cumulative delta) may differ between real-time and historical views due to TradingView's intrabar data resolution hierarchy. Real-time uses the most granular data available (1-tick on Premium+), while historical bars are recalculated with coarser intervals. This affects the delta scoring condition (+/-1 point), dashboard values, POC/VA step lines, and cumulative session delta. The prior-bar POC used in level proximity uses a `[1]` offset to minimize repainting. Requires TradingView Premium or Ultimate plan; returns N/A on lower plans.

2. **Cumulative session delta drift**: because per-bar delta values may shift during historical recalculation, the cumulative session delta (`FP Cum D` in the dashboard) can diverge from what was observed in real-time. Treat cumulative delta as directional context rather than a precise metric.

3. **Pre-market levels require extended hours**: if extended hours are not enabled, PM High/Low will not populate.

4. **Single-symbol calibration**: defaults are calibrated for SPY. Other instruments require parameter adjustment.

5. **No backtest capability**: this is an `indicator()`, not a `strategy()`.

6. **MTF request.security() limitations**: 5-min data updates when 5-min bars close, not continuously within 15-min bars.

7. **Opening range granularity**: effective range always rounds up to the nearest 15-min bar.

8. **TICK data availability**: requires exchange data in your TradingView plan. Only available during RTH.

9. **Squeeze recency window hardcoded**: the 2-bar window is in source code, not an input.

10. **TICK smoothing period hardcoded**: SMA(5) is in source code.

11. **Sparse signal environment**: ~26 RTH bars/session means signals are inherently rare. Some sessions may produce zero signals, particularly at higher min score settings.

12. **Morning star / evening star sensitivity**: the indecision bar threshold (body < 30% of range) may need adjustment in very volatile sessions.

13. **RSI divergence sensitivity**: the 0.5 ATR price proximity threshold and 3-point RSI minimum gap are hardcoded. Very tight ranges or extreme volatility may produce false positives or miss legitimate divergences. Adjust the lookback window as the primary tuning lever.

14. **Prior day VWAP accuracy**: `ta.vwap()` inside `request.security()` on the daily timeframe uses TradingView's session-anchored VWAP calculation. If the prior day's data is incomplete or the session definition differs from the chart, the value may not match what other platforms report.

15. **Score trend uses closing scores**: the 3-bar score history is computed from confirmed bar scores. Intra-bar score changes are not reflected until bar close.

16. **Weighted scoring favors structural alignment**: by design, a signal can reach score 6 (all structural) with zero confirmation conditions. At the default min score of 7, at least one confirmation is always required. Lowering below 6 in weighted mode allows pure-structural signals, which may be too permissive.
