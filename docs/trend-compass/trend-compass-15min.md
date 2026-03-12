# Trend Compass 15m

15-minute optimized strategic intraday trend assessment overlay for equities, ETFs, and indices.

---

## Design Philosophy

The 15-minute variant is the critical bridge between your intraday execution layer (the 0DTE Scalpers) and your multi-day contextual layer (the 1H and 4H Trend Compasses). It answers: **What is the structural intraday trend, and how is it interacting with the immediate multi-hour and multi-day moving averages?**

The 15-minute timeframe was selected over the 10-minute for three architectural reasons:

1. **Fractional Alignment:** The 15m chart divides cleanly into the 1H (4 bars) and 4H (16 bars) timeframes, allowing for cleaner moving average synchronization.
2. **Execution Synergy:** This variant shares a timeframe with the highest-conviction intraday execution tool (`spy_0dte_scalper_15min.pine`), providing direct 1:1 context for trade filtering.
3. **Noise Filtration:** At 26 bars per standard RTH session, it absorbs momentary volatility spikes that trigger false divergence and phase shifts on sub-15m charts, preserving the integrity of the trend phase classifier.

**Key architectural difference from the 1H variant**: The 15m chart tracks four higher-timeframe references directly above it: 1H 50 EMA, 4H 50 EMA, Daily 50 EMA, and Daily 200 EMA. It drops the Weekly 50 EMA, as a weekly moving average is too distant to provide actionable resistance/support for a 15-minute intraday swing.

---

## Key Differences from Other Variants

| Feature | 4H Variant | 1H Variant | 15m Variant |
|---------|------------|------------|-------------|
| EMA Ribbon | 10/21/50 (local 4H) | 10/21/50 (local 1H) | 10/21/50 (local 15m) |
| Anchor EMA | Daily 200 EMA (HTF) | Daily 200 EMA (HTF) | Daily 200 EMA (HTF) |
| HTF #1 | Daily 50 EMA | 4H 50 EMA | **1H 50 EMA** |
| HTF #2 | Daily 200 EMA | Daily 50 EMA | **4H 50 EMA** |
| HTF #3 | Weekly 50 EMA | Daily 200 EMA | **Daily 50 EMA** |
| HTF #4 | — | Weekly 50 EMA | **Daily 200 EMA** |
| Divergence Pivots | 4L/2R default | 3L/2R default | **3L/2R default** |
| Divergence Decay | 20 bars (~3.5 days) | 30 bars (~4.6 days) | **40 bars (~1.5 days)** |
| ATR/BB Percentile LB| 200 bars (~33 days) | 300 bars (~46 days) | **390 bars (~15 days)** |
| EMA Compression | 0.3% spread | 0.2% spread | **0.15% spread** |
| EMA Slope Threshold | 0.005 | 0.003 | **0.002** |
| Cross Detection | Fast/Mid: 4, Mid/Slow: 6 | Fast/Mid: 5, Mid/Slow: 8 | **Fast/Mid: 6, Mid/Slow: 10** |
| Fibonacci Lookback | 100 bars (~17 days) | 150 bars (~23 days) | **260 bars (~10 days)** |
| `request.security` | 6 calls | 7 calls | **7 calls** |

---

## Features

- **EMA Ribbon** (10/21/50) with dynamic cloud fill, computed on 15m bars for intraday swing context.
- **1H 50 EMA** pulled via `request.security()` — immediate multi-hour bridge plotted as a purple step line.
- **4H 50 EMA** pulled via `request.security()` — multi-day context plotted as a green step line.
- **Daily 50 EMA** pulled via `request.security()` — macro swing trend plotted as a cyan step line.
- **Daily 200 EMA** pulled via `request.security()` — the institutional anchor plotted as a thick orange line.
- **Trend Phase Classifier** — six states: EMERGING, ACCELERATING, MATURE, EXHAUSTING, CONSOLIDATING, REVERSING.
- **Composite Weighted Trend Score** (-11 to +11) from 8 conditions across structural and confirmation tiers.
- **Score Trend Tracking** (IMPROVING / STABLE / FADING) via 5-bar SMA comparison.
- **ADX Slope Analysis** — RISING / FLAT / FALLING trend strength momentum.
- **50 EMA Slope + Acceleration** — quantifies trend angle and curvature on the 15m timeframe with high sensitivity (0.002 threshold).
- **RSI Divergence Detection** — bullish, bearish, hidden bull, hidden bear with visual line segments. Tuned for fast 15m confirmation (3L/2R).
- **MACD Divergence Detection** — same four-state detection on the MACD histogram.
- **OBV Trend Confirmation** — CONFIRMING / DIVERGING / NEUTRAL.
- **Relative Volume** (20-bar SMA baseline) with SPIKE / ABOVE / NORMAL / DRY classification.
- **ATR Percentile Ranking** — LOW / NORMAL / HIGH / EXTREME regime classification with 390-bar lookback (~15 days).
- **Bollinger Band Width Percentile** — compression detection with 390-bar lookback.
- **52-Week High / Low** pulled from Daily timeframe for accuracy.
- **Prior Day High / Low / Close** reference levels (blue dashed).
- **Prior Week High / Low / Close** reference levels (purple dashed).
- **Fibonacci Retracement** (optional, default OFF) — 38.2%, 50%, 61.8% levels over 260-bar lookback (~10 trading days).
- **Ichimoku Cloud** (optional, default OFF) — standard 9/26/52/26 parameters.
- **19-row heads-up dashboard** with color-coded status indicators outlining HTF EMA alignment.
- **12 alert conditions** covering phase changes, score thresholds, divergences, volatility shifts, and compression breakout watches.

