# Trend Compass 1H

1-hour optimized strategic trend assessment overlay for equities, ETFs, and indices.

---

## Design Philosophy

The 1-hour variant bridges the gap between the intraday Market Monitors and the 4-Hour Trend Compass. It answers: **What is the intraday-to-multi-day trend? Is it gaining or losing momentum? Are divergences forming?** — with the fastest update cycle in the Trend Compass family.

The 1H chart produces ~6.5 bars per RTH session (~10+ with extended hours), making it granular enough to track trend shifts developing within a single trading day, while slow enough to filter out the noise that dominates sub-hourly timeframes. It serves active swing traders and position builders who need trend context refreshed multiple times per session without dropping to the rapid-fire cadence of the scalping indicators.

**Key architectural difference from the 4H variant**: The 1H chart introduces a fourth HTF layer — the **4H 50 EMA** — as the immediate structural reference above the local timeframe. This sits between the local 1H ribbon and the Daily EMAs, providing a "bridge" level that captures the multi-day momentum the 4H Trend Compass evaluates locally. The full HTF stack is: 4H 50 EMA → Daily 50 EMA → Daily 200 EMA → Weekly 50 EMA.

**Key architectural difference from the Daily variant**: Like the 4H variant, the 1H chart has no meaningful local 200 EMA (200 bars on 1H ≈ 31 trading days — insufficient for institutional structural significance). The Daily 200 EMA is pulled via `request.security()` as the primary structural anchor.

---

## Key Differences from Other Variants

| Feature | Daily Variant | 4H Variant | 1H Variant |
|---------|--------------|------------|------------|
| EMA Ribbon | 10/21/50 (local) | 10/21/50 (local, 4H bars) | 10/21/50 (local, 1H bars) |
| Anchor EMA | 200 (local daily) | Daily 200 EMA (HTF) | Daily 200 EMA (HTF) |
| HTF #1 | Weekly 50 EMA | Daily 50 EMA | **4H 50 EMA** |
| HTF #2 | — | Daily 200 EMA | Daily 50 EMA |
| HTF #3 | — | Weekly 50 EMA | Daily 200 EMA |
| HTF #4 | — | — | Weekly 50 EMA |
| Key Levels | Prior Week + Prior Month H/L/C | Prior Day + Prior Week H/L/C | Prior Day + Prior Week H/L/C |
| 52-Week H/L | Local `ta.highest(252)` | Via Daily `request.security` | Via Daily `request.security` |
| Divergence Pivots | 5L/3R default | 4L/2R default | **3L/2R default** |
| Divergence Decay | 15 bars (~15 days) | 20 bars (~3.5 days) | **30 bars (~4.6 days)** |
| ATR/BB Percentile LB | 100 bars (~5 months) | 200 bars (~33 days) | **300 bars (~46 days)** |
| EMA Compressed Threshold | 0.5% spread | 0.3% spread | **0.2% spread** |
| EMA Slope Threshold | 0.01 | 0.005 | **0.003** |
| Cross Detection Window | Fast/Mid: 3, Mid/Slow: 5 | Fast/Mid: 4, Mid/Slow: 6 | **Fast/Mid: 5, Mid/Slow: 8** |
| Fibonacci Lookback | 50 bars (~2.5 months) | 100 bars (~17 days) | **150 bars (~23 days)** |
| `request.security` calls | 3 | 6 | **7** |
| Dashboard Rows | 17 | 18 | **19** |

---

## Features

