# 0DTE SPY Scalper - 1m

**Platform**: TradingView
**Language**: Pine Script v6
**Chart**: SPY 1-Minute (optimized)
**Theme**: Dark theme (vibrant, high-contrast palette)

---

## Overview

A single-overlay indicator purpose-built for scalping 0DTE (zero days to expiration) SPY options on the 1-minute chart. It combines trend structure, momentum, volatility, volatility compression detection (TTM Squeeze), market breadth (NYSE TICK), and key price levels into a unified decision-support system with real-time signal generation and a heads-up dashboard.

The indicator does not auto-trade. It surfaces high-confluence setups as CALLS or PUTS labels on the chart, backed by an AND-gate signal engine that evaluates multiple independent conditions per bar with optional squeeze and breadth filters. All thresholds are configurable.

---

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Indicator Components](#indicator-components)
  - [EMA Ribbon](#ema-ribbon)
  - [VWAP and Bands](#vwap-and-bands)
  - [RSI](#rsi)
  - [ADX / DMI](#adx--dmi)
  - [ATR](#atr)
  - [TTM Squeeze](#ttm-squeeze)
  - [NYSE TICK Index](#nyse-tick-index)
  - [Volume Footprint](#volume-footprint)
  - [Key Price Levels](#key-price-levels)
  - [Regime / Mode Detection](#regime--mode-detection)
  - [Signal Engine](#signal-engine)
  - [Dashboard](#dashboard)
  - [Alerts](#alerts)
- [Visual Reference](#visual-reference)
  - [Color Map](#color-map)
  - [Chart Element Legend](#chart-element-legend)
  - [Dashboard Fields](#dashboard-fields)
- [Tuning Guide](#tuning-guide)
  - [Signal Strictness](#signal-strictness)
  - [EMA Lengths](#ema-lengths)
  - [ADX Thresholds](#adx-thresholds)
  - [VWAP Extended Distance](#vwap-extended-distance)
  - [Signal Cooldown](#signal-cooldown)
  - [Signal Time Window](#signal-time-window)
  - [Opening Range Duration](#opening-range-duration)
  - [RSI Thresholds](#rsi-thresholds)
  - [TTM Squeeze Settings](#ttm-squeeze-settings)
  - [NYSE TICK Settings](#nyse-tick-settings)
  - [Tuning Profiles](#tuning-profiles)
- [Architecture Notes](#architecture-notes)
- [Known Limitations](#known-limitations)
- [Changelog](#changelog)

---

## Features

- **EMA Ribbon** with dynamic cloud fill that shifts color based on trend alignment (bullish, bearish, mixed).
- **Session-anchored VWAP** with configurable standard deviation bands.
- **RSI, ADX, and ATR** calculated internally and displayed in the dashboard (not plotted as separate panes, preserving chart real estate).
- **TTM Squeeze detection** identifying Bollinger Band compression inside Keltner Channels — a precursor to volatility expansion and directional breakouts. Three states: SQUEEZE ON (compression building), FIRED (breakout beginning), and OFF (normal volatility). Optional signal filter suppresses entries during active squeeze.
- **NYSE TICK Index** pulled via `request.security()` for real-time market breadth confirmation. Six-tier classification from EXTREME BEAR to EXTREME BULL. Optional signal filter requires directional TICK alignment.
- **Volume Footprint** (v1.2) integration via `request.footprint()` for intrabar order flow visibility. Displays buy/sell volume, delta with strength classification (STRONG BUY through STRONG SELL), and Point of Control (POC) as a step-line reference level. Dashboard-only — does not affect signal generation. Requires TradingView Premium or Ultimate plan; degrades gracefully to N/A on lower plans.
- **Pre-Market High/Low** automatically detected and drawn as horizontal levels once RTH begins.
- **Prior Day High/Low/Close** pulled from the daily timeframe and plotted as dotted reference levels.
- **Opening Range** (configurable duration) captured at RTH open and drawn as dashed levels.
- **Session HOD/LOD** tracked in real time with dynamically updating lines.
- **Regime Classifier** that categorizes the current market state into one of five modes: BULLISH, BEARISH, RANGING, NO TRADE, or TRANSITION.
- **AND-gate signal engine** that requires all core conditions to pass and generates CALLS or PUTS labels, with optional squeeze and TICK filters that can be independently enabled.
- **VWAP cross markers** (diamond shapes) for quick visual identification of VWAP reclaims and rejections.
- **Real-time dashboard** (table overlay) showing 20 fields of live indicator data including squeeze state, TICK value, and footprint order flow, with color-coded status text.
- **Dual alert system**: static `alertcondition()` entries for TradingView's standard alert UI (including squeeze fired), plus dynamic `alert()` calls with interpolated context including squeeze and TICK status for webhook/notification pipelines.
- **Bar confirmation gate** to prevent signals from firing on incomplete (still-forming) bars, eliminating phantom signals.
- **Signal cooldown** to prevent label spam during rapid price action.

---

## Installation

1. Open TradingView and navigate to a **SPY 1-minute chart**.
2. Open the **Pine Editor** (bottom panel).
3. Delete any existing code in the editor.
4. Paste the entire contents of `spy_0dte_scalper_1min.pine` into the editor.
5. Click **"Add to chart"** (or "Save" then "Add to chart").
6. The indicator appears as an overlay with the dashboard in the top-right corner.

**Required chart settings**:

- **Extended hours**: Enable "Extended Trading Hours" in chart settings if you want Pre-Market High/Low levels to populate. Without extended hours data, those fields will show "N/A" in the dashboard and no PM level lines will render.
- **Timezone**: The script defaults to `America/New_York` and is configurable. Your TradingView chart timezone setting does not affect the indicator's session detection.

**Data requirements**:

- **NYSE TICK**: The TICK index (`USI:TICK`) requires your TradingView data plan to include the relevant exchange data. If the symbol is unavailable, the TICK dashboard row will show "N/A" and TICK-related signal filters will pass through (no blocking). You can disable the TICK feature entirely via the "Enable NYSE TICK Index" toggle.

---

## Indicator Components

### EMA Ribbon

Three exponential moving averages that form a visual ribbon around price action.

| Parameter | Default | Description |
|-----------|---------|-------------|
| Fast EMA Length | 8 | Fastest-reacting EMA, hugs price tightly |
| Mid EMA Length | 13 | Intermediate smoothing |
| Slow EMA Length | 21 | Trend anchor, slowest to react |
| Show EMA Cloud Fill | true | Toggles the shaded fill between fast and slow EMAs |

**Cloud behavior**:

- **Green fill**: all three EMAs are stacked bullish (fast > mid > slow).
- **Red fill**: all three EMAs are stacked bearish (fast < mid < slow).
- **Yellow fill**: mixed/transitional alignment.

The cloud provides an at-a-glance read on whether the short-term trend structure supports a directional bias. When the cloud is green and price is riding above the ribbon, the path of least resistance is up. Red cloud with price below = bearish structure. Yellow = chop, proceed with caution.

### VWAP and Bands

The Volume Weighted Average Price resets each trading session and serves as the primary institutional reference level for intraday trading.

| Parameter | Default | Description |
|-----------|---------|-------------|
| Show VWAP | true | Toggle VWAP line visibility |
| Show VWAP Bands | true | Toggle standard deviation bands |
| VWAP Band Multiplier | 1.0 | Number of standard deviations for upper/lower bands |

**Band calculation**: uses cumulative volume-weighted variance from session open. The bands represent one standard deviation of price dispersion around VWAP. Price outside the bands suggests overextension and potential mean-reversion risk.

**Dashboard integration**: the VWAP Dist field shows signed distance in points. When absolute distance exceeds 1 ATR, the status reads "EXTENDED" in red, warning against chasing.

### RSI

Relative Strength Index (momentum oscillator). Calculated internally and displayed in the dashboard only — not plotted on the chart to avoid cluttering the overlay.

| Parameter | Default | Description |
|-----------|---------|-------------|
| RSI Length | 14 | Lookback period |
| Overbought | 70 | Upper threshold |
| Oversold | 30 | Lower threshold |

**Dashboard status values**:

- **OVERBOUGHT** (red): RSI >= 70. Momentum is extended to the upside; CALLS signals are less reliable.
- **OVERSOLD** (green): RSI <= 30. Momentum is extended to the downside; PUTS signals are less reliable.
- **NEUTRAL** (white): RSI between 30-70. Directional room remains.

For 0DTE scalping, RSI is most useful as a filter rather than a standalone signal. The signal engine requires RSI to be within the neutral zone (between oversold and overbought) for both CALLS and PUTS to fire, avoiding entries at exhaustion points.

### ADX / DMI

The Average Directional Index measures trend strength regardless of direction. +DI and -DI measure the directional components.

| Parameter | Default | Description |
|-----------|---------|-------------|
| ADX Length | 14 | Lookback for ADX/DI calculation |
| No Trend Threshold | 15 | Below this = NO TRADE regime |
| Trending Threshold | 25 | Dividing line between weak and confirmed trend |
| Strong Trend Threshold | 40 | Above this = strong directional move |

**ADX status mapping**:

| ADX Value | Status | Color | Implication |
|-----------|--------|-------|-------------|
| < 15 | NO TREND | Red | Chop. Avoid trading. Theta decay will eat you alive. |
| 15 - 24.9 | WEAK/RANGING | Yellow | Directional bias is forming but not confirmed. |
| 25 - 39.9 | TRENDING | Green | Confirmed trend. Signals are higher conviction. |
| >= 40 | STRONG TREND | Green | Strong directional move. Trend-following entries preferred. |

**+DI / -DI**: displayed in the dashboard as a spread. When +DI > -DI, bullish pressure dominates. When -DI > +DI, bearish pressure dominates. The dashboard cell is colored green or red accordingly.

### ATR

Average True Range measures realized volatility on the 1-minute timeframe.

| Parameter | Default | Description |
|-----------|---------|-------------|
| ATR Length | 14 | Lookback period |

ATR is used in two ways:

1. **Dashboard display**: gives a real-time read on per-bar volatility. Useful for gauging whether current conditions support the expected move size for your option strike selection.
2. **Signal filtering**: the VWAP extension filter uses `ATR * i_vwapExtFilter` (default 1.5 ATR) to determine whether price is overextended from VWAP, suppressing signals when chasing risk is elevated.

### TTM Squeeze

The TTM Squeeze detects periods of volatility compression by comparing Bollinger Bands to Keltner Channels. When Bollinger Bands contract inside the Keltner Channels, a "squeeze" is active — volatility is compressing and a directional breakout is likely imminent. When the squeeze releases (BB expand outside KC), energy stored during compression typically drives a strong directional move.

| Parameter | Default | Description |
|-----------|---------|-------------|
| Enable TTM Squeeze Detection | true | Master toggle for squeeze calculations and dashboard display |
| BB Length | 20 | Bollinger Band SMA lookback |
| BB Multiplier | 2.0 | Standard deviations for Bollinger Bands |
| KC Length | 20 | Keltner Channel EMA lookback |
| KC Multiplier | 1.5 | ATR multiplier for Keltner Channels |

**Squeeze states**:

| State | Condition | Dashboard Color | Implication |
|-------|-----------|----------------|-------------|
| **SQUEEZE ON** | BB upper < KC upper AND BB lower > KC lower | Orange | Compression active. Volatility is building. A breakout is forming. Avoid entering — wait for the fire. |
| **FIRED** | Squeeze was ON on previous bar, now OFF | Green | Volatility expansion just began. Highest-probability moment for directional entry. |
| **OFF** | BB outside KC (normal volatility) | Gray/Green | Normal conditions. If recently fired (within 5 bars), shown in green with recency context. |

**Momentum histogram**: a linear regression of the midline delta between BB and KC basis is calculated internally. When the squeeze fires, this histogram's direction (positive = bullish, negative = bearish) indicates the likely breakout direction. This is included in the squeeze fired alert message.

**Signal integration**: the squeeze is integrated as an **opt-in AND-gate filter** on the signal engine via the "Require Squeeze Fired" toggle (default OFF). When enabled, signals are suppressed during active squeeze — they only fire when squeeze is OFF or has just FIRED. This prevents entering during compression when price is range-bound and 0DTE theta decay is highest.

**Why it matters for 0DTE**: volatility squeezes on the 1-minute chart often resolve within 5-15 bars. The squeeze-to-fire transition catches the exact moment energy releases, aligning your 0DTE entry with expanding realized volatility rather than decaying range-bound conditions.

### NYSE TICK Index

The NYSE TICK measures the net number of NYSE stocks ticking up minus those ticking down at any given moment. It serves as a real-time market breadth proxy — when TICK is strongly positive, broad buying pressure supports SPY upside; when strongly negative, selling pressure supports SPY downside.

| Parameter | Default | Description |
|-----------|---------|-------------|
| Enable NYSE TICK Index | true | Master toggle for TICK data pull and dashboard display |
| TICK Symbol | USI:TICK | Symbol for NYSE TICK data (also supports USI:TICK.NQ for NASDAQ) |
| TICK Bullish Threshold | +500 | Above this = bullish breadth confirmation |
| TICK Bearish Threshold | -500 | Below this = bearish breadth confirmation |
| TICK Extreme Threshold | 800 | Absolute value above this = extreme reading |

**TICK status tiers** (dashboard display):

| TICK Value | Status | Color | Implication |
|-----------|--------|-------|-------------|
| > +800 | EXTREME BULL | Green | Very strong buying breadth. High-conviction bullish environment. |
| +500 to +800 | BULLISH | Green | Broad buying pressure. CALLS entries are supported. |
| 0 to +500 | MILD BULL | Neutral | Slight positive breadth. Directional lean but not confirmed. |
| -500 to 0 | MILD BEAR | Neutral | Slight negative breadth. Directional lean but not confirmed. |
| -800 to -500 | BEARISH | Red | Broad selling pressure. PUTS entries are supported. |
| < -800 | EXTREME BEAR | Red | Very strong selling breadth. High-conviction bearish environment. |

**Data pull**: uses `request.security(i_tickSymbol, "1", close)` to pull the 1-minute TICK value. The TICK index only produces data during RTH (9:30 AM - 4:00 PM ET). During pre-market and after-hours, TICK returns `na` and the dashboard displays "N/A".

**Signal integration**: the TICK is integrated as an **opt-in AND-gate filter** via the "Require TICK Alignment" toggle (default OFF). When enabled, CALLS require `TICK > 0` (any positive breadth) and PUTS require `TICK < 0` (any negative breadth). This is a light directional filter — it does not require strong TICK readings, just confirmation that broad market direction aligns with the signal.

**Why it matters for 0DTE**: SPY is a basket of ~500 stocks. A SPY CALLS entry when the TICK is deeply negative means you're fighting broad market selling pressure — even if SPY's chart looks bullish, the underlying components are being sold. TICK alignment improves signal reliability by confirming that the directional bet is supported by broad market participation.

### Key Price Levels

Horizontal reference levels drawn on the chart. Each has a dashed or dotted line style and a right-edge label for identification.

#### Pre-Market High / Low

- **Detection**: tracks the highest high and lowest low during the pre-market session (4:00 AM - 9:29 AM ET).
- **Rendering**: dashed horizontal lines drawn once RTH begins, extending to the right edge.
- **Requirement**: the chart must have Extended Trading Hours enabled in chart settings. Without extended hours data, these levels will not populate.

#### Prior Day High / Low / Close

- **Source**: pulled from the daily timeframe via `request.security()` using the prior completed day's data.
- **Rendering**: dotted horizontal lines with right-edge labels (PDH, PDL, PDC).
- **No repainting risk**: the `[1]` offset with `lookahead_on` accesses only the prior completed daily bar, which is fixed.

#### Opening Range

- **Detection**: captures the high and low of the first N minutes of RTH (configurable, default 5 minutes).
- **Rendering**: dashed horizontal lines with right-edge labels (OR HIGH, OR LOW) drawn once the opening range period completes.
- **Usage**: opening range breakouts/breakdowns are classic 0DTE triggers. Price clearing OR HIGH with volume = bullish continuation. Price breaking OR LOW = bearish.

#### Session HOD / LOD

- **Detection**: running high-of-day and low-of-day tracked from RTH open, updated on every bar.
- **Rendering**: solid horizontal lines with right-edge labels (HOD, LOD) that dynamically move as new highs/lows print.
- **Reset**: clears and starts fresh at each RTH open.

### Regime / Mode Detection

A composite classifier that evaluates the current market state by combining ADX trend strength, EMA ribbon alignment, and price position relative to VWAP.

| Mode | Conditions | Signal Behavior |
|------|-----------|-----------------|
| **BULLISH** | ADX >= 15, EMAs stacked bullish, price above VWAP | CALLS favored |
| **BEARISH** | ADX >= 15, EMAs stacked bearish, price below VWAP | PUTS favored |
| **RANGING** | ADX < 15 with price away from VWAP, or ADX 15-25 with mixed EMA/VWAP | Both directions possible (lower conviction) |
| **NO TRADE** | ADX < 15 and price near VWAP (within 0.5 ATR) | Low-conviction chop. Avoid. |
| **TRANSITION** | ADX >= 25 but mixed EMA/VWAP alignment | Emerging trend; use caution |

The regime classifier serves as a top-level contextual indicator. Its primary purpose is informing you about current market state so you can avoid taking directional 0DTE trades during choppy, low-ADX conditions where theta decay dominates and price whipsaws generate false signals.

The mode is displayed prominently in the dashboard and updates in real time. Mode changes trigger alerts.

### Signal Engine

The core decision engine that generates CALLS and PUTS labels on the chart.

#### AND-Gate Model

The 1-minute variant uses an AND-gate model: all core conditions must pass simultaneously, plus any enabled optional filters. This is stricter than the 5-minute variant's scoring model but appropriate for the higher signal frequency on 1-minute bars.

**CALLS conditions (all must be true)**:

| # | Condition | Rationale |
|---|-----------|-----------|
| 1 | Price is above VWAP or just crossed above | Institutional bias is bullish |
| 2 | EMA fast > slow, or price reclaiming slow EMA | Trend structure supports upside |
| 3 | RSI is between oversold and overbought (30-70) | Momentum has room to run |
| 4 | Bullish candle pattern (engulfing, strong close, or hammer) | Price action confirmation |

**PUTS conditions (all must be true)**:

| # | Condition | Rationale |
|---|-----------|-----------|
| 1 | Price is below VWAP or just crossed below | Institutional bias is bearish |
| 2 | EMA fast < slow, or price breaking slow EMA | Trend structure supports downside |
| 3 | RSI is between oversold and overbought (30-70) | Momentum has room to run |
| 4 | Bearish candle pattern (engulfing, strong close, or shooting star) | Price action confirmation |

#### Signal Filters

In addition to the core conditions, the following filters must all pass. These are evaluated before the core conditions — if any filter fails, the signal is suppressed regardless of core condition alignment.

| Filter | Default | Description |
|--------|---------|-------------|
| **Time Window** | 09:35-15:45 ET | Signal must occur within the configured window. Avoids opening auction chop and final gamma/pin risk. |
| **Cooldown** | 5 bars (5 min) | Minimum bars since last signal. Prevents label spam. |
| **Bar Confirmation** | On | Signals fire only on confirmed (closed) bars. Prevents phantom signals. |
| **ADX Gate** | On | ADX must be above No-Trend Threshold (15). |
| **VWAP Extension** | 1.5 ATR | Signals suppressed when price is > 1.5 ATR from VWAP. |
| **Squeeze Filter** | Off | When enabled, suppresses signals during active squeeze (squeeze ON). Signals only fire when squeeze is OFF or just FIRED. |
| **TICK Filter** | Off | When enabled, CALLS require TICK > 0, PUTS require TICK < 0. Light directional breadth confirmation. |

The squeeze and TICK filters default to OFF so they do not affect the v1.0 signal behavior unless explicitly enabled. When enabled, they act as additional AND-gate conditions — they must pass alongside all other filters and core conditions.

#### Visual Output

When a signal fires, the following visual elements render on that bar:

1. **Label**: "CALLS" (green, below bar) or "PUTS" (red, above bar) with a hover tooltip showing RSI, ADX, VWAP distance, mode, ATR, EMA status, squeeze state, and TICK value.
2. **Arrow**: small triangle shape (up for CALLS, down for PUTS) as a secondary quick-scan marker.
3. **Background flash**: faint green or red background highlight on the signal bar (toggleable).

#### VWAP Cross Markers

Independent of the signal engine, small diamond shapes appear on the chart whenever price crosses VWAP during RTH:

- **Lime diamond below bar**: bullish VWAP cross (price crossed above).
- **Red diamond above bar**: bearish VWAP cross (price crossed below).

These are informational markers, not trade signals. They highlight potential inflection points worth watching.

### Dashboard

A real-time information table overlaid on the chart. Positioned in the top-right corner by default (configurable).

#### Layout

Three-column table with a header row and 15 data rows (16 total). Dark navy background with high-contrast text. Color-coded values provide instant status reads without needing to interpret raw numbers.

#### Fields

| Row | Field | Value | Color Logic |
|-----|-------|-------|-------------|
| Header | SPY 0DTE Scalper - 1m | — | Purple accent bar |
| 1 | Price | Current close + bar direction | Green = green bar, Red = red bar |
| 2 | Mode | Regime classification + advice | Green/Red/Yellow/Orange/Gray per mode |
| 3 | RSI (14) | RSI value + status | Red = overbought, Green = oversold, White = neutral |
| 4 | ADX | ADX value + trend status | Red = no trend, Yellow = weak, Green = trending |
| 5 | VWAP Dist | Signed distance in points + status | Green = near, Yellow = moderate, Red = extended |
| 6 | PM High | Distance to PM High + status | Red = resistance (below), Green = cleared (above) |
| 7 | PM Low | Distance to PM Low + status | Green = support (above), Red = broken (below) |
| 8 | ATR (14) | Current ATR value | Neutral (informational) |
| 9 | EMA Trend | Ribbon alignment status | Green = bullish, Red = bearish, Yellow = mixed |
| 10 | Signal | Last signal on current bar | Green = CALLS, Red = PUTS, Gray = none |
| 11 | Day Chg | Percentage change from session open | Green = positive, Red = negative |
| 12 | Rel Vol | Current volume / 20-bar avg volume | Green + "SPIKE" if > 1.5x, Yellow = above avg, Gray = below |
| 13 | DI Spread | +DI / -DI directional values | Green = +DI dominant (BULLS), Red = -DI dominant (BEARS) |
| 14 | Squeeze | TTM Squeeze state + context | Orange = ON/BUILDING, Green = FIRED/BREAKOUT or recent, Gray = OFF/NORMAL |
| 15 | TICK | NYSE TICK value + breadth tier | Green = bullish/extreme bull, Red = bearish/extreme bear, Neutral = mild, Gray = N/A or OFF |

When TTM Squeeze or NYSE TICK features are disabled, their respective rows display "OFF" in gray. When TICK data is unavailable (pre-market, after-hours, or data plan limitation), the row displays "N/A".

### Alerts

#### Static Alerts (alertcondition)

These appear in TradingView's "Create Alert" dropdown when the indicator is on your chart. You manually configure each alert and choose your notification method.

| Alert Name | Trigger |
|------------|---------|
| 0DTE: CALLS Signal | CALLS signal fires |
| 0DTE: PUTS Signal | PUTS signal fires |
| 0DTE: VWAP Cross Bullish | Price crosses above VWAP during RTH |
| 0DTE: VWAP Cross Bearish | Price crosses below VWAP during RTH |
| 0DTE: Regime Mode Change | Regime mode transitions to a different state |
| 0DTE: Squeeze Fired | TTM Squeeze transitions from ON to OFF (volatility expansion) |

#### Dynamic Alerts (alert)

These fire automatically with rich, interpolated messages containing real-time context. Useful for webhook integrations (Discord bots, Telegram, custom APIs).

**Example CALLS alert message**:

```
0DTE CALLS | SPY 586.42 | RSI 54.3 | ADX 27.8 | VWAP Dist 0.65 | Mode: BULLISH | Sqz: OFF | TICK: 342
```

**Example squeeze fired alert message**:

```
0DTE SQUEEZE FIRED | SPY 586.80 | Momentum: BULLISH | ADX 22.5 | Mode: TRANSITION
```

**Example mode change alert message**:

```
0DTE MODE SHIFT | RANGING -> BULLISH | SPY 587.10
```

Dynamic alerts use `alert.freq_once_per_bar_close` for signals and squeeze fires (no duplicates within a bar) and `alert.freq_once_per_bar` for mode changes. Squeeze and TICK context is appended to signal alerts only when the respective feature is enabled.

---

## Visual Reference

### Color Map

All colors are selected for maximum contrast against TradingView's dark theme (pure black background).

| Element | Hex | Preview | Notes |
|---------|-----|---------|-------|
| VWAP | `#FFFFFF` | White | Solid line, weight 2 |
| VWAP Bands | `#00BCD4` | Cyan | 50% transparency, circle style |
| EMA 8 (Fast) | `#FFD600` | Bright Yellow | Thin line |
| EMA 13 (Mid) | `#FF9100` | Bright Orange | Thin line |
| EMA 21 (Slow) | `#FF1744` | Hot Pink/Red | Thin line |
| EMA Cloud (Bull) | `#00E676` | Neon Green | 85% transparent fill |
| EMA Cloud (Bear) | `#FF1744` | Neon Red | 85% transparent fill |
| EMA Cloud (Mixed) | `#FFD600` | Yellow | 90% transparent fill |
| PM High | `#FF5252` | Bright Red | Dashed line + right label |
| PM Low | `#69F0AE` | Bright Green | Dashed line + right label |
| Prior Day High | `#FF6E40` | Coral | Dotted line + "PDH" label |
| Prior Day Low | `#64FFDA` | Teal | Dotted line + "PDL" label |
| Prior Day Close | `#B0BEC5` | Gray | Dotted line + "PDC" label |
| OR High | `#E040FB` | Magenta | Dashed line + "OR HIGH" label |
| OR Low | `#B388FF` | Purple | Dashed line + "OR LOW" label |
| Session HOD | `#00E676` | Neon Green | Solid line + "HOD" label |
| Session LOD | `#FF1744` | Neon Red | Solid line + "LOD" label |
| CALLS Signal | `#00E676` | Neon Green | Label + arrow + bg flash |
| PUTS Signal | `#FF1744` | Neon Red | Label + arrow + bg flash |
| VWAP Cross (Bull) | `#76FF03` | Lime | Diamond shape below bar |
| VWAP Cross (Bear) | `#FF1744` | Red | Diamond shape above bar |
| Squeeze ON | `#FF9100` | Orange | Dashboard status color |
| Squeeze FIRED | `#00E676` | Green | Dashboard status color |

### Chart Element Legend

Every plotted line registers in TradingView's built-in legend (top-left, next to ticker info). You can click any legend entry to toggle its visibility. The entries are:

```
EMA 8 (Fast)  |  EMA 13 (Mid)  |  EMA 21 (Slow)  |  VWAP  |  VWAP +1σ  |  VWAP -1σ
CALLS Arrow  |  PUTS Arrow  |  VWAP Cross Bull  |  VWAP Cross Bear  |  Signal Background Flash
```

Horizontal key levels (PM High/Low, PDH/PDL/PDC, OR, HOD/LOD) are drawn with `line.new()` and labeled at the right edge of the chart, so they are identified by their text labels rather than legend entries.

### Dashboard Fields

See the [Dashboard](#dashboard) section for the full field table. The dashboard uses a dark navy background (`#1A1A2E`) with a magenta header bar and light gray metric labels. Value cells are dynamically colored based on the state of each metric.

---

## Tuning Guide

All parameters are exposed as configurable inputs in TradingView's indicator settings panel. This section explains what each tunable controls and how to adjust it for different trading styles.

### Signal Strictness

The 1-minute variant uses an AND-gate model where all core conditions must pass. The primary strictness controls are the optional filters (ADX gate, squeeze filter, TICK filter) and the signal filter thresholds.

With default settings (squeeze and TICK filters OFF), the signal engine requires: VWAP position + EMA alignment + RSI neutral + candle pattern + ADX gate + time window + cooldown + VWAP extension check.

Enabling additional filters progressively tightens entry requirements:

| Configuration | Effective Strictness | Typical Signals/Session |
|--------------|---------------------|------------------------|
| Defaults (squeeze OFF, TICK OFF) | Moderate | 5-15 |
| Squeeze filter ON | Stricter | 3-10 |
| TICK filter ON | Stricter | 3-10 |
| Both squeeze + TICK ON | Very strict | 1-5 |

### EMA Lengths

**Inputs**: Fast EMA Length, Mid EMA Length, Slow EMA Length
**Defaults**: 8, 13, 21

The EMA ribbon defines short-term trend structure. These lengths are Fibonacci-based and work well on the 1-minute chart for capturing momentum shifts over 8-21 minute windows.

| Adjustment | Effect |
|------------|--------|
| Shorter lengths (e.g., 5/8/13) | More responsive ribbon. Cloud flips faster. Signals trigger earlier but with more noise. Better for ultra-fast scalps (30-second to 2-minute holds). |
| Longer lengths (e.g., 13/21/34) | Smoother ribbon. Cloud flips less often. Signals are slower but more reliable. Better for slightly longer scalps (3-10 minute holds). |
| Wider spread (e.g., 5/13/34) | Cloud is wider. Bullish/bearish stacking requires a stronger move to confirm. Fewer mixed-cloud periods. |

**Important**: the EMA stack (bullish = fast > mid > slow) directly feeds the regime classifier and signal conditions. Changing these lengths changes when the mode flips and when signals fire.

### ADX Thresholds

**Inputs**: No Trend Threshold, Trending Threshold, Strong Trend Threshold
**Defaults**: 15, 25, 40

ADX thresholds control the regime classifier and the minimum directional strength required for signals.

| Parameter | Tuning Impact |
|-----------|---------------|
| No Trend Threshold (15) | **Raising this** (e.g., to 20) makes the NO TRADE regime more aggressive, suppressing signals in marginal conditions. **Lowering it** (e.g., to 10) allows signals in choppier environments. For 0DTE, erring higher is safer because theta decay punishes low-conviction entries. |
| Trending Threshold (25) | Controls when the regime classifies as TRENDING vs WEAK/RANGING. Raising this requires stronger trends before directional bias kicks in. |
| Strong Trend Threshold (40) | Informational for the dashboard. Does not directly affect signal logic. |

**Recommendation**: if you trade the opening 30 minutes (high-vol environment), the default of 15 works well. If you trade the midday session where SPY often ranges, consider raising the No Trend Threshold to 18-20 to avoid getting chopped up.

### VWAP Extended Distance

**Input**: Max VWAP Distance (ATR multiples)
**Default**: 1.5

Controls when signals are suppressed due to overextension from VWAP.

| Value | Behavior |
|-------|----------|
| Lower (1.0) | Very conservative. Suppresses signals early. Prevents chasing extended moves. |
| Default (1.5) | Balanced for 1-minute price action. |
| Higher (2.0-3.0) | Allows wider moves before suppression. Better for trending days where SPY can sustain extended runs from VWAP. |

### Signal Cooldown

**Input**: Cooldown Bars Between Signals
**Default**: 5

Minimum number of bars (minutes on 1-min chart) that must pass before another signal can fire.

| Value | Behavior |
|-------|----------|
| Lower (1-3) | Signals fire more frequently during sustained moves. Can result in label clusters during strong trends. Useful if you're scaling into positions. |
| Higher (10-20) | Signals fire rarely. Each one represents a distinct setup well-separated in time. Useful if you take one position at a time and hold for several minutes. |
| Default (5) | One signal per 5-minute window. Reasonable balance for most scalping styles. |

### Signal Time Window

**Input**: Signal Time Window
**Default**: 0935-1545

Defines the window during which signals are allowed to fire. Times are in Eastern Time (ET).

| Adjustment | Rationale |
|------------|-----------|
| Start at 0930 | Include the opening bar. High volatility, high reward, but also high noise and whipsaw risk. |
| Start at 0935 (default) | Skip the first 5 minutes. Avoids the opening auction chaos where 0DTE spreads are widest and fills are worst. |
| Start at 0945 | Skip the first 15 minutes. Wait for the opening range to establish before taking signals. More conservative. |
| End at 1545 (default) | Stop signals 15 minutes before close. Avoids the final gamma/pin risk window where 0DTE options can swing violently on delta hedging flows. |
| End at 1555 | Allow signals into the last 5 minutes. Only for experienced traders who understand pin risk and are comfortable with rapid gamma decay. |
| End at 1500 | Stop signals an hour before close. Very conservative. Avoids the entire late-day volatility regime shift. |

### Opening Range Duration

**Input**: Opening Range Minutes
**Default**: 5

Number of minutes from RTH open to capture for the opening range.

| Value | Usage |
|-------|-------|
| 5 (default) | 5-minute opening range. The most common institutional reference. Breakouts above OR High or below OR Low on volume are high-probability 0DTE triggers. |
| 15 | 15-minute opening range. More conservative, wider range. Breakouts are higher conviction but take longer to set up. |
| 2-3 | Micro opening range. Very tight levels that break quickly. Useful for the most aggressive opening trades but generates more false breakouts. |

### RSI Thresholds

**Inputs**: Overbought, Oversold
**Defaults**: 70, 30

Controls the RSI zone classification and the signal filter that prevents entries at momentum extremes.

The signal engine requires RSI to be between oversold and overbought for both CALLS and PUTS to fire. This prevents buying calls when momentum is already exhausted (overbought) or buying puts when selling is overdone (oversold).

| Adjustment | Effect |
|------------|--------|
| Tighter (65/35) | Signals are filtered out earlier as RSI approaches extremes. Fewer signals, but each has more momentum room. |
| Wider (75/25) | Allows signals closer to extremes. More signals fire, but some may be at points where the move is nearly exhausted. |

### TTM Squeeze Settings

**Inputs**: BB Length, BB Multiplier, KC Length, KC Multiplier
**Defaults**: 20, 2.0, 20, 1.5

The default BB(20, 2.0) vs KC(20, 1.5) is the standard TTM Squeeze configuration used across most platforms. Adjustments change squeeze sensitivity:

| Adjustment | Effect |
|------------|--------|
| Lower KC Multiplier (1.0-1.25) | Keltner Channels are narrower. Squeezes trigger more frequently (BB compress inside KC more easily). More squeeze events but each may be less significant. |
| Higher KC Multiplier (1.75-2.0) | Keltner Channels are wider. Squeezes are rarer and represent more extreme compression events. Higher conviction when they fire. |
| Shorter lengths (10-15) | More responsive to recent volatility changes. Squeezes resolve faster. Better for ultra-short-term 1-minute scalps. |
| Longer lengths (25-30) | Smoother. Squeezes represent more sustained compression periods. Fewer false squeeze events. |

**Squeeze filter toggle**: "Require Squeeze Fired" (default OFF). Enable this to suppress all signals during active squeeze periods. When ON, signals only fire when the squeeze is OFF or has just FIRED, aligning entries with volatility expansion rather than compression.

### NYSE TICK Settings

**Inputs**: TICK Bullish/Bearish/Extreme Thresholds
**Defaults**: +500 / -500 / 800

These thresholds control the dashboard status tiers and (when the TICK filter is enabled) the directional alignment requirements.

| Adjustment | Effect |
|------------|--------|
| Tighter bullish/bearish (300/-300) | Dashboard classifies breadth as bullish/bearish more easily. Informational only — the signal filter uses TICK > 0 / < 0 regardless. |
| Wider bullish/bearish (700/-700) | Dashboard requires stronger readings before labeling breadth as directional. |
| Higher extreme (1000) | Fewer EXTREME BULL/BEAR classifications. Only the most powerful breadth readings qualify. |
| Lower extreme (600) | More frequent extreme classifications. |

**TICK filter toggle**: "Require TICK Alignment" (default OFF). When ON, CALLS require `TICK > 0` and PUTS require `TICK < 0`. This is a light filter — it only requires directional agreement, not a strong reading. It catches the worst-case scenario: entering a CALLS trade when broad market breadth is negative.

**Symbol selection**: `USI:TICK` covers NYSE-listed stocks. `USI:TICK.NQ` covers NASDAQ-listed stocks. For SPY trading, `USI:TICK` is the standard choice since SPY's components are primarily NYSE-listed.

### Tuning Profiles

Here are three preset configurations for common trading styles. Apply these by adjusting the inputs in the indicator settings.

#### Profile: Conservative / Sniper

For traders who want 1-5 high-conviction trades per session.

```
EMA Lengths:           13 / 21 / 34
ADX No Trend:          20
Cooldown:              10
Signal Start:          0945
Signal End:            1530
RSI OB/OS:             65 / 35
Squeeze Filter:        ON
TICK Filter:           ON
```

#### Profile: Balanced (Default)

For traders who want moderate signal frequency with reasonable accuracy.

```
EMA Lengths:           8 / 13 / 21
ADX No Trend:          15
Cooldown:              5
Signal Start:          0935
Signal End:            1545
RSI OB/OS:             70 / 30
Squeeze Filter:        OFF
TICK Filter:           OFF
```

#### Profile: Aggressive / Scalper

For experienced traders who want frequent signals and manage risk through position sizing and rapid exits.

```
EMA Lengths:           5 / 8 / 13
ADX No Trend:          12
Cooldown:              3
Signal Start:          0930
Signal End:            1555
RSI OB/OS:             75 / 25
Squeeze Filter:        OFF
TICK Filter:           OFF
```

---

## Architecture Notes

**Performance**: the script uses `var` declarations for persistent state (lines, labels, level values, cooldown counters) to avoid reallocation on every bar. The dashboard table is updated only on `barstate.islast` to reduce computation overhead on historical bars. TTM Squeeze calculations (BB, KC, linear regression) use native `ta.*` functions and add negligible overhead.

**`request.*()` budget**: the script makes 3 `request.*()` calls total, consolidated via tuple returns (v1.0.1): 1 tuple on `"D"` (prior day high, low, close, current day open), 1 for NYSE TICK on `"1"` (same-timeframe, no bar-count multiplier), and 1 for `request.footprint()` (volume footprint data). This is well within TradingView's limit of 40 calls per script. When TICK is disabled, 2 calls are active; when footprint is also disabled, 1 call.

**Repainting**: all signal logic is gated by `barstate.isconfirmed` by default. This means signals only appear after a bar closes, not during its formation. This eliminates phantom signals but introduces a 1-bar delay on live charts. TICK data from `request.security()` uses `lookahead=barmerge.lookahead_off` ensuring no forward-looking data.

**Object limits**: Pine Script v6 allows a maximum of 500 labels, 500 lines, and 200 boxes per script. On a full trading day (~390 RTH bars on 1-min), signal labels, level labels, and HOD/LOD lines are well within these limits. However, if you run the script across multiple days of extended hours data, older labels and lines will be automatically garbage-collected by TradingView (oldest first).

**VWAP bands**: the band calculation uses cumulative variance (`ta.cum`) which resets with the session. On the first bar of each session, variance is zero and bands will overlap VWAP. This is expected behavior and resolves within a few bars.

**Timezone**: session detection uses the configurable `i_tz` input, defaulting to `America/New_York`.

**Squeeze momentum**: the linear regression histogram (`ta.linreg`) is calculated but not plotted on the chart (it's an overlay indicator, not a pane). The momentum value and direction are included in squeeze fired alerts and are available for future enhancements.

---

## Known Limitations

1. **Footprint data repaints by design**: Volume footprint values (delta, POC, buy/sell volume) may differ between real-time and historical views due to TradingView's intrabar data resolution hierarchy. Real-time uses the most granular data available (1-tick on Premium+), while historical bars are recalculated with coarser intervals. This affects dashboard values only — footprint data does not influence signal generation on the 1-minute variant. Requires TradingView Premium or Ultimate plan; returns N/A on lower plans.

2. **Footprint POC is approximate**: the POC step-line plots the midpoint of the highest-volume price row. With the default 10 ticks/row ($0.10 per row on SPY), the plotted POC may differ from the exact highest-volume tick by up to $0.05.

3. **Pre-market levels require extended hours**: if your chart does not have extended trading hours enabled, PM High/Low will not populate. The dashboard will show "N/A" and no PM level lines will render.

4. **Single-symbol calibration**: the script is designed for SPY. It will technically work on any symbol, but the default parameters (VWAP extended distance, ATR-based level proximity, dashboard labeling, TICK thresholds) are calibrated for SPY's price range and volatility characteristics. Significant parameter adjustment would be needed for other instruments.

5. **TICK data availability**: the NYSE TICK symbol (`USI:TICK`) requires your TradingView data plan to include the relevant exchange. If unavailable, the TICK row shows "N/A" and TICK-based signal filters pass through without blocking. TICK data is only available during RTH (9:30 AM - 4:00 PM ET).

6. **No backtest capability**: this is an `indicator()`, not a `strategy()`. It cannot be backtested through TradingView's strategy tester. Converting to a strategy would require defining entry/exit rules, position sizing, and stop/target logic.

7. **Equal-weighted conditions**: all AND-gate conditions are treated as binary pass/fail. In practice, some conditions (like VWAP position) may be more predictive than others. The 5-minute variant's scoring model partially addresses this by allowing partial confluence.