---

## Role in the Indicator Suite

| Indicator | Primary TF | Focus | Update Cadence |
|-----------|-----------|-------|----------------|
| 0DTE Scalper 15m | 15-min | Session directional bias + entries | Every 15 min |
| Market Monitor 5m | 5-min | Enhanced watchlist bias | Every 5 min |
| **Trend Compass 15m** | **15-Min** | **Intraday structural trend** | **Every 15 min** |
| Trend Compass 1H | 1-Hour | Intraday swing trend | ~6.5x per session |
| Trend Compass 4H | 4-Hour | Multi-day swing trend | ~2x per session |

**Workflow**: The 1H and 4H Compasses establish the macro and multi-day swing bias. The 15m Trend Compass determines if the current intraday session is participating in that trend, failing, or consolidating. It serves as the direct contextual filter for the 15m 0DTE Scalper.

---

## Installation

1. Open TradingView and navigate to any **15-Minute chart**.
2. Open the **Pine Editor** (bottom panel).
3. Delete any existing code in the editor.
4. Paste the entire contents of `trend_compass_15m.pine`.
5. Click **Add to chart**.

### Data Requirements

- **1-Hour, 4-Hour, Daily, and Weekly data access** required for HTF EMA pulls and reference calculations. Standard on all TradingView plans.
- **Sufficient 15m bar history** for percentile calculations (390+ bars default).

---

## Indicator Components

### EMA Ribbon (10/21/50)

Computed locally on 15m bars. On a 15m chart with 26 bars per RTH session:

- **10 EMA**: ~2.5 hours. Captures immediate session momentum.
- **21 EMA**: ~5.25 hours. The intermediate trend covering roughly one RTH session.
- **50 EMA**: ~2 days. The primary intraday structural trend line.

The **EMA Stack** assessment incorporates the Daily 200 EMA as the macro anchor:

- **COMPRESSED**: Triggers when the spread between fast and slow is < 0.15%. A tighter threshold is required on 15m because sub-hourly averages cluster more tightly than daily averages.

### HTF EMAs (1H 50, 4H 50, Daily 50, Daily 200)

Four institutional-grade reference lines pulled from higher timeframes. The Weekly 50 is discarded in this variant to prioritize the 1H 50, which provides actionable intraday support/resistance.

| Line | Color | Style | Purpose |
|------|-------|-------|---------|
| 1H 50 EMA | Purple (#E040FB) | Step line, thin | Immediate multi-hour bridge. |
| 4H 50 EMA | Green (#66BB6A) | Step line, thin | Multi-day structural context. |
| Daily 50 EMA | Cyan (#26C6DA) | Step line, thick | Macro swing trend. |
| Daily 200 EMA | Orange (#FF6D00) | Solid, thick | The institutional dividing line. |

### Composite Trend Score

Weighted architecture identical to higher timeframe variants.

**Structural Conditions (2x weight):**

1. 15m EMA Stack (Fast > Mid > Slow)
2. Price vs Daily 200 EMA
3. Price vs 4H 50 EMA (replaces Weekly 50 condition from HTF variants)

**Confirmation Conditions (1x weight):**

1. ADX + DI Direction
2. RSI Position (>55 or <45)
3. MACD vs Signal
4. OBV Slope
5. 15m 50 EMA Slope

### Divergence Detection (RSI + MACD)

- **Default pivots**: 3 left / 2 right. This confirms divergences exactly 30 minutes (2 bars) after the actual pivot high/low forms.
- **Decay window**: 40 bars (~1.5 RTH trading days). Divergences expire cleanly after the current and subsequent session to prevent stale intraday signals.

---

## Dashboard

19-row compact table highlighting HTF EMA proximity:

| Row | Field | Content |
|-----|-------|---------|
| 0 | Header | "Trend Compass - 15m" (merged, purple accent bar) |
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
| 16 | D 50 EMA | ABOVE/BELOW |
| 17 | 4H 50 EMA | ABOVE/BELOW |
| 18 | 1H 50 EMA | ABOVE/BELOW |

---

## Tuning Guide

### EMA Ribbon

- **10/21/50 (Default)**: Best for smoothing intraday noise while preserving RTH session structure.
- **8/21/34**: Fibonacci-based. Slightly faster, aligns well with opening range breakouts.

### Divergence Pivot Bars

- **3L/2R (Default)**: Fast confirmation.
- **5L/3R**: Use this if you are getting too many shallow intraday divergence signals during choppy, low-volume mid-day sessions.

### Percentile Lookbacks (ATR / BBW)

- **390 bars (Default)**: Represents ~15 RTH trading days (26 bars * 15). Gives a solid two-to-three week baseline for volatility.
- **130 bars**: Represents exactly 1 trading week. Use for highly responsive, short-term volatility breakout detection.

---

## Architecture Notes

- **`request.security()` budget**: 7 calls total: 1H 50 EMA, 4H 50 EMA, Daily 50 EMA, Daily 200 EMA, prior day H/L/C, prior week H/L/C, 52-week high/low.
- **Alignment limitations**: Using `lookahead=barmerge.lookahead_off` to pull 1H and 4H data down to the 15m chart ties the values to the close of the last completed timeframe bar. This prevents repainting, but creates "step-functions" in the plot lines exactly at the hourly and 4-hourly boundaries.
- **OBV Aggregation**: Because this operates on a sub-hourly chart, OBV values will vary based on your broker's handling of pre-market/after-hours volume data. The slope direction remains reliable, but absolute values should not be compared across different feeds.