- **EMA Ribbon** (10/21/50) with dynamic cloud fill, computed on 1H bars for intraday-to-multi-day swing context.
- **4H 50 EMA** pulled via `request.security()` — the immediate HTF structural bridge plotted as a green step line.
- **Daily 200 EMA** pulled via `request.security()` — the institutional anchor plotted as a thick orange line.
- **Daily 50 EMA** pulled via `request.security()` — intermediate structural trend plotted as a cyan step line.
- **Weekly 50 EMA** pulled via `request.security()` — macro trend reference plotted as a purple step line.
- **Trend Phase Classifier** — six states: EMERGING, ACCELERATING, MATURE, EXHAUSTING, CONSOLIDATING, REVERSING.
- **Composite Weighted Trend Score** (-11 to +11) from 8 conditions across structural and confirmation tiers.
- **Score Trend Tracking** (IMPROVING / STABLE / FADING) via 5-bar SMA comparison.
- **ADX Slope Analysis** — RISING / FLAT / FALLING trend strength momentum.
- **50 EMA Slope + Acceleration** — quantifies trend angle and curvature on the 1H timeframe.
- **RSI Divergence Detection** — bullish, bearish, hidden bull, hidden bear with visual line segments. Pivot detection tuned for 1H speed (3L/2R).
- **MACD Divergence Detection** — same four-state detection on the MACD histogram with 1H-tuned pivots.
- **OBV Trend Confirmation** — CONFIRMING / DIVERGING / NEUTRAL.
- **Relative Volume** (20-bar SMA baseline) with SPIKE / ABOVE / NORMAL / DRY classification.
- **ATR Percentile Ranking** — LOW / NORMAL / HIGH / EXTREME regime classification with 300-bar lookback.
- **Bollinger Band Width Percentile** — compression detection with 300-bar lookback.
- **52-Week High / Low** pulled from Daily timeframe for accuracy, with position-in-range percentage.
- **Prior Day High / Low / Close** reference levels (blue dashed).
- **Prior Week High / Low / Close** reference levels (purple dashed).
- **Fibonacci Retracement** (optional, default OFF) — 38.2%, 50%, 61.8% levels over 150-bar lookback (~23 trading days on 1H).
- **Ichimoku Cloud** (optional, default OFF) — standard 9/26/52/26 parameters.
- **19-row heads-up dashboard** with color-coded status indicators.
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
| **Trend Compass 1H** | **1-Hour** | **Intraday swing trend** | **~6.5x per session** |
| Trend Compass 4H | 4-Hour | Multi-day swing trend | ~2x per session |
| Trend Compass Daily | Daily | Strategic position trend | Once per day |

**Workflow**: Trend Compass Daily sets the macro bias. Trend Compass 4H provides multi-day swing context. **Trend Compass 1H provides intraday swing context** — is today's price action strengthening or weakening the multi-day trend? Market Monitors scan for short-timeframe alignment. 0DTE Scalpers execute.

The 1H variant is particularly valuable for:

- **Same-day trend assessment** before deploying scalping tools.
- **Intraday divergence detection** that the 4H variant is too slow to catch.
- **4H 50 EMA monitoring** as a key support/resistance level for swing entries.
- **Pre-close evaluation** — has today's action improved or degraded the trend score?

---

## Installation

1. Open TradingView and navigate to any **1-Hour chart** (SPY, NVDA, AAPL, etc.).
2. Open the **Pine Editor** (bottom panel).
3. Delete any existing code in the editor.
4. Paste the entire contents of `trend_compass_1h.pine`.
5. Click **Add to chart**.

### Data Requirements

- **4-Hour, Daily, and Weekly data access** required for HTF EMA pulls and 52-week calculations. Standard on all TradingView plans.
- **Sufficient 1H bar history** for percentile calculations (300+ bars default). Standard for most liquid instruments.

### Chart Settings

No session-based logic. Chart timezone settings do not affect calculations. Apply to any chart timezone.

---

## Indicator Components

### EMA Ribbon (10/21/50)

Computed locally on 1H bars. On a 1H chart with ~6.5 bars per RTH session:

- **10 EMA**: ~1.5 trading days. Captures very short-term intraday momentum shifts.
- **21 EMA**: ~3.2 trading days. The intermediate trend on the 1H timeframe.
- **50 EMA**: ~7.7 trading days. The primary structural trend line — roughly equivalent to a single trading week.

The **EMA Stack** assessment incorporates the Daily 200 EMA as the anchor:

| State | Condition | Interpretation |
|-------|-----------|---------------|
| Full Bullish Stack | 1H Fast > Mid > Slow, all above D200 EMA | All timeframes aligned bullish |
| Partial Bull | 1H ribbon bullish, but below D200 | Developing uptrend, not yet structural |
| Full Bearish Stack | 1H Fast < Mid < Slow, all below D200 EMA | All timeframes aligned bearish |
| Partial Bear | 1H ribbon bearish, but above D200 | Developing downtrend or pullback |
| Compressed | EMA spread < 0.2% | Trend transition (tighter threshold than 4H due to 1H bar scale) |
| Mixed | Any other | No clear alignment |

### HTF EMAs (4H 50, Daily 50, Daily 200, Weekly 50)

Four institutional-grade reference lines pulled from higher timeframes:

