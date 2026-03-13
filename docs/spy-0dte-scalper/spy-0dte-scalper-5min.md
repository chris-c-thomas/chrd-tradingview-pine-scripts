# 0DTE SPY Scalper - 5m

**Platform**: TradingView
**Language**: Pine Script v6
**Chart**: SPY 5-Minute (optimized)
**Theme**: Dark theme (vibrant, high-contrast palette)

---

## Overview

A single-overlay indicator purpose-built for scalping 0DTE (zero days to expiration) SPY options on the 5-minute chart. It combines trend structure, momentum, volatility, volatility compression detection (TTM Squeeze), market breadth (NYSE TICK), key price levels, and a multi-timeframe confirmation layer into a unified decision-support system with real-time signal generation and a heads-up dashboard.

This is the 5-minute companion to the 1-minute SPY 0DTE Scalper. It targets swing-scalp trades with 5-25 minute hold times. The 5-minute chart produces ~78 RTH bars per session, so the signal engine is calibrated for higher conviction per signal with lower total signal count compared to the 1-minute variant.

The indicator does not auto-trade. It surfaces high-confluence setups as CALLS or PUTS labels on the chart, backed by a scoring engine that evaluates up to nine independent conditions per bar. All thresholds are configurable.

---

## Table of Contents

- [Features](#features)
- [Key Differences from 1-Minute Variant](#key-differences-from-1-minute-variant)
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
  - [Multi-Timeframe Confirmation](#multi-timeframe-confirmation)
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
  - [Minimum Signal Score](#minimum-signal-score)
  - [EMA Lengths](#ema-lengths)
  - [ADX Thresholds](#adx-thresholds)
  - [VWAP Extended Distance](#vwap-extended-distance)
  - [Signal Cooldown](#signal-cooldown)
  - [Signal Time Window](#signal-time-window)
  - [Opening Range Duration](#opening-range-duration)
  - [RSI Thresholds](#rsi-thresholds)
  - [TTM Squeeze Settings](#ttm-squeeze-settings)
  - [NYSE TICK Settings](#nyse-tick-settings)
  - [Multi-Timeframe Toggle](#multi-timeframe-toggle)
  - [Tuning Profiles](#tuning-profiles)
- [Architecture Notes](#architecture-notes)
- [Known Limitations](#known-limitations)
- [Changelog](#changelog)

---

## Features

- **EMA Ribbon** (9/15/21) with dynamic cloud fill that shifts color based on trend alignment (bullish, bearish, mixed).
- **Session-anchored VWAP** with configurable standard deviation bands.
- **RSI, ADX, and ATR** calculated internally and displayed in the dashboard (not plotted as separate panes, preserving chart real estate).
- **TTM Squeeze detection** identifying Bollinger Band compression inside Keltner Channels — a precursor to volatility expansion and directional breakouts. Three states: SQUEEZE ON (compression building), FIRED (breakout beginning), and OFF (normal volatility). Integrated as an enhancement scoring condition with recency window.
- **NYSE TICK Index** pulled via `request.security()` for real-time market breadth confirmation. Six-tier classification from EXTREME BEAR to EXTREME BULL. Integrated as an enhancement scoring condition requiring strong directional breadth.
- **Volume Footprint** (v1.2) integration via `request.footprint()` for intrabar order flow visibility. Displays delta with strength classification, buy/sell volume, and Point of Control (POC) as a step-line level. Optional enhancement scoring condition #10 (OFF by default to preserve backward compatibility). Requires TradingView Premium or Ultimate plan; degrades gracefully to N/A on lower plans.
- **Multi-Timeframe confirmation layer** pulling 1-minute RSI and EMA trend alignment via `request.security()` for lower-timeframe context on the 5-minute chart.
- **Pre-Market High/Low** automatically detected and drawn as horizontal levels once RTH begins.
- **Prior Day High/Low/Close** pulled from the daily timeframe and plotted as dotted reference levels.
- **Opening Range** (default 15 minutes / 3 bars) captured at RTH open and drawn as dashed levels.
- **Session HOD/LOD** tracked in real time with dynamically updating lines.
- **Regime Classifier** that categorizes the current market state into one of five modes: BULLISH, BEARISH, RANGING, NO TRADE, or TRANSITION.
- **5+4+1 confluence scoring engine** with 5 core conditions, 4 optional enhancement conditions (MTF trend, level proximity, squeeze recency, TICK breadth), and 1 optional footprint delta condition (#10, OFF by default). Generates CALLS or PUTS labels when the score meets the configurable threshold.
- **Expanded candle pattern detection** optimized for 5-minute bars: engulfing, strong close, hammer/shooting star, inside bar breakout, and 3-bar momentum.
- **VWAP cross markers** (diamond shapes) for quick visual identification of VWAP reclaims and rejections.
- **Real-time dashboard** (table overlay) showing 21 fields of live indicator data including MTF, squeeze, TICK, and footprint order flow rows, with color-coded status text.
- **Dual alert system**: static `alertcondition()` entries for TradingView's standard alert UI (including squeeze fired), plus dynamic `alert()` calls with interpolated context including score, MTF, squeeze, and TICK status.
- **Bar confirmation gate** to prevent signals from firing on incomplete (still-forming) bars.
- **Signal cooldown** (default 3 bars = 15 minutes) to prevent label spam.

---

## Key Differences from 1-Minute Variant

| Aspect | 1-Minute | 5-Minute | Why |
|--------|----------|----------|-----|
| EMA Lengths | 8/13/21 | 9/15/21 | 5-min bars need slightly wider windows; 9/15/21 is near-Fibonacci and captures 45-105 min swings |
| RSI Length | 14 | 10 | 5-min bars are smoother; RSI(14) is too sluggish. RSI(10) = 50 min lookback |
| RSI OB/OS | 70/30 | 65/35 | 5-min RSI rarely hits extremes intraday; tighter thresholds catch exhaustion earlier |
| ADX No-Trend | 15 | 18 | ADX reads higher on smoother bars; 18 provides a meaningful no-trend gate |
| Opening Range | 5 min (5 bars) | 15 min (3 bars) | 1 bar is useless as a range; 15-min is the institutional standard |
| Signal Cooldown | 5 bars (5 min) | 3 bars (15 min) | Balances spam prevention with ~78 bars/session |
| Signal Window Start | 09:35 | 09:45 | First 3 bars establish OR; signals start after OR completes |
| VWAP Ext Filter | 1.5 ATR | 2.0 ATR | 5-min ATR is larger absolute; wider multiplier avoids premature extension flags |
| Candle Patterns | 4 patterns | 6 patterns | Added inside bar breakout and 3-bar momentum, optimized thresholds |
| Strong Close Threshold | 50% of range | 55% of range | 5-min candles have proportionally larger wicks |
| Hammer/Star Threshold | 2.0x body | 1.8x body | 5-min patterns are slightly less extreme |
| Scoring Model | AND-gate (all must pass) | 5+4 with score threshold | Flexible scoring allows partial confluence; configurable min score |
| MTF Confirmation | N/A | 1-min RSI + EMA trend | Lower-TF context adds precision without chart clutter |
| Squeeze Integration | AND-gate filter (opt-in) | Scoring condition #8 | 5-min scores squeeze recency (fired within 3 bars = ~15 min) |
| TICK Integration | AND-gate filter (opt-in) | Scoring condition #9 | 5-min scores strong TICK readings (above/below thresholds) |
| Dashboard Rows | 16 | 18 | Added 1m Trend, 1m RSI, Squeeze, TICK rows |
| Level Proximity Scoring | Not in signal logic | Explicit condition (#7) | Near-support/resistance scored as enhancement condition |
| Line Extend Distance | 300 bars left | 60 bars left | 5-min sessions are ~78 bars; 300 would extend off-chart |

---

## Installation

1. Open TradingView and navigate to a **SPY 5-minute chart**.
2. Open the **Pine Editor** (bottom panel).
3. Delete any existing code in the editor.
4. Paste the entire contents of `spy_0dte_scalper_5min.pine` into the editor.
5. Click **"Add to chart"** (or "Save" then "Add to chart").
6. The indicator appears as an overlay with the dashboard in the top-right corner.

**Required chart settings**:

- **Extended hours**: Enable "Extended Trading Hours" in chart settings if you want Pre-Market High/Low levels to populate. Without extended hours data, those fields will show "N/A" in the dashboard and no PM level lines will render.
- **Timezone**: The script defaults to `America/New_York` and is configurable. Your TradingView chart timezone setting does not affect the indicator's session detection.

**Data requirements**:

- **NYSE TICK**: The TICK index (`USI:TICK`) requires your TradingView data plan to include the relevant exchange data. If the symbol is unavailable, the TICK dashboard row will show "N/A" and TICK scoring conditions will not contribute to the score (no blocking). You can disable the TICK feature entirely via the "Enable NYSE TICK Index" toggle.

**Companion usage**: this indicator can be used alongside the 1-minute variant. Run the 1-minute version on a 1-minute SPY chart in a separate tab/layout, and this 5-minute version on a 5-minute SPY chart. The 5-minute version already pulls 1-minute data internally via MTF, so running both is optional but provides visual context on the lower timeframe.

---

## Indicator Components

### EMA Ribbon

Three exponential moving averages that form a visual ribbon around price action.

| Parameter | Default | Description |
|-----------|---------|-------------|
| Fast EMA Length | 9 | 9 bars = 45 minutes. Fastest-reacting EMA. |
| Mid EMA Length | 15 | 15 bars = 75 minutes. Intermediate smoothing. |
| Slow EMA Length | 21 | 21 bars = 105 minutes. Trend anchor. |
| Show EMA Cloud Fill | true | Toggles the shaded fill between fast and slow EMAs. |

**Cloud behavior**:

- **Green fill**: all three EMAs are stacked bullish (fast > mid > slow).
- **Red fill**: all three EMAs are stacked bearish (fast < mid < slow).
- **Yellow fill**: mixed/transitional alignment.

On the 5-minute chart, EMA cloud flips represent ~45-105 minute trend shifts. A green cloud with price riding above the ribbon indicates a strong intraday uptrend structure that typically supports 15-30 minute swing-scalp holds.

### VWAP and Bands

The Volume Weighted Average Price resets each trading session and serves as the primary institutional reference level.

| Parameter | Default | Description |
|-----------|---------|-------------|
| Show VWAP | true | Toggle VWAP line visibility |
| Show VWAP Bands | true | Toggle standard deviation bands |
| VWAP Band Multiplier | 1.0 | Number of standard deviations for upper/lower bands |

**Band calculation**: uses cumulative volume-weighted variance from session open. The bands represent one standard deviation of price dispersion around VWAP. On a 5-minute chart, band width builds more gradually than on 1-minute since fewer bars contribute to the cumulative variance in the early session.

### RSI

Relative Strength Index. Calculated internally with a 10-period default (50 minutes on 5-min chart) and displayed in the dashboard only.

| Parameter | Default | Description |
|-----------|---------|-------------|
| RSI Length | 10 | Lookback period (10 bars = 50 minutes) |
| Overbought | 65 | Upper threshold (tighter than 1-min's 70) |
| Oversold | 35 | Lower threshold (tighter than 1-min's 30) |

**Why tighter thresholds**: on a 5-minute chart, RSI is inherently smoother because each bar aggregates more price action. RSI(10) on 5-min rarely reaches 70/30 intraday on SPY. Using 65/35 catches momentum exhaustion at levels that are actually meaningful on this timeframe. The signal engine requires RSI to be within the neutral zone for both CALLS and PUTS to fire.

### ADX / DMI

The Average Directional Index measures trend strength regardless of direction.

| Parameter | Default | Description |
|-----------|---------|-------------|
| ADX Length | 14 | Lookback for ADX/DI calculation (14 bars = 70 minutes) |
| No Trend Threshold | 18 | Below this = NO TRADE / RANGING regime |
| Trending Threshold | 25 | Above this = confirmed trend |
| Strong Trend Threshold | 40 | Above this = strong directional move |

**ADX status mapping**:

| ADX Value | Status | Color | Implication |
|-----------|--------|-------|-------------|
| < 18 | NO TREND | Red | Chop. Avoid trading. Theta decay dominates. |
| 18 - 24.9 | WEAK/RANGING | Yellow | Directional bias forming but not confirmed. |
| 25 - 39.9 | TRENDING | Green | Confirmed trend. Signals are higher conviction. |
| >= 40 | STRONG TREND | Green | Strong directional move. Trend-following entries preferred. |

The raised no-trend threshold (18 vs. 15 on 1-min) reflects that ADX on 5-minute bars reads higher during mild ranging conditions. An ADX of 15 on 5-min often represents genuine chop that would destroy 0DTE positions.

### ATR

Average True Range measures realized volatility on the 5-minute timeframe.

| Parameter | Default | Description |
|-----------|---------|-------------|
| ATR Length | 14 | Lookback period (14 bars = 70 minutes) |

ATR on 5-minute bars is approximately 2.5-4x larger than 1-minute ATR in absolute terms. This is expected (more price movement in 5 minutes). The ATR value is used for VWAP extension filtering, level proximity detection, and is displayed in the dashboard for option strike selection context.

### TTM Squeeze

The TTM Squeeze detects periods of volatility compression by comparing Bollinger Bands to Keltner Channels. When Bollinger Bands contract inside the Keltner Channels, a "squeeze" is active — volatility is compressing and a directional breakout is likely imminent. When the squeeze releases (BB expand outside KC), energy stored during compression typically drives a strong directional move.

| Parameter | Default | Description |
|-----------|---------|-------------|
| Enable TTM Squeeze Detection | true | Master toggle for squeeze calculations and dashboard display |
| BB Length | 20 | Bollinger Band SMA lookback (20 bars = 100 minutes on 5-min) |
| BB Multiplier | 2.0 | Standard deviations for Bollinger Bands |
| KC Length | 20 | Keltner Channel EMA lookback |
| KC Multiplier | 1.5 | ATR multiplier for Keltner Channels |

**Squeeze states**:

| State | Condition | Dashboard Color | Implication |
|-------|-----------|----------------|-------------|
| **SQUEEZE ON** | BB upper < KC upper AND BB lower > KC lower | Orange | Compression active. Volatility is building. A breakout is forming. |
| **FIRED** | Squeeze was ON on previous bar, now OFF | Green | Volatility expansion just began. Highest-probability moment for directional entry. |
| **OFF** | BB outside KC (normal volatility) | Gray/Green | Normal conditions. If recently fired (within 3 bars / ~15 min), shown in green with recency context. |

**Momentum histogram**: a linear regression of the midline delta between BB and KC basis is calculated internally. When the squeeze fires, this histogram's direction (positive = bullish, negative = bearish) indicates the likely breakout direction. This is included in the squeeze fired alert message.

**Signal integration (scoring model)**: on the 5-minute variant, squeeze is integrated as **enhancement condition #8** in the scoring system. The condition scores +1 when the squeeze has just FIRED or fired within the last 3 bars (~15 minutes). This rewards signals that occur during or immediately after volatility expansion — the highest-probability window for directional moves. Unlike the 1-minute variant's AND-gate filter, the scoring model means a missing squeeze condition does not block signals; it simply means one fewer point toward the minimum score threshold.

**Why the 3-bar recency window**: on a 5-minute chart, a squeeze fire represents a ~100-minute compression cycle releasing. The directional energy from that release typically persists for 3-6 bars (15-30 minutes). The 3-bar window captures the initial expansion phase where signal quality is highest.

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

**Signal integration (scoring model)**: on the 5-minute variant, TICK is integrated as **enhancement condition #9** in the scoring system. For CALLS, it scores +1 when `TICK > i_tickBull` (+500 default). For PUTS, it scores +1 when `TICK < i_tickBear` (-500 default). This is stricter than the 1-minute variant's filter (which only requires `TICK > 0`); the 5-minute scoring model rewards strong breadth confirmation rather than just directional agreement.

**Why it matters for 0DTE**: SPY is a basket of ~500 stocks. A SPY CALLS entry when the TICK is deeply negative means you're fighting broad market selling pressure — even if SPY's chart looks bullish, the underlying components are being sold. Strong TICK confirmation improves signal reliability by confirming that the directional bet is supported by broad market participation.

### Multi-Timeframe Confirmation

The 5-minute variant's primary enhancement over the 1-minute version. Pulls 1-minute data via `request.security()` to provide lower-timeframe context without requiring a separate chart.

| Parameter | Default | Description |
|-----------|---------|-------------|
| Enable 1-Min MTF Confirmation | true | Master toggle for all MTF features |
| 1-Min RSI Length | 14 | RSI lookback on the 1-minute timeframe |
| 1-Min Fast EMA Length | 8 | Fast EMA on 1-minute (matches 1-min variant default) |
| 1-Min Slow EMA Length | 21 | Slow EMA on 1-minute (matches 1-min variant default) |

**What gets pulled from the 1-minute chart**:

| Data | Usage |
|------|-------|
| 1-Min RSI | Dashboard display + awareness. When 5-min RSI looks neutral but 1-min RSI is already OB/OS, momentum may be exhausted on the lower timeframe. |
| 1-Min EMA Trend (fast > slow?) | Signal scoring condition #6. When 1-min trend aligns with 5-min direction, the signal scores an extra point. |

**Signal integration**: when MTF is enabled, the signal engine includes condition #6: "1-min EMA trend confirms the 5-min signal direction." This is additive — it improves signal quality when present but does not gate signals when absent (unless `i_minScore` is set high enough to require it).

**Dashboard integration**: two dedicated rows (1m Trend, 1m RSI) appear in the dashboard. When MTF is disabled, these rows show "OFF" in gray.

**Performance note**: all three 1-minute values (RSI, EMA fast, EMA slow) are pulled in a single consolidated `request.security()` tuple, requiring only one pass through the 1-minute bar data.

### Key Price Levels

Horizontal reference levels drawn on the chart. Each has a dashed or dotted line style and a right-edge label.

#### Pre-Market High / Low

- **Detection**: tracks the highest high and lowest low during the pre-market session (4:00 AM - 9:29 AM ET).
- **Rendering**: dashed horizontal lines drawn once RTH begins, extending 60 bars left (~5 hours on 5-min chart) and 5 bars right.
- **Requirement**: Extended Trading Hours must be enabled in chart settings.

#### Prior Day High / Low / Close

- **Source**: daily timeframe via `request.security()` using the prior completed day.
- **Rendering**: dotted horizontal lines with right-edge labels (PDH, PDL, PDC).
- **No repainting risk**: the `[1]` offset with `lookahead_on` accesses only the prior completed daily bar.

#### Opening Range

- **Detection**: captures the high and low of the first N minutes of RTH. Default is 15 minutes (3 bars on 5-min chart).
- **Bar count**: `orBarsNeeded = math.ceil(i_orMinutes / 5)`. The input is in minutes and automatically converts to the appropriate number of 5-minute bars.
- **Rendering**: dashed horizontal lines with right-edge labels (OR HIGH, OR LOW) drawn once the opening range period completes.
- **Usage**: the 15-minute opening range is an institutional standard. Breakout above OR HIGH with volume on the 5-min chart is a high-probability continuation signal. Failure to break the OR in the first hour often leads to ranging conditions.

#### Session HOD / LOD

- **Detection**: running high-of-day and low-of-day from RTH open, updated on every bar.
- **Rendering**: solid horizontal lines with right-edge labels that dynamically update.
- **Reset**: clears at each RTH open.

### Regime / Mode Detection

A composite classifier combining ADX trend strength, EMA ribbon alignment, and price position relative to VWAP.

| Mode | Conditions | Implication |
|------|-----------|-------------|
| **BULLISH** | ADX >= 18, EMAs stacked bullish, price above VWAP | Favor CALLS entries |
| **BEARISH** | ADX >= 18, EMAs stacked bearish, price below VWAP | Favor PUTS entries |
| **RANGING** | ADX 18-25 or mixed EMA/VWAP | Caution; both directions possible |
| **NO TRADE** | ADX < 18 and price near VWAP | Avoid. Low-conviction chop. |
| **TRANSITION** | ADX >= 18 but EMA/VWAP not aligned | Emerging trend; use caution |

On the 5-minute chart, mode transitions are more significant than on 1-minute. A mode flip from RANGING to BULLISH on 5-min represents a meaningful ~1-hour structural shift. Mode changes trigger alerts.

### Signal Engine

The core decision engine that generates CALLS and PUTS labels on the chart.

#### Scoring System (5+4 Model)

Each signal direction is evaluated using 5 core conditions and 4 optional enhancement conditions. Each true condition adds 1 point to the score. A signal fires when the score meets or exceeds the configured minimum threshold.

**CALLS conditions (bullish)**:

| # | Type | Condition | Rationale |
|---|------|-----------|-----------|
| 1 | Core | Price above VWAP or crossing above | Institutional bias is bullish |
| 2 | Core | EMA ribbon bullish or price reclaiming slow EMA | Trend structure supports upside |
| 3 | Core | RSI between 35-65 (neutral zone) | Momentum has room to run |
| 4 | Core | Bullish candle pattern detected | Price action confirmation |
| 5 | Core | ADX above no-trend threshold (18) | Measurable directional movement |
| 6 | Enhancement | 1-min EMA trend is bullish (MTF) | Lower-TF trend confirms direction |
| 7 | Enhancement | Price near support level (within 1.0 ATR) | Favorable risk/reward at structure |
| 8 | Enhancement | TTM Squeeze fired or fired within last 3 bars | Volatility expanding — directional energy releasing |
| 9 | Enhancement | NYSE TICK > bullish threshold (+500) | Broad market breadth confirms buying pressure |

**Candle patterns that satisfy condition #4**:

- Bullish engulfing (close > prior high, open <= prior low)
- Strong bull close (body > 55% of range)
- Hammer (lower wick > 1.8x body)
- Inside bar breakout to the upside (prior bar inside, current breaks above)
- 3-bar bullish momentum (3 consecutive higher closes)

**PUTS conditions**: mirror of CALLS with inverted direction. Uses bearish engulfing, strong bear close, shooting star, inside bar breakdown, and 3-bar bearish momentum. TICK condition requires TICK < bearish threshold (-500).

#### Minimum Score

| `i_minScore` | Typical Signals/Session | Behavior |
|---|---|---|
| 3 | 10-20+ | Very aggressive. Most core conditions don't need to align. High false positive rate. |
| 4 (default) | 5-12 | Balanced. Requires 4 of 5 core conditions, or 3 core + 1-2 enhancements. |
| 5 | 2-6 | Conservative. Requires all 5 core, or 4 core + 1 enhancement. |
| 6 | 0-3 | Strict. Requires 5 core + 1 enhancement. Very few signals. |
| 7 | 0-2 | Very selective. Requires 5 core + 2 enhancements. |
| 8 | 0-1 | Near-sniper. Requires 5 core + 3 enhancements. Very rare signals. |
| 9 | 0-1 | Sniper. All 9 conditions must be true. May produce zero signals on most sessions. |

**Recommendation**: start at 4. If too many signals are noise, increase to 5. If you're missing moves, ensure MTF and squeeze/TICK are enabled (adds scoring opportunities) before dropping to 3. Scores of 6+ are meaningful only when multiple enhancement features are active.

#### Signal Filters

Filters must pass before scoring is evaluated:

| Filter | Description |
|--------|-------------|
| **Time Window** | Default 09:45-15:45 ET. Starts after OR completes. Ends 15 min before close. |
| **Cooldown** | 3 bars (15 min) minimum between same-direction signals. |
| **Bar Confirmation** | Signals fire only on confirmed (closed) bars. No phantom signals. |
| **ADX Gate** | When enabled, ADX must be above no-trend threshold. |
| **VWAP Extension** | Signals suppressed when price is > 2.0 ATR from VWAP (overextended). |

Note: unlike the 1-minute variant, squeeze and TICK are integrated as scoring conditions (not filters) on the 5-minute variant. They contribute to the score but do not independently block signals.

#### Visual Output

When a signal fires:

1. **Label**: "CALLS" (green, below bar) or "PUTS" (red, above bar) with a hover tooltip showing score, RSI, ADX, VWAP distance, mode, ATR, EMA status, MTF data, squeeze state, and TICK value.
2. **Arrow**: small triangle shape as a secondary quick-scan marker.
3. **Background flash**: faint green or red background highlight on the signal bar.

#### VWAP Cross Markers

Diamond shapes on VWAP crosses during RTH:

- **Lime diamond below bar**: bullish VWAP cross (price crossed above).
- **Red diamond above bar**: bearish VWAP cross (price crossed below).

On a 5-minute chart, VWAP crosses are less frequent and more significant than on 1-minute. Each cross represents a 5-minute candle definitively closing above or below the institutional fair value level.

### Dashboard

A real-time information table overlaid in the top-right corner.

#### Layout

Three-column table with a header row and 17 data rows (18 total). Dark navy background with magenta header bar. Color-coded values for instant status reads.

#### Fields

| Row | Field | Value | Color Logic |
|-----|-------|-------|-------------|
| Header | SPY 0DTE Scalper - 5m | — | Purple accent bar |
| 1 | Price | Current close + bar direction | Green = green bar, Red = red bar |
| 2 | Mode | Regime classification + advice | Green/Red/Yellow/Gray per mode |
| 3 | RSI (10) | RSI value + status | Red = OB, Green = OS, White = neutral |
| 4 | ADX | ADX value + trend status | Red/Yellow/Green per strength |
| 5 | VWAP Dist | Signed distance + status | Green = near, Yellow = moderate, Red = extended |
| 6 | PM High | Distance to PM High + status | Red = resistance, Green = cleared |
| 7 | PM Low | Distance to PM Low + status | Green = support, Red = broken |
| 8 | ATR (14) | Current ATR value | Neutral (informational) |
| 9 | EMA Trend | 5-min ribbon alignment | Green = bullish, Red = bearish, Yellow = mixed |
| 10 | Signal | Current bar signal + score | Green = CALLS + score/9, Red = PUTS + score/9, Gray = none |
| 11 | Day Chg | % change from session open | Green = positive, Red = negative |
| 12 | Rel Vol | Volume / 20-bar avg | Green + "SPIKE" if > 1.5x |
| 13 | DI Spread | +DI / -DI values | Green = bulls, Red = bears |
| 14 | 1m Trend | 1-min EMA alignment (MTF) | Green/Red/Yellow; "OFF" if MTF disabled |
| 15 | 1m RSI | 1-min RSI value + status (MTF) | Same color logic as 5-min RSI row |
| 16 | Squeeze | TTM Squeeze state + context | Orange = ON/BUILDING, Green = FIRED/BREAKOUT or recent (with bars-ago context), Gray = OFF/NORMAL |
| 17 | TICK | NYSE TICK value + breadth tier | Green = bullish/extreme bull, Red = bearish/extreme bear, Neutral = mild, Gray = N/A or OFF |

When TTM Squeeze, NYSE TICK, or MTF features are disabled, their respective rows display "OFF" in gray. The table layout remains stable regardless of feature state. When TICK data is unavailable (pre-market, after-hours, or data plan limitation), the row displays "N/A".

### Alerts

#### Static Alerts (alertcondition)

| Alert Name | Trigger |
|------------|---------|
| 0DTE-5m: CALLS Signal | CALLS signal fires |
| 0DTE-5m: PUTS Signal | PUTS signal fires |
| 0DTE-5m: VWAP Cross Bullish | Price crosses above VWAP |
| 0DTE-5m: VWAP Cross Bearish | Price crosses below VWAP |
| 0DTE-5m: Regime Mode Change | Mode transitions to a different state |
| 0DTE-5m: Squeeze Fired | TTM Squeeze transitions from ON to OFF (volatility expansion) |

#### Dynamic Alerts (alert)

Rich, interpolated messages including score, MTF, squeeze, and TICK status. Examples:

```
0DTE-5m CALLS | SPY 586.42 | Score 6/9 | RSI 52.3 | ADX 27.8 | VWAP Dist 0.45 | Mode: BULLISH | 1m: BULLISH | Sqz: OFF | TICK: 623
```

```
0DTE-5m SQUEEZE FIRED | SPY 586.80 | Momentum: BULLISH | ADX 22.5 | Mode: TRANSITION
```

```
0DTE-5m MODE SHIFT | RANGING -> BULLISH | SPY 587.10
```

Dynamic alerts use `alert.freq_once_per_bar_close` for signals and squeeze fires (no duplicates within a bar) and `alert.freq_once_per_bar` for mode changes. Squeeze and TICK context is appended to signal alerts only when the respective feature is enabled.

---

## Visual Reference

### Color Map

All colors are selected for maximum contrast against TradingView's dark theme.

| Element | Hex | Notes |
|---------|-----|-------|
| VWAP | `#FFFFFF` | Solid white, weight 2 |
| VWAP Bands | `#00BCD4` | Cyan, circle style |
| EMA 9 (Fast) | `#FFD600` | Bright yellow |
| EMA 15 (Mid) | `#FF9100` | Bright orange |
| EMA 21 (Slow) | `#FF1744` | Hot pink/red |
| EMA Cloud (Bull) | `#00E676` | Neon green, 85% transparent |
| EMA Cloud (Bear) | `#FF1744` | Neon red, 85% transparent |
| EMA Cloud (Mixed) | `#FFD600` | Yellow, 90% transparent |
| PM High | `#FF5252` | Bright red, dashed |
| PM Low | `#69F0AE` | Bright green, dashed |
| Prior Day High | `#FF6E40` | Coral, dotted |
| Prior Day Low | `#64FFDA` | Teal, dotted |
| Prior Day Close | `#B0BEC5` | Gray, dotted |
| OR High | `#E040FB` | Magenta, dashed |
| OR Low | `#B388FF` | Purple, dashed |
| CALLS Signal | `#00E676` | Neon green label + arrow + bg |
| PUTS Signal | `#FF1744` | Neon red label + arrow + bg |
| VWAP Cross (Bull) | `#76FF03` | Lime diamond |
| VWAP Cross (Bear) | `#FF1744` | Red diamond |
| Squeeze ON | `#FF9100` | Orange, dashboard status |
| Squeeze FIRED | `#00E676` | Green, dashboard status |

### Chart Element Legend

```
EMA 9 (Fast)  |  EMA 15 (Mid)  |  EMA 21 (Slow)  |  VWAP  |  VWAP +1σ  |  VWAP -1σ
CALLS Arrow  |  PUTS Arrow  |  VWAP Cross Bull  |  VWAP Cross Bear  |  Signal Background Flash
```

Key levels (PM, PD, OR, HOD/LOD) are drawn with `line.new()` and identified by right-edge text labels.

### Dashboard Fields

See [Dashboard](#dashboard) for the full 18-field table. The dashboard adapts to MTF/squeeze/TICK on/off state — when features are disabled, their rows show "OFF" in gray rather than disappearing, so the table layout remains stable.

---

## Tuning Guide

All parameters are exposed as configurable inputs. This section covers tuning for the 5-minute timeframe specifically.

### Minimum Signal Score

**Input**: Minimum Signal Score
**Default**: 4
**Range**: 2-9

This is the most impactful setting. It controls how many of the 9 possible conditions must be true for a signal to fire.

| Score | Signals/Session | Best For |
|-------|-----------------|----------|
| 3 | 10-20+ | Aggressive scalpers. High noise. |
| 4 (default) | 5-12 | Balanced. ~4 core conditions must align. |
| 5 | 2-6 | Conservative. Requires near-full core alignment or core + enhancements. |
| 6 | 0-3 | Very selective. Near-perfect core + enhancement confluence. |
| 7 | 0-2 | Strict. Requires 5 core + 2 of 4 enhancements. |
| 8-9 | 0-1 | Sniper. Practically requires all features enabled and all conditions true. |

**Recommendation**: start at 4. If too many signals are noise, increase to 5. If you're missing moves, ensure MTF, squeeze, and TICK are all enabled (maximizes scoring opportunities) before dropping to 3. Scores of 7+ only make sense with all 4 enhancement features active (MTF, level proximity, squeeze, TICK).

### EMA Lengths

**Defaults**: 9, 15, 21

| Adjustment | Effect |
|------------|--------|
| Shorter (5/9/15) | More responsive. Cloud flips represent ~25-75 min shifts. Better for aggressive scalps. |
| Default (9/15/21) | Balanced. Cloud represents ~45-105 min trend windows. |
| Longer (13/21/34) | Smoother. Cloud flips are rare but high-conviction. Better for swing-scalp holds of 15-30 min. |

### ADX Thresholds

**Default No-Trend**: 18

| Threshold | Behavior |
|-----------|----------|
| 14-16 | Permissive. Allows signals during mild chop. Riskier for 0DTE. |
| 18 (default) | Balanced. Filters out most low-conviction environments. |
| 20-22 | Strict. Only allows signals during clear trends. Reduces midday signal count significantly. |

### VWAP Extended Distance

**Input**: Max VWAP Distance (ATR multiples)
**Default**: 2.0

Signals are suppressed when price is more than this many ATR units from VWAP (overextended).

| Value | Behavior |
|-------|----------|
| 1.5 | Conservative. Suppresses earlier. Prevents chasing. |
| 2.0 (default) | Balanced for 5-min ATR magnitudes. |
| 3.0 | Permissive. Allows signals during strong trend runs where VWAP divergence is sustained. |

### Signal Cooldown

**Input**: Signal Cooldown (bars)
**Default**: 3 (= 15 minutes)

| Value | Behavior |
|-------|----------|
| 1-2 (5-10 min) | Signals can fire frequently during sustained moves. Risk of label clusters. |
| 3 (default, 15 min) | One signal per 15-minute window per direction. Balanced for ~78 bars/session. |
| 4-6 (20-30 min) | Very selective. Each signal is well-separated. Good for single-position traders. |

### Signal Time Window

**Default**: 09:45 - 15:45 ET

| Adjustment | Rationale |
|------------|-----------|
| Start 09:35 | Include the OR formation period. High reward but high noise. Only for experienced traders. |
| Start 09:45 (default) | Starts after the 15-min opening range completes. Recommended. |
| Start 10:00 | Wait for the first 30 minutes to establish structure. Very conservative. |
| End 15:45 (default) | Stops 15 min before close. Avoids gamma/pin risk. |
| End 15:30 | Stops 30 min early. Extra-conservative for 0DTE. |
| End 15:55 | Signals nearly to close. Only if you understand pin risk and rapid gamma decay. |

### Opening Range Duration

**Input**: Opening Range Minutes
**Default**: 15

On the 5-minute chart, the input is automatically converted to bars (`ceil(minutes / 5)`).

| Value | Bars | Usage |
|-------|------|-------|
| 5 | 1 | Single bar. Too narrow for meaningful range. Not recommended on 5-min. |
| 15 (default) | 3 | 15-minute OR. Institutional standard. Breakouts are high-probability. |
| 30 | 6 | 30-minute range. Very wide. Fewer breakout opportunities but highest conviction. |

### RSI Thresholds

**Defaults**: 65 (OB) / 35 (OS)

| Adjustment | Effect |
|------------|--------|
| Tighter (60/40) | More aggressive filtering. Signals only fire when RSI has significant room. Fewer signals. |
| Default (65/35) | Balanced for 5-min RSI dynamics. |
| Wider (70/30) | Matches 1-min defaults. Allows signals closer to extremes. More signals but some at exhaustion points. |

### TTM Squeeze Settings

**Inputs**: BB Length, BB Multiplier, KC Length, KC Multiplier
**Defaults**: 20, 2.0, 20, 1.5

The default BB(20, 2.0) vs KC(20, 1.5) is the standard TTM Squeeze configuration. On a 5-minute chart, BB(20) represents a 100-minute lookback for the compression detection.

| Adjustment | Effect |
|------------|--------|
| Lower KC Multiplier (1.0-1.25) | Squeezes trigger more frequently. More scoring opportunities but each may be less significant. |
| Higher KC Multiplier (1.75-2.0) | Squeezes are rarer and represent more extreme compression. Higher conviction when they contribute to signal scores. |
| Shorter lengths (10-15) | More responsive. Faster squeeze cycles. Better for capturing shorter compression periods. |
| Longer lengths (25-30) | Smoother. Squeezes represent more sustained compression. Fewer false squeeze events. |

**Scoring integration**: condition #8 scores +1 when `squeezeFired OR sqzBarsSinceFire <= 3`. The 3-bar recency window (~15 minutes) captures the initial volatility expansion phase. To increase the recency window, this would require a code modification.

### NYSE TICK Settings

**Inputs**: TICK Bullish/Bearish/Extreme Thresholds
**Defaults**: +500 / -500 / 800

These thresholds control both the dashboard status tiers and the scoring condition requirements.

| Adjustment | Effect |
|------------|--------|
| Tighter bullish/bearish (300/-300) | TICK scoring condition fires more easily. More signals score the breadth point. |
| Wider bullish/bearish (700/-700) | Requires stronger breadth for the score point. Fewer signals benefit from TICK confirmation. |
| Higher extreme (1000) | Fewer EXTREME BULL/BEAR dashboard classifications. |

**Scoring integration**: condition #9 requires `TICK > i_tickBull` for CALLS and `TICK < i_tickBear` for PUTS. Unlike the 1-minute variant's light filter (TICK > 0), the 5-minute scoring model demands a meaningful breadth reading before awarding the point. This is appropriate because the 5-minute chart already filters for higher-quality setups, and breadth should be convincing to add value.

**Symbol selection**: `USI:TICK` covers NYSE-listed stocks. `USI:TICK.NQ` covers NASDAQ-listed stocks. For SPY trading, `USI:TICK` is the standard choice since SPY's components are primarily NYSE-listed.

### Multi-Timeframe Toggle

**Input**: Enable 1-Min MTF Confirmation
**Default**: true

When enabled, adds 1-min RSI and EMA trend to the dashboard and makes condition #6 (MTF trend alignment) available for scoring.

| Setting | Effect |
|---------|--------|
| ON (default) | Full MTF layer. Condition #6 available. Dashboard shows 1m data. One `request.security()` tuple call active. |
| OFF | No MTF data. Conditions #1-5, #7-9 only. Dashboard shows "OFF" for MTF rows. Slightly faster load. |

**When to disable**: if you're running the 1-minute variant simultaneously on a separate chart and don't want redundant MTF computation. Or if you prefer a simpler signal model.

### Tuning Profiles

#### Profile: Conservative / Swing-Scalp

For traders targeting 1-3 high-conviction trades per session with 15-30 minute holds.

```
EMA Lengths:           13 / 21 / 34
ADX No Trend:          22
Cooldown:              4 bars (20 min)
Signal Start:          0945
Signal End:            1530
RSI OB/OS:             60 / 40
MTF Confirmation:      ON
Squeeze Detection:     ON
TICK Index:            ON
Min Score:             6
```

#### Profile: Balanced (Default)

For traders targeting 5-12 signals per session with moderate conviction.

```
EMA Lengths:           9 / 15 / 21
ADX No Trend:          18
Cooldown:              3 bars (15 min)
Signal Start:          0945
Signal End:            1545
RSI OB/OS:             65 / 35
MTF Confirmation:      ON
Squeeze Detection:     ON
TICK Index:            ON
Min Score:             4
```

#### Profile: Aggressive / Scalper

For experienced traders who want frequent signals and manage risk through position sizing and rapid exits.

```
EMA Lengths:           5 / 9 / 15
ADX No Trend:          14
Cooldown:              2 bars (10 min)
Signal Start:          0935
Signal End:            1555
RSI OB/OS:             70 / 30
MTF Confirmation:      OFF
Squeeze Detection:     OFF
TICK Index:            OFF
Min Score:             3
```

---

## Architecture Notes

**Performance**: the script uses `var` declarations for persistent state (lines, labels, level values, cooldown counters) to avoid reallocation on every bar. TTM Squeeze calculations (BB, KC, linear regression) use native `ta.*` functions and add negligible overhead. The dashboard table is updated only on `barstate.islast`.

**`request.*()` budget**: the script makes 4 `request.*()` calls total, consolidated via tuple returns (v1.0.1): 1 tuple on `"1"` for MTF (1-min RSI, EMA fast, EMA slow), 1 for NYSE TICK on `"1"` (separate symbol, cannot merge with MTF tuple), 1 tuple on `"D"` (prior day high, low, close, current day open), and 1 for `request.footprint()` (volume footprint data). This is well within TradingView's limit of 40 calls per script. Disabling MTF removes 1 call; disabling TICK removes 1; disabling footprint removes 1.

**Repainting**: all signal logic is gated by `barstate.isconfirmed`. Signals appear after bar close only. MTF data from `request.security()` uses `lookahead=barmerge.lookahead_off` (no forward-looking data) for 1-minute pulls, ensuring no repainting from lower-timeframe data.

**Object limits**: Pine Script v6 allows 500 labels, 500 lines, 200 boxes. On a 5-minute chart, a full RTH session is ~78 bars. Even with signals on every bar (impossible given cooldowns), object limits are never approached. Line extend distances are set to 60 bars (appropriate for 5-min session length) rather than 300 (which would extend far off-chart).

**Opening range bar math**: `orBarsNeeded = math.ceil(i_orMinutes / 5)` converts the minute-based input to 5-minute bars. For the default 15 minutes, this yields 3 bars. Non-multiples of 5 round up (e.g., 12 minutes = 3 bars = 15 minutes effective).

**VWAP bands**: cumulative variance resets with the session. On the first bar, bands overlap VWAP. This resolves within 1-2 bars on the 5-minute chart.

**Timezone**: session detection uses the configurable `i_tz` input, defaulting to `America/New_York`.

**Squeeze momentum**: the linear regression histogram (`ta.linreg`) is calculated but not plotted on the chart (overlay indicator). The momentum value and direction are included in squeeze fired alerts and are available for future enhancements.

---

## Known Limitations

1. **Footprint data repaints by design**: Volume footprint values (delta, POC, buy/sell volume) may differ between real-time and historical views due to TradingView's intrabar data resolution hierarchy. When footprint delta scoring (#10) is enabled, a signal that fired in real-time could shift by +-1 point on chart reload. This is why scoring is OFF by default. Requires TradingView Premium or Ultimate plan; returns N/A on lower plans.

2. **Footprint POC is approximate**: the POC step-line plots the midpoint of the highest-volume price row. With the default 25 ticks/row ($0.25 per row on SPY), the plotted POC may differ from the exact highest-volume tick by up to $0.125.

3. **Pre-market levels require extended hours**: if your chart does not have Extended Trading Hours enabled, PM High/Low will not populate.

4. **Single-symbol calibration**: defaults are calibrated for SPY's price range and volatility. Other instruments require parameter adjustment.

5. **Equal-weighted scoring**: all 10 conditions contribute equally to the score. Some conditions (like VWAP position) may be more predictive than others. The 15-minute variant's weighted scoring model partially addresses this by separating structural and confirmation tiers.

6. **No backtest capability**: this is an `indicator()`, not a `strategy()`. It cannot be backtested through TradingView's strategy tester.

7. **MTF request.security() limitations**: the 1-minute data pulled by `request.security()` represents the close of the last 1-minute bar, not a real-time tick. Within a forming 5-minute bar, the 1-min data updates every minute (when a new 1-min bar closes), not continuously. This is a TradingView platform limitation.

8. **Opening range granularity**: the OR input is in minutes but the effective range is always rounded up to the nearest 5-minute bar. A 7-minute OR input results in a 10-minute (2-bar) effective OR.

9. **TICK data availability**: the NYSE TICK symbol (`USI:TICK`) requires your TradingView data plan to include the relevant exchange. If unavailable, the TICK row shows "N/A" and the TICK scoring condition does not contribute to the score (it will never award a point, but will not block signals). TICK data is only available during RTH (9:30 AM - 4:00 PM ET).

10. **Squeeze recency window is hardcoded**: the 3-bar recency window for squeeze scoring (condition #8) is defined in the source code, not as a configurable input. Modifying the window requires editing the Pine Script source (`sqzBarsSinceFire <= 3`).
