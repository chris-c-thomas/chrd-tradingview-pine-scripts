# Changelog

All notable changes to this project are documented in this file.

---

## (2026-03-02) -- Alert System Fix (All Trend Compass Variants)

- Fixed Pine Script v6 string handling errors in the Alerts section across Daily, 4H, and 1H variants.
- Replaced invalid `nz()` usage on string series (`trendPhase`, `atrRegime`) with proper `na()` guards to prevent compilation failures.
- Refactored all `alertcondition()` message arguments to use compile-time constant strings (required by Pine v6).
- Removed dynamic string concatenation from alert messages (phase names, ATR regime values, BB percentile values), which Pine does not allow inside `alertcondition()`.

**Technical details**:

- `string prevPhase = na(trendPhase[1]) ? "CONSOLIDATING" : trendPhase[1]`
- `string prevAtrRegime = na(atrRegime[1]) ? "NORMAL" : atrRegime[1]`
- All alert messages now use static templates with `{{ticker}}` and `{{interval}}` placeholders only.

**Impact**:

- Restores full compilation compatibility in Pine v6.
- Prevents runtime alert creation failures in TradingView.
- No changes to scoring, calculations, visuals, or alert logic conditions — message text only.

---

## (2026-03-02) -- Trend Compass 1H

- New 1-Hour variant of the Trend Compass family, bridging the gap between intraday Market Monitors and the 4-Hour Trend Compass.
- Fastest feedback loop in the Trend Compass family: catches intraday trend shifts and divergences that the 4H is too slow to detect.
- Intended use: same-day trend assessment before deploying scalping tools, intraday divergence monitoring, and 4H 50 EMA support/resistance evaluation.

**Architecture**:

