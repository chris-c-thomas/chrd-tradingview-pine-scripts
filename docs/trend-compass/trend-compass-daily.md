# Trend Compass - Daily

Strategic trend assessment overlay for equities, ETFs, and indices.

---

## Design Philosophy

This indicator answers the questions your 0DTE scalpers and Market Monitors don't: **What is the multi-day/multi-week trend? How healthy is it? Is it accelerating, maturing, or showing signs of exhaustion?**

It is purpose-built for a fundamentally different use case than the existing intraday suite:

- **No signal generation.** No CALLS/PUTS labels. This is a context and assessment tool.
- **Ticker-agnostic.** Works on NVDA, AAPL, GOOGL, SPY, QQQ, IWM, or any liquid instrument. No hardcoded symbols or session-specific logic.
- **Multi-timeframe internally.** Pulls Weekly context via `request.security()` so you don't need separate charts for the macro view.
- **Trend phase classification.** Six distinct states that map the lifecycle of a trend from birth through exhaustion.
- **Divergence detection.** RSI and MACD divergences with visual line segments on the chart — the primary early warning system for trend reversals.
- **Optional modules.** Ichimoku Cloud and Fibonacci retracement are available as toggles for traders who use them, without cluttering the chart for those who don't.

---

## Table of Contents

- [Features](#features)
- [Role in the Indicator Suite](#role-in-the-indicator-suite)
- [Installation](#installation)
- [Indicator Components](#indicator-components)
  - [EMA Ribbon (10/21/50)](#ema-ribbon-102150)
  - [Anchor EMA (200)](#anchor-ema-200)
  - [HTF Context (Weekly 50 EMA)](#htf-context-weekly-50-ema)
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
  - [EMA Lengths](#ema-lengths)
  - [Anchor EMA Length](#anchor-ema-length)
  - [HTF Timeframe Selection](#htf-timeframe-selection)
  - [ADX Slope Sensitivity](#adx-slope-sensitivity)
  - [RSI / MACD Divergence Pivot Bars](#rsi--macd-divergence-pivot-bars)
  - [ATR Percentile Lookback](#atr-percentile-lookback)
  - [BB Width Percentile Lookback](#bb-width-percentile-lookback)
  - [Fibonacci Lookback](#fibonacci-lookback)
  - [Ichimoku Parameters](#ichimoku-parameters)
  - [Scoring Mode](#scoring-mode)
- [Architecture Notes](#architecture-notes)
- [Known Limitations](#known-limitations)
- [Changelog](#changelog)

---

## Features

- **EMA Ribbon** (10/21/50) with dynamic cloud fill that shifts color based on trend alignment (bullish, bearish, mixed).
- **200 EMA Anchor** plotted as a thick structural reference line — the institutional dividing line between bullish and bearish structure.
- **Weekly 50 EMA** pulled via `request.security()` for macro trend confirmation, plotted as a step line overlay.
- **Trend Phase Classifier** — six states: EMERGING, ACCELERATING, MATURE, EXHAUSTING, CONSOLIDATING, REVERSING. Maps the lifecycle of a trend.
- **Composite Weighted Trend Score** (-11 to +11) from 8 conditions across structural and confirmation tiers.
- **Score Trend Tracking** (IMPROVING / STABLE / FADING) via 5-bar SMA comparison.
- **ADX Slope Analysis** — not just "is there a trend" but "is the trend gaining or losing strength."
- **50 EMA Slope + Acceleration** — quantifies the angle and curvature of the primary trend line.
- **RSI Divergence Detection** — bullish divergence, bearish divergence, hidden bull, hidden bear. Visual line segments on the chart connecting divergent pivots.
- **MACD Divergence Detection** — same four-state detection on the MACD histogram.
- **OBV Trend Confirmation** — confirms whether volume supports or contradicts the price trend.
- **Relative Volume** (20-bar SMA baseline) with SPIKE / ABOVE / NORMAL / DRY classification.
- **ATR Percentile Ranking** — LOW / NORMAL / HIGH / EXTREME regime classification.
- **Bollinger Band Width Percentile** — compression detection for breakout anticipation.
- **52-Week High / Low** levels with position-in-range percentage.
- **Prior Week High / Low / Close** reference levels.
- **Prior Month High / Low / Close** reference levels.
- **Fibonacci Retracement** (optional, default OFF) — auto-calculated 38.2%, 50%, 61.8% levels from the most recent significant range.
- **Ichimoku Cloud** (optional, default OFF) — traditional Ichimoku Kinko Hyo overlay with Tenkan, Kijun, Senkou A/B, and Chikou Span.
- **17-row heads-up dashboard** with color-coded status indicators.
- **12 alert conditions** covering phase changes, score thresholds, divergences, golden/death crosses, volatility shifts, and compression breakout watches.

---

## Role in the Indicator Suite

| Indicator | Primary TF | Focus | Signal Type |
|-----------|-----------|-------|-------------|
| 0DTE Scalper 1m | 1-min | Micro-entry timing | CALLS/PUTS labels |
| 0DTE Scalper 5m | 5-min | Swing-scalp trades | CALLS/PUTS labels |
| 0DTE Scalper 15m | 15-min | Directional bias + swings | CALLS/PUTS labels |
| Market Monitor 1m | Any intraday | Multi-chart watchlist bias | Bias score only |
| Market Monitor 5m | 5-min | Enhanced watchlist bias | Weighted bias score |
| **Trend Compass Daily** | **Daily** | **Strategic trend assessment** | **Trend phase + score** |

The Trend Compass occupies a fundamentally different niche. The scalpers and monitors are intraday tactical tools. The Trend Compass is a strategic assessment: is NVDA in a healthy uptrend? Is QQQ losing momentum? Is IWM showing early reversal signs?

**Workflow integration**: Check the Trend Compass to establish the macro bias, then use the Market Monitors to scan your watchlist for intraday alignment, then deploy the 0DTE Scalpers on the strongest setups.

---

## Installation

1. Open TradingView and navigate to any **Daily chart** (SPY, NVDA, AAPL, etc.).
2. Open the **Pine Editor** (bottom panel).
3. Delete any existing code in the editor.
4. Paste the entire contents of `trend_compass_daily_v1_0.pine`.
5. Click **Add to chart**.

### Data Requirements

- **252 bars of daily history** required for 52-week high/low calculations. Most liquid stocks/ETFs will have this by default.
- **Weekly data access** required for HTF EMA confirmation and prior week levels. Standard on all TradingView plans.
- **Monthly data access** required for prior month levels. Standard on all plans.

### Chart Settings

This indicator has no session-based logic (no RTH boundaries, no opening range). Chart timezone settings do not affect any calculations. Apply to any chart timezone.

---

## Indicator Components

### EMA Ribbon (10/21/50)

Three EMAs with a cloud fill between fast and slow. These are the standard trend-following lengths for daily charts:

- **10 EMA**: ~2 weeks of trading. Captures short-term momentum shifts.
- **21 EMA**: ~1 month. The intermediate trend anchor that swing traders focus on.
- **50 EMA**: ~2.5 months. The primary structural trend line. Institutional mean-reversion target.

**Alignment states:**

| State | Condition | Interpretation |
|-------|-----------|---------------|
| Full Bullish Stack | Fast > Mid > Slow > 200 EMA | All timeframes aligned bullish. Strongest configuration. |
| Partial Bull | Fast > Mid > Slow, but below 200 | Developing uptrend, not yet structural. Recovery in progress. |
| Full Bearish Stack | Fast < Mid < Slow < 200 EMA | All timeframes aligned bearish. Strongest downtrend. |
| Partial Bear | Fast < Mid < Slow, but above 200 | Developing downtrend or healthy pullback within uptrend. |
| Compressed | EMA spread < 0.5% | Trend transition. Indecision. Breakout imminent. |
| Mixed | Any other | No clear alignment. Chop. |

The **cloud fill** between fast and slow EMAs provides instant visual context: green = bullish alignment, red = bearish, yellow = mixed.

### Anchor EMA (200)

The 200-day EMA is the institutional dividing line. It is the single most-watched technical level on a daily chart.

- **Above 200 EMA**: Structural regime is bullish. Institutional bias favors buying dips.
- **Below 200 EMA**: Structural regime is bearish. Institutional bias favors selling rallies.
- **Distance from 200 EMA**: Shown as a percentage in the dashboard. Large positive distances (>10%) suggest extended conditions; large negative distances suggest oversold conditions at the structural level.

Plotted as a thick orange line for maximum visibility.

### HTF Context (Weekly 50 EMA)

The 50-period EMA from the Weekly timeframe, pulled via `request.security()`. On a daily chart, this represents ~250 trading days (~1 year) of smoothed trend.

- Used as a scoring condition in the composite trend engine.
- Plotted as a purple step line for visual reference.
- Configurable: can switch to Monthly timeframe for an even longer-term view.

### Trend Phase Classifier

The core differentiator of this indicator. Six states that map where a trend is in its lifecycle:

| Phase | Conditions | Interpretation | Action Bias |
|-------|-----------|----------------|-------------|
| **EMERGING** | ADX < 25, ADX rising, EMAs separating | New trend forming out of consolidation | Early entry. Tightest risk/reward. |
| **ACCELERATING** | ADX > 20, ADX rising, EMA slope confirming | Trend gaining momentum | Trend-following. Add on pullbacks. |
| **MATURE** | ADX > 30, ADX flat | Trend intact but momentum plateauing | Trail stops. Reduce new entries. |
| **EXHAUSTING** | ADX declining from > 30 | Trend losing steam | Prepare for consolidation or reversal. Take profits. |
| **CONSOLIDATING** | ADX < 20, EMAs compressed | No trend. Range-bound. | Avoid directional bias. Wait for breakout. |
| **REVERSING** | Recent EMA crossovers (fast/mid within 3 bars or mid/slow within 5) | Active trend change underway | Re-evaluate bias. Watch for confirmation. |

The phase classification uses ADX value, ADX slope (rate of change), EMA spread compression, and EMA crossover detection. Priority order ensures the most actionable state is always shown.

### Composite Trend Score

Weighted scoring engine architecturally consistent with the Market Monitor 5m and 15-min scalper.

**Structural Conditions (2x weight when weighted mode enabled):**

| # | Condition | Bullish (+2) | Bearish (-2) |
|---|-----------|-------------|-------------|
| 1 | EMA Stack | Fast > Mid > Slow | Fast < Mid < Slow |
| 2 | Price vs 200 EMA | Above | Below |
| 3 | Price vs HTF EMA | Above Weekly 50 | Below Weekly 50 |

**Confirmation Conditions (1x weight always):**

| # | Condition | Bullish (+1) | Bearish (-1) |
|---|-----------|-------------|-------------|
| 4 | ADX + DI Direction | ADX > 20 AND DI+ > DI- | ADX > 20 AND DI- > DI+ |
| 5 | RSI Position | RSI > 55 | RSI < 45 |
| 6 | MACD vs Signal | MACD above signal | MACD below signal |
| 7 | OBV Slope | OBV EMA rising | OBV EMA falling |
| 8 | 50 EMA Slope | Positive | Negative |

**Score Ranges (weighted mode, max = +/-11):**

| Score | Label | Interpretation |
|-------|-------|---------------|
| +7 to +11 | STRONG BULL | High-conviction bullish trend. All major components aligned. |
| +4 to +6 | LEAN BULL | Moderate bullish lean. Trend present, watch for confirmation. |
| +1 to +3 | WEAK BULL | Slight bullish tilt. Low conviction. |
| 0 | NEUTRAL | No directional bias. Choppy or transitioning. |
| -1 to -3 | WEAK BEAR | Slight bearish tilt. |
| -4 to -6 | LEAN BEAR | Moderate bearish lean. |
| -7 to -11 | STRONG BEAR | High-conviction bearish trend. |

**Score Trend (IMPROVING / STABLE / FADING)**: Compares the current score to its 5-bar SMA. Detects whether the trend is strengthening or deteriorating before the raw score moves significantly.

### ADX / DMI + Slope Analysis

Standard 14-period ADX with DI+/DI- directional indicators. The key enhancement is **slope analysis**:

- **ADX Slope**: Rate of change over a configurable lookback (default 5 bars). Tells you whether trend strength is increasing (RISING), decreasing (FALLING), or stable (FLAT).
- A high ADX with a falling slope is more bearish for trend continuation than a moderate ADX with a rising slope.

**ADX Tiers:**

| ADX Value | Tier | Interpretation |
|-----------|------|---------------|
| > 45 | EXTREME | Very strong trend. Rare. Often precedes exhaustion. |
| 30 - 45 | STRONG | Clear trending conditions. Trade with the trend. |
| 20 - 30 | MODERATE | Trend present but not dominant. Selective entries. |
| < 20 | WEAK | No meaningful trend. Range-bound conditions. |

### RSI + Divergence Detection

14-period RSI with pivot-based divergence detection.

**Divergence Types:**

| Type | Price Action | RSI Action | Signal |
|------|-------------|-----------|--------|
| Bullish Divergence | Lower low | Higher low | Potential bottom / reversal up |
| Bearish Divergence | Higher high | Lower high | Potential top / reversal down |
| Hidden Bullish | Higher low | Lower low | Trend continuation (pullback in uptrend) |
| Hidden Bearish | Lower high | Higher high | Trend continuation (bounce in downtrend) |

Divergences are drawn as dashed line segments on the chart connecting the divergent pivot points. Regular divergences use green (bullish) or red (bearish) lines. The dashboard shows the current divergence state.

Divergence state decays after 15 bars to avoid stale signals.

### MACD + Divergence Detection

Standard MACD (12/26/9) with histogram-based divergence detection. Same four-state system as RSI divergence.

MACD is included here (and not in the intraday scalper suite) because it is specifically useful on Daily/4H timeframes for institutional-grade momentum assessment. On 1-minute charts it produces noise; on Daily charts it is a standard institutional signal.

MACD divergence lines are drawn with dotted style in cyan (bullish) or purple (bearish) to distinguish from RSI divergence lines.

### OBV Trend Confirmation

On-Balance Volume with a 21-period EMA smoothing. OBV confirms whether volume participation supports the price trend:

| State | Condition | Interpretation |
|-------|-----------|---------------|
| CONFIRMING | Price trend and OBV slope agree | Healthy trend. Volume supports direction. |
| DIVERGING | Price trend and OBV slope disagree | Warning. Price moving without volume conviction. |
| NEUTRAL | No clear assessment | Inconclusive. |

OBV divergence from price is one of the earliest warnings of trend weakness. Price can continue for a while without volume support, but not indefinitely.

### Relative Volume

Current bar volume divided by the 20-bar SMA of volume:

| Value | Label | Interpretation |
|-------|-------|---------------|
| > 2.0x | SPIKE | Unusual volume. News, earnings, or institutional activity. |
| > 1.2x | ABOVE | Healthy participation above average. |
| 0.8x - 1.2x | NORMAL | Typical volume. |
| < 0.8x | DRY | Below-average volume. Low conviction moves. |

### ATR Regime Classification

14-period ATR ranked against its own history over a configurable lookback (default 100 bars):

| Percentile | Regime | Interpretation |
|-----------|--------|---------------|
| >= 90th | EXTREME | Outsized moves. Volatility spike. |
| >= 70th | HIGH | Active trending conditions. |
| 30th - 70th | NORMAL | Standard conditions. |
| < 30th | LOW | Compressed volatility. Breakout building. |

### Bollinger Band Width Percentile

BB Width = (Upper - Lower) / Middle * 100, ranked as a percentile over a configurable lookback:

| Percentile | Label | Interpretation |
|-----------|-------|---------------|
| < 10th | COMPRESSED | Extreme compression. Breakout highly likely. |
| < 30th | NARROW | Below-average width. Volatility expansion approaching. |
| 30th - 70th | NORMAL | Standard conditions. |
| > 70th | WIDE | Above-average width. Active trending or volatile. |
| > 90th | EXPANDED | Extreme width. Often follows major moves. |

BB Width Percentile at sub-10th levels is one of the most reliable precursors to a volatility breakout.

### Key Structural Levels

**Prior Week High / Low / Close** — drawn as blue dashed lines. Institutional order flow clusters around weekly pivots. Prior week high is resistance; prior week low is support; prior week close is the "fair value" reference.

**Prior Month High / Low / Close** — drawn as purple dashed lines. Monthly levels carry even more structural weight. A monthly high breakout is significant; a monthly low breakdown is significant.

**52-Week High / Low** — drawn as solid orange lines. The ultimate structural context. How far are we from the 52-week extremes? The dashboard shows position-in-range as a percentage (0% = at 52-week low, 100% = at 52-week high).

### Fibonacci Retracement (optional)

**Default: OFF.** Toggle on in settings under "Fibonacci Retracement."

Auto-calculates the highest high and lowest low over a configurable lookback (default 50 bars on daily = ~2.5 months), then draws the standard retracement levels:

- **38.2%** — shallow retracement. Strong trend continuation zone.
- **50.0%** — midpoint. Institutional reference level.
- **61.8%** — deep retracement. Golden ratio. Last line of defense for trend continuation.

Levels are drawn as dashed horizontal lines with percentage labels.

### Ichimoku Cloud (optional)

**Default: OFF.** Toggle on in settings under "Ichimoku Cloud."

Traditional Ichimoku Kinko Hyo with standard daily parameters:

| Component | Default | Description |
|-----------|---------|-------------|
| Tenkan-sen | 9 | Conversion line. Short-term momentum. |
| Kijun-sen | 26 | Base line. Medium-term trend. |
| Senkou A | (Tenkan + Kijun) / 2 | Leading Span A. Displaced forward 26 bars. |
| Senkou B | 52-period midpoint | Leading Span B. Displaced forward 26 bars. |
| Chikou | Close | Lagging Span. Displaced backward 26 bars. |

The Kumo (cloud) between Senkou A and B is filled green (bullish) or red (bearish). Price above the cloud = bullish; below = bearish; inside = transitional.

Ichimoku is a complete trading system in itself but adds significant visual complexity. It's provided as an opt-in module for traders who specifically use it.

### 52-Week High / Low

Calculated via `ta.highest(high, 252)` and `ta.lowest(low, 252)` using 252 trading days (1 year). Requires 252 bars of history on the chart.

The dashboard shows **position in 52-week range** as a percentage:

- 90%+ — near 52-week highs. Strong trend or potential resistance.
- 50% — middle of range.
- 10% or below — near 52-week lows. Potential capitulation or support.

### Dashboard

17-row compact table with three columns, architecturally consistent with the existing suite:

| Row | Field | Content |
|-----|-------|---------|
| 0 | Header | "Trend Compass - Daily" (merged, purple accent bar) |
| 1 | Phase | EMERGING / ACCELERATING / MATURE / EXHAUSTING / CONSOLIDATING / REVERSING |
| 2 | Score | Composite score with label (STRONG BULL, LEAN BEAR, etc.) |
| 3 | Momentum | IMPROVING / STABLE / FADING (score trend) |
| 4 | EMA Stack | FULL BULL / PARTIAL BULL / COMPRESSED / PARTIAL BEAR / FULL BEAR / MIXED |
| 5 | ADX | Value + tier label |
| 6 | ADX Slope | RISING / FLAT / FALLING + slope value |
| 7 | RSI | Value + OB/OS/BULLISH/BEARISH/NEUTRAL |
| 8 | MACD | Histogram direction + above/below signal |
| 9 | RSI Div | BULL DIV / BEAR DIV / HIDDEN BULL / HIDDEN BEAR / NONE |
| 10 | MACD Div | Same four states + NONE |
| 11 | OBV | CONFIRMING / DIVERGING / NEUTRAL |
| 12 | Rel Vol | Multiplier + SPIKE/ABOVE/NORMAL/DRY |
| 13 | ATR | Value + regime + percentile |
| 14 | BB Width | Percentile + COMPRESSED/NARROW/NORMAL/WIDE/EXPANDED |
| 15 | HTF Trend | Weekly (or Monthly) trend alignment |
| 16 | 200 EMA Δ | Distance from 200 EMA (%) + 52-week position (%) |

Configurable position (4 corners) and size (Tiny / Small / Normal / Large).

### Alerts

12 alert conditions:

| Alert | Trigger | Use Case |
|-------|---------|----------|
| Trend Phase Changed | Any phase transition | Lifecycle shift notification |
| Strong Bullish Trend | Score >= +7 (weighted) or +5 (equal) | High-conviction bull setup |
| Strong Bearish Trend | Score <= -7 (weighted) or -5 (equal) | High-conviction bear setup |
| Score Crossed Neutral | Score crosses zero | Trend loss or reversal |
| RSI Bullish Divergence | New RSI bull div detected | Potential bottom |
| RSI Bearish Divergence | New RSI bear div detected | Potential top |
| MACD Bullish Divergence | New MACD bull div detected | Potential bottom |
| MACD Bearish Divergence | New MACD bear div detected | Potential top |
| Golden Cross (50/200) | 50 EMA crosses above 200 EMA | Major structural bullish shift |
| Death Cross (50/200) | 50 EMA crosses below 200 EMA | Major structural bearish shift |
| ATR Regime Change | Volatility regime shifts | Conditions changing |
| BB Extreme Compression | BB width < 5th percentile | Breakout imminent |

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
| 200 EMA | Deep orange | #FF6D00 |
| HTF EMA | Purple | #E040FB |
| Prior Week levels | Blue | #42A5F5 |
| Prior Month levels | Purple | #AB47BC |
| 52-Week levels | Salmon | #FF8A65 |
| Fib 38.2% | Teal | #80CBC4 |
| Fib 50.0% | Amber | #FFD54F |
| Fib 61.8% | Orange | #FF8A65 |
| Ichimoku Tenkan | Blue | #2979FF |
| Ichimoku Kijun | Orange | #FF6D00 |
| Kumo Cloud (bull) | Green (88% transparent) | #00E676 |
| Kumo Cloud (bear) | Red (88% transparent) | #FF1744 |
| RSI div lines | Green (bull) / Red (bear) | Dashed |
| MACD div lines | Cyan (bull) / Purple (bear) | Dotted |

### Chart Element Legend

| Element | Style | When Visible |
|---------|-------|-------------|
| Fast EMA (10) | Thin green line | Always (toggleable) |
| Mid EMA (21) | Thin yellow line | Always (toggleable) |
| Slow EMA (50) | Thin red line | Always (toggleable) |
| 200 EMA | Thick orange line | Always (toggleable) |
| EMA Cloud | Semi-transparent fill | When Show Cloud enabled |
| Weekly 50 EMA | Purple step line | When HTF enabled |
| Prior Week H/L | Blue dashed lines | When enabled |
| Prior Week C | Blue dotted line | When enabled |
| Prior Month H/L | Purple dashed lines | When enabled |
| Prior Month C | Purple dotted line | When enabled |
| 52-Week H/L | Salmon solid lines | When enabled |
| Fib levels | Colored dashed lines | When Fib enabled |
| Ichimoku Cloud | Filled cloud + lines | When Ichimoku enabled |
| RSI divergence | Dashed line on price | When divergence detected |
| MACD divergence | Dotted line on price | When divergence detected |

---

## Tuning Guide

### EMA Lengths

Default 10/21/50 are the standard daily trend-following periods. Alternatives:

- **8/13/34**: Fibonacci-based. Slightly more responsive.
- **12/26/50**: Matches MACD fast/slow internally. Conceptual consistency.
- **20/50/100**: More conservative. Fewer whipsaws, slower signals.

### Anchor EMA Length

Default 200. The 200-day moving average is the most widely watched level in equity markets. Changing this is not recommended unless you have a specific thesis.

- **100**: More responsive. Shows intermediate structural trend.
- **150**: Compromise between 100 and 200.

### HTF Timeframe Selection

Default: Weekly. For an even longer-term macro view, switch to Monthly. The Monthly 50 EMA represents ~50 months (~4 years) of smoothed trend — genuinely long-term structural context.

### ADX Slope Sensitivity

Default threshold: 0.3. This determines what counts as "RISING" vs "FLAT" vs "FALLING."

- **Lower (0.1-0.2)**: More sensitive. Will classify more periods as rising/falling.
- **Higher (0.5+)**: Less sensitive. Only significant slope changes register.

### RSI / MACD Divergence Pivot Bars

Default: 5 left, 3 right. These control how significant a swing point must be to qualify as a pivot for divergence detection.

- **Larger values (8L/5R)**: Fewer, more significant divergences. Higher conviction per signal.
- **Smaller values (3L/2R)**: More frequent divergences. More noise but earlier detection.

The right bars parameter introduces a lag equal to that many bars — the divergence is confirmed N bars after the actual pivot.

### ATR Percentile Lookback

Default: 100 bars (~5 months on daily). This is the window against which current ATR is ranked.

- **50 bars**: More responsive. Captures recent volatility shifts quickly.
- **200+ bars**: Longer context. Ranks against nearly a full year of volatility history.

### BB Width Percentile Lookback

Default: 100 bars. Same tradeoff as ATR percentile.

### Fibonacci Lookback

Default: 50 bars (~2.5 months). This defines the window for finding the highest high and lowest low that anchor the Fibonacci levels.

- **20 bars**: Short-term swing. Levels change frequently.
- **100 bars**: Longer-term structure. More stable levels.

### Ichimoku Parameters

Default: 9/26/52/26 (standard). These are the original Ichimoku settings designed for the Japanese 6-day trading week (9 = 1.5 weeks, 26 = 1 month, 52 = 2 months).

Some Western traders prefer 10/30/60/30 to adapt for the 5-day week, but the originals remain the most widely used and are recommended.

### Scoring Mode

**Weighted (default)**: Structural conditions (EMA alignment, price vs 200 EMA, price vs HTF EMA) score 2 points each. Confirmation conditions score 1 point each. This reflects the reality that structural position matters more than any single oscillator reading.

**Equal**: All conditions score 1 point. Max = +/-8. Simpler to interpret.

---

## Architecture Notes

- **`request.security()` budget**: 3 calls total (Weekly EMA, prior week H/L/C, prior month H/L/C). Well under the 40-call limit.
- **Loops**: Two loops (ATR percentile ranking, BB width percentile ranking), each over configurable lookback (default 100 bars). Both run once per bar and are lightweight.
- **Object counts**: Lines for key levels, divergence lines, and optional Fibonacci. Labels for Fibonacci annotations. One table for the dashboard. Well under TradingView's limits given the lower bar density of daily charts.
- **Dashboard**: Only rendered on `barstate.islast` to avoid per-bar table overhead.
- **Divergence state**: Uses `var` variables for persistent pivot tracking across bars. State decays after 15 bars to prevent stale signals.
- **Ichimoku displacement**: Uses the `offset` parameter on `plot()` for forward/backward displacement. This is handled natively by TradingView's rendering engine.

---

## Known Limitations

- **Daily-optimized**: While the indicator will technically run on any timeframe, the EMA lengths, divergence pivot parameters, trend phase thresholds, and key level references are calibrated for daily bars. A 4-Hour variant with recalibrated parameters is planned.
- **52-week H/L requires 252 bars of history**: If the chart doesn't have enough history (new IPO, limited data), these levels will be based on available data only.
- **Divergence detection has inherent lag**: The right bars parameter means divergences are confirmed N bars after the actual pivot. This is a fundamental property of pivot-based detection.
- **Fibonacci levels are range-based, not swing-based**: The fib module uses the highest high and lowest low over a lookback window, not algorithmically detected swing points. This is simpler and more stable but may not always match hand-drawn fibs.
- **Prior week/month levels use `lookahead=barmerge.lookahead_on`**: This is correct for daily charts (we want the completed prior period), but means these levels are forward-looking in backtesting contexts. This is standard practice for reference levels.
- **OBV is volume-dependent**: For instruments with unreliable or synthetic volume data (some indices, some forex), OBV readings may be misleading. Disable the OBV module for these instruments.