| Line | Color | Style | Purpose |
|------|-------|-------|---------|
| 4H 50 EMA | Green (#66BB6A) | Step line, thin | Immediate HTF bridge. ~31 trading days smoothed. |
| Daily 50 EMA | Cyan (#26C6DA) | Step line, thick | Intermediate structure. Swing traders key off this. |
| Daily 200 EMA | Orange (#FF6D00) | Solid, thick | The institutional dividing line. Primary structural anchor. |
| Weekly 50 EMA | Purple (#E040FB) | Step line, thick | Macro trend. ~1 year smoothed. |

All four are individually toggleable. The Daily 200 EMA is used as the anchor in both the EMA Stack assessment and the composite trend score.

### Trend Phase Classifier

Identical six-state system to the other variants with wider crossover detection windows (5/8 bars) to account for the higher bar frequency on 1H:

| Phase | Conditions | Interpretation |
|-------|-----------|----------------|
| **EMERGING** | ADX < 25, ADX rising, EMAs separating | New trend forming |
| **ACCELERATING** | ADX > 20, ADX rising | Trend gaining momentum |
| **MATURE** | ADX > 30, ADX flat | Trend intact, momentum plateauing |
| **EXHAUSTING** | ADX declining from > 30 | Trend losing steam |
| **CONSOLIDATING** | ADX < 20, EMAs compressed | Range-bound |
| **REVERSING** | Recent EMA crossovers detected | Trend change underway |

### Composite Trend Score

Same weighted architecture as the Daily and 4H variants.

**Structural Conditions (2x weight):**

| # | Condition | Bullish (+2) | Bearish (-2) |
|---|-----------|-------------|-------------|
| 1 | 1H EMA Stack | Fast > Mid > Slow | Fast < Mid < Slow |
| 2 | Price vs Daily 200 EMA | Above | Below |
| 3 | Price vs Weekly 50 EMA | Above | Below |

**Confirmation Conditions (1x weight):**

| # | Condition | Bullish (+1) | Bearish (-1) |
|---|-----------|-------------|-------------|
| 4 | ADX + DI Direction | ADX > 20 AND DI+ > DI- | ADX > 20 AND DI- > DI+ |
| 5 | RSI Position | RSI > 55 | RSI < 45 |
| 6 | MACD vs Signal | Above signal | Below signal |
| 7 | OBV Slope | Positive | Negative |
| 8 | 1H 50 EMA Slope | Positive | Negative |

**Weighted max: +/-11.** Score interpretation and IMPROVING / STABLE / FADING tracking identical to other variants.

Note: The 4H 50 EMA and Daily 50 EMA are display-only reference levels. They are not scored directly — their influence is captured indirectly through the EMA stack assessment and the slope condition.

### Divergence Detection (RSI + MACD)

Same four-state system as other variants but with the fastest pivot parameters in the family:

- **Default pivots**: 3 left / 2 right (vs 4/2 on 4H, 5/3 on Daily). Detects divergences at the earliest possible confirmation point.
- **Decay window**: 30 bars (~4.6 trading days on 1H). Divergences expire after roughly one trading week.
- **Visual**: RSI divergences = dashed lines (green/red). MACD divergences = dotted lines (cyan/purple).

The 3L/2R configuration means a pivot is confirmed just 2 hours after the actual swing point, making this the most responsive divergence detector in the Trend Compass family.

### Key Structural Levels

**Prior Day H/L/C** — blue dashed/dotted lines. The most immediately relevant reference for 1H entries and exits.

**Prior Week H/L/C** — purple dashed/dotted lines. Weekly structural pivots for multi-day swing context.

**52-Week High / Low** — pulled from the Daily timeframe via `request.security()` for accuracy.

### All Other Components

ATR regime, BB Width percentile, OBV, relative volume, Fibonacci, and Ichimoku are functionally identical to the 4H variant. The only calibration differences are the percentile lookbacks (300 bars on 1H ≈ 46 trading days) and the Fibonacci lookback (150 bars ≈ 23 trading days).

---

## Dashboard

19-row compact table:

| Row | Field | Content |
|-----|-------|---------|
| 0 | Header | "Trend Compass - 1h" (merged, purple accent bar) |
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
| 16 | D 50 EMA | ABOVE/BELOW + 4H 50 alignment indicator |
| 17 | W 50 EMA | Weekly trend alignment |
| 18 | 52w Pos | Position percentage + high/low values |

Row 16 is unique to the 1H variant — it shows Daily 50 EMA position alongside the 4H 50 EMA alignment indicator.

---

## Alerts

12 alert conditions (identical to other variants):

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
| 4H 50 EMA | Green | #66BB6A |
| Daily 50 EMA | Cyan | #26C6DA |
| Daily 200 EMA | Deep orange | #FF6D00 |
| Weekly 50 EMA | Purple | #E040FB |
| Prior Day levels | Blue | #42A5F5 |
| Prior Week levels | Purple | #AB47BC |
| 52-Week levels | Salmon | #FF8A65 |
| RSI div lines | Green (bull) / Red (bear) | Dashed |
| MACD div lines | Cyan (bull) / Purple (bear) | Dotted |

---

## Tuning Guide

### EMA Ribbon

The 10/21/50 defaults work well on 1H for intraday swing context. Alternatives:

- **8/13/34**: Fibonacci-based. More responsive.
- **13/34/89**: Very conservative for 1H. Filters out most intraday noise.
- **5/13/21**: Aggressive. Useful for faster swing trading, but more whipsaw.

### Divergence Pivot Bars

Default 3L/2R provides the fastest detection in the Trend Compass family. The 2-bar right delay means divergences confirm ~2 hours after the actual pivot.

- **2L/1R**: Extremely fast. High noise. Not recommended without additional confluence.
- **4L/2R**: Matches the 4H variant. Fewer, more significant divergences.
- **5L/3R**: Matches the Daily variant. Very conservative on 1H.

### ATR / BB Width Percentile Lookback

Default 300 bars (~46 trading days). Alternatives:

- **150 bars** (~23 days): More responsive.
- **500 bars** (~77 days): Very long context spanning ~3.5 months.

### Fibonacci Lookback

Default 150 bars (~23 trading days). Alternatives:

- **50 bars** (~8 days): Short-term intraday swing.
- **100 bars** (~15 days): Moderate, roughly two trading weeks.
- **300 bars** (~46 days): Broader multi-week structure.

### Ichimoku Parameters

Standard 9/26/52/26. On 1H bars:

- Tenkan (9): ~1.4 trading days
- Kijun (26): ~4.0 trading days
- Senkou B (52): ~8.0 trading days
- Displacement (26): ~4.0 trading days forward/back

### EMA Compression Threshold

Default 0.2%. Tighter than the 4H variant (0.3%) because 1H EMA values are naturally closer together. Increase to 0.25-0.3% if too many periods classify as COMPRESSED.

### EMA Slope Threshold

Default 0.003. Controls whether the 50 EMA slope is classified as positive or negative. Increase to 0.005 if firing too easily; decrease to 0.002 if too restrictive.

---

## Architecture Notes

- **`request.security()` budget**: 7 calls total: 4H 50 EMA, Daily 200 EMA, Daily 50 EMA, Weekly 50 EMA, prior day H/L/C (tuple), prior week H/L/C (tuple), 52-week high/low (tuple). Well within the 40-call limit.
- **Loops**: Two loops (ATR percentile, BB width percentile) over 300-bar default lookback. Both lightweight, run once per bar.
- **Object counts**: Lines for key levels and divergences, labels for Fibonacci, one table for dashboard. Well within limits.
- **Dashboard**: Only rendered on `barstate.islast`.
- **Golden/Death Cross detection**: Performed on the Daily HTF EMA values. May lag by one 1H bar within the same daily candle.
- **52-Week H/L accuracy**: Pulled from Daily timeframe, identical to the 4H variant.

---

## Known Limitations

- **1H-optimized**: All parameters and HTF mappings are calibrated for 1-hour bars. For other timeframes, use the appropriate variant.
- **7 `request.security` calls**: The most of any Trend Compass variant. Still within TradingView limits but reduces headroom when combining with other indicators.
- **Golden/Death Cross detection on HTF data**: Can lag by one 1H bar within the triggering daily candle.
- **52-Week H/L via `request.security`**: Requires 252 bars of daily history. Standard for liquid instruments.
- **OBV on 1H bars**: Aggregation depends on extended hours setting. Trend direction is reliable; absolute values may differ from daily OBV.
- **Divergence detection lag**: 2-bar right pivot = ~2 hours of confirmation delay. Fastest in the family but not instantaneous.
- **Higher divergence frequency**: 3L/2R on 1H produces more signals than other variants. Use trend phase and composite score for filtering.
- **Percentile loop computation**: 300-bar lookback is the largest in the family. Computationally trivial but marginally more than the 4H variant's 200-bar loops.