- EMA Ribbon (10/21/50) computed on 1H bars with 0.2% compression threshold (tighter than 4H's 0.3%).
- Four HTF EMA layers: 4H 50 EMA (new bridge level unique to 1H variant), Daily 50 EMA, Daily 200 EMA (anchor), Weekly 50 EMA — all individually toggleable.
- Composite Weighted Trend Score (-11 to +11) from 8 conditions: 3 structural (2x weight) + 5 confirmation (1x). Same scoring architecture as Daily and 4H variants.
- Trend Phase Classifier: EMERGING / ACCELERATING / MATURE / EXHAUSTING / CONSOLIDATING / REVERSING.
- RSI/MACD divergence detection with 3L/2R pivots (fastest in family), 30-bar decay (~4.6 trading days).
- ATR/BB width percentile ranking over 300-bar lookback (~46 trading days).
- 52-week high/low via Daily `request.security()`.

**1H-specific calibration**:

- EMA slope threshold: 0.003 (vs 0.005 on 4H, 0.01 on Daily) for smaller per-bar moves.
- Cross detection windows: fast/mid 5 bars, mid/slow 8 bars (wider in bar count, equivalent calendar time).
- Divergence pivots: 3L/2R (vs 4L/2R on 4H, 5L/3R on Daily) with ~2-hour confirmation delay.
- Percentile lookbacks: 300 bars (vs 200 on 4H, 100 on Daily) to maintain ~46 days of context.
- Fibonacci lookback: 150 bars (~23 trading days) vs 100 on 4H, 50 on Daily.
- 4H 50 EMA as unique structural reference: fills gap between local 1H 50 EMA (~7.7 days) and Daily 50 EMA (~50 days). Plotted as green step line.

**Dashboard and alerts**:

- 19-row dashboard (largest in family): adds combined D50/4H50 alignment row and 4H 50 EMA display.
- 12 alert conditions (same set as Daily and 4H): phase change, strong bull/bear, neutral cross, RSI/MACD divergences (4), golden/death cross, ATR regime change, BB extreme compression.
- 7 `request.security` calls (most in family): 4H 50 EMA, D50 EMA, D200 EMA, W50 EMA, prior day H/L/C, prior week H/L/C, 52w H/L.

**Key levels**:

- Prior Day H/L/C + Prior Week H/L/C reference levels.
- Fibonacci retracement (optional, default OFF): 150-bar lookback.
- Ichimoku Cloud (optional, default OFF): standard 9/26/52/26.

---

## (2026-02-27) -- Trend Compass Daily + 4H

- New strategic trend assessment indicator family for multi-day/multi-week trend evaluation on any equity, ETF, or index.
- Ticker-agnostic design: no hardcoded symbols, no session-specific logic. Works on NVDA, AAPL, GOOGL, SPY, QQQ, IWM, or any liquid instrument.
- No signal generation — pure trend context, strength assessment, and structural positioning.

**Daily variant (TC-D)**:

- EMA Ribbon (10/21/50) with dynamic cloud fill and 200 EMA anchor (local).
- Weekly 50 EMA via `request.security()` for macro trend context.
- Trend Phase Classifier: EMERGING / ACCELERATING / MATURE / EXHAUSTING / CONSOLIDATING / REVERSING based on ADX value, ADX slope, EMA spread, and crossover detection.
- Composite Weighted Trend Score (-11 to +11) with 8 conditions: 3 structural (2x weight) + 5 confirmation (1x).
- Score trend tracking (IMPROVING / STABLE / FADING) via 5-bar SMA comparison.
- ADX slope analysis: RISING / FLAT / FALLING trend strength momentum.
- 50 EMA slope + slope acceleration (second derivative) for trend curvature analysis.
- RSI divergence detection (bullish, bearish, hidden bull, hidden bear) with visual dashed line segments on price chart. Pivot-based (5L/3R default), 15-bar decay.
- MACD (12/26/9) divergence detection with dotted line segments. Distinct visual style from RSI divergences.
- OBV trend confirmation via 21-EMA slope: CONFIRMING / DIVERGING / NEUTRAL.
- Relative volume (20-bar SMA baseline) with SPIKE / ABOVE / NORMAL / DRY classification.
- ATR percentile ranking (100-bar lookback) with LOW / NORMAL / HIGH / EXTREME regime classification.
- Bollinger Band width percentile (100-bar lookback) for compression/breakout detection.
- 52-week high/low via `ta.highest(252)` / `ta.lowest(252)` with position-in-range percentage.
- Prior Week H/L/C and Prior Month H/L/C reference levels.
- Fibonacci retracement (optional, default OFF): 38.2% / 50% / 61.8% auto-calculated from lookback range.
- Ichimoku Cloud (optional, default OFF): standard 9/26/52/26 with Kumo fill.
- 17-row dashboard with configurable position and size.
- 12 alert conditions: phase change, strong bull/bear, neutral cross, RSI/MACD divergences (4), golden/death cross (50/200), ATR regime change, BB extreme compression.
- 3 `request.security` calls (Weekly EMA, prior week H/L/C, prior month H/L/C).

**4-Hour variant (TC-4H)**:

- Same architecture as Daily with recalibrated parameters for 4-hour bars.
- Anchor EMA = Daily 200 EMA via `request.security()` (not local — 200 bars on 4H = 33 days, insufficient for institutional weight).
- Three HTF EMA layers: Daily 50 EMA, Daily 200 EMA, Weekly 50 EMA — all individually toggleable.
- 52-week H/L pulled from Daily timeframe via `request.security()` for accuracy.
- Key levels: Prior Day H/L/C + Prior Week H/L/C (vs Week + Month on Daily).
- Divergence pivots tuned faster: 4L/2R default (vs 5L/3R), 20-bar decay (~3.5 days vs 15 days).
- Percentile lookbacks widened to 200 bars (~33 trading days) to cover equivalent calendar time.
- EMA compression threshold tightened to 0.3% (vs 0.5%) for 4H bar scale.
- EMA slope threshold tightened to 0.005 (vs 0.01) for smaller per-bar moves.
- Crossover detection windows widened: fast/mid 4 bars, mid/slow 6 bars (vs 3/5 on Daily).
- Golden/Death Cross alerts fire on Daily 50/200 EMA values from `request.security()`.
- 18-row dashboard (adds D 200 EMA + W 50 EMA rows, separate 52w position row).
- 12 alert conditions (same set as Daily).
- 6 `request.security` calls (D200 EMA, D50 EMA, W50 EMA, prior day H/L/C, prior week H/L/C, 52w H/L).

---

## (2026-02-27) -- Market Monitor 5m

- New 5-minute optimized Market Monitor variant with weighted directional bias scoring.
- Weighted scoring engine: structural conditions (EMA alignment, VWAP position, Anchor EMA) score 2x; confirmation conditions (RSI, DI, HTF, TICK, Squeeze momentum) score 1x. Configurable toggle between weighted and equal modes.
- Bias score range expanded to +/-11 (weighted) or +/-8 (equal) vs the general-purpose monitor's +/-5.
- Session phase detection: Open Drive / Morning / Midday / Power Hour / Close classification based on Eastern Time.
- TTM Squeeze integration with BB/KC compression detection, three-state classification, momentum direction, and bars-since-fired tracking.
- NYSE TICK Index integration with six-tier breadth classification and configurable thresholds.
- ATR regime classification via percentile ranking (LOW / NORMAL / HIGH / EXTREME) over configurable lookback.
- Prior Day VWAP latched from previous session close and plotted as reference level.
- 15-minute Anchor EMA (50 EMA) plotted as institutional trend reference line.
- Opening Range levels with configurable duration (default 15 min / 3 bars on 5m).
- Session HOD/LOD real-time tracking with dashboard position context.
- Bias trend tracking (IMPROVING / STABLE / FADING) via 3-bar SMA comparison.
- Expanded 18-row dashboard with session phase, squeeze state, TICK readings, OR position, and session context.
- 9 alert conditions including squeeze fired, TICK extremes, and adaptive strong bias thresholds.
- 4 `request.security` calls total — within headroom for 8+ simultaneous instances.

---

## (2026-02-24) -- Market Monitor 1m

- General-purpose directional-bias overlay for multi-chart watchlist monitoring (6-8 charts per window).
- Timeframe-adaptive design: works from 1-minute through Daily without reconfiguration. VWAP auto-disables on Daily+ charts; bias engine swaps price anchor from VWAP to slow EMA accordingly.
- EMA Ribbon (9 / 21 / 50) with dynamic cloud fill — deliberately wider than the 0DTE scalper variants to provide meaningful trend context across timeframes.
- Session-anchored VWAP with configurable standard deviation bands and automatic intraday detection.
- Previous Day High / Low / Close reference levels via `request.security` on the Daily timeframe.
- Composite directional bias score (-5 to +5) from 5 equal-weighted binary conditions: EMA alignment, price vs VWAP/anchor, RSI bias, DI spread, and HTF EMA confirmation.
- HTF EMA confirmation layer with automatic timeframe mapping (1m→5m, 5m→15m, 15m→1H, etc.).
- Relative volume gauge (20-period SMA baseline) with SPIKE / ABOVE / BELOW classification.
- Optional background tint (94% transparency) triggered at configurable bias score threshold.
- Compact 12-row dashboard with three size options (Tiny / Small / Normal) and four corner positions.
- 6 alert conditions: strong bullish/bearish bias, bias crossed neutral, volume spike, RSI overbought/oversold.
- 3 `request.security` calls total — lightweight footprint for 8+ simultaneous instances.
- No signal generation — pure context and bias assessment. No loops, arrays, or maps; fully vectorized.

---

## (2026-02-20) -- All Scalper Variants

- TTM Squeeze detection with three-state classification and momentum histogram.
- NYSE TICK Index integration with six-tier breadth classification.
- Expanded dashboard rows for squeeze and TICK status.
- New alert conditions for squeeze fired events.
- Enhanced dynamic alert messages with squeeze and TICK context.

**1-Minute specific**: Squeeze and TICK added as optional AND-gate filters (default OFF). VWAP cross markers added.

**5-Minute specific**: Squeeze recency and TICK breadth added as scoring conditions #8 and #9 in the 5+4 engine. Level proximity added as scoring condition #7. Expanded candle pattern detection (inside bar breakout, 3-bar momentum).

**15-Minute specific**: Integrated into scoring and directional bias system with timeframe-appropriate calibration.

---

## (2026-02-18) -- Initial Release

- EMA Ribbon with dynamic cloud fill.
- Session-anchored VWAP with standard deviation bands.
- RSI, ADX/DMI, ATR calculations with dashboard display.
- Pre-market high/low, prior day H/L/C, opening range, session HOD/LOD levels.
- Five-mode regime classifier.
- Signal engine (AND-gate for 1-min, scoring for 5-min and 15-min).
- CALLS/PUTS labels with tooltips, arrow shapes, background flash.
- Real-time dashboard with color-coded status indicators.
- Dual alert system (alertcondition + dynamic alert).
- Market Monitor v1.0 with composite bias scoring (-5 to +5).
