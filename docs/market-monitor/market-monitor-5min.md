# Market Monitor - 5m

5-minute optimized directional-bias overlay for multi-chart watchlist monitoring.

---

## Design Philosophy

This indicator extends the general-purpose Market Monitor with features specifically tuned for 5-minute chart monitoring. It retains the "glanceable bias score" paradigm while adding weighted scoring, session awareness, market breadth, volatility compression detection, and richer reference levels. It remains a **context and bias tool** — no signal generation.

The 5-minute timeframe produces ~78 RTH bars per session, providing enough granularity for swing-scalp context without the noise of 1-minute charts. This variant is purpose-built for that cadence.

---

## Key Differences from General-Purpose Market Monitor

| Feature | General (v1.0) | 5-Minute (v1.0) |
|---------|---------------|-----------------|
| Bias score range | -5 to +5 (equal weight) | -11 to +11 weighted (or -8 to +8 equal) |
| Scoring mode | All conditions 1x | Structural 2x / Confirmation 1x (configurable) |
| Structural conditions | N/A | EMA alignment, VWAP position, Anchor EMA |
| Confirmation conditions | N/A | RSI, DI, HTF, TICK, Squeeze momentum |
| Session phases | None | Open Drive / Morning / Midday / Power Hour / Close |
| NYSE TICK | Not included | Integrated with 6-tier classification |
| TTM Squeeze | Not included | BB/KC compression detection with momentum direction |
| ATR regime | Raw value only | Percentile-ranked: LOW / NORMAL / HIGH / EXTREME |
| Opening Range | Not included | Configurable duration (default 15 min / 3 bars) |
| Session HOD/LOD | Not included | Real-time tracking with dashboard position |
| Prior Day VWAP | Not included | Latched from previous session close |
| Anchor EMA | Not included | 15m 50 EMA plotted as institutional reference |
| Bias trend | Not tracked | IMPROVING / STABLE / FADING via 3-bar SMA comparison |
| HTF mapping | Auto-adaptive | Fixed to 15m (optimal for 5m primary) |
| Dashboard rows | 12 | 18 |
| Alert count | 6 | 9 |

---

## Components

### EMA Ribbon (9 / 21 / 50)

Three EMAs with a cloud fill between fast and slow. Default lengths match the general-purpose variant for cross-chart consistency.

- **Bullish alignment**: Fast > Mid > Slow (green cloud)
- **Bearish alignment**: Fast < Mid < Slow (red cloud)
- **Mixed**: Any other configuration (yellow cloud)

### VWAP + Bands

Session-anchored VWAP with configurable standard deviation bands. Auto-detects intraday timeframes and hides on Daily+ charts.

### Anchor EMA (15m 50 EMA)

The 50-period EMA from the 15-minute timeframe, plotted on the 5-minute chart as a purple line. This represents institutional-grade trend structure and serves as a key structural condition in the bias engine. It provides context that a single-timeframe EMA ribbon cannot — whether the broader 15-minute structure supports the direction visible on the 5-minute chart.

### Previous Day High / Low / Close

Static reference levels from the prior daily bar, pulled via `request.security`. Institutional order flow clusters around these levels.

### Prior Day VWAP

The closing VWAP value from the previous session, latched at each new RTH session open. Provides a volume-weighted fair value reference from the prior day. Plotted as purple cross markers.

### Session HOD / LOD

Real-time tracking of the session's high and low during RTH. Plotted as circle markers. The dashboard shows whether price is AT HOD, AT LOD, or MID-range.

### Opening Range

Captures the high and low of the first N minutes of RTH (configurable, default 15 minutes = 3 bars on 5m). Once the OR window closes, levels are drawn as horizontal lines for the remainder of the session. The dashboard shows whether price is ABOVE OR, BELOW OR, or IN OR.

### Session Phase Detection

Classifies the current time-of-day into one of six phases based on Eastern Time:

| Phase | Time (ET) | Dashboard Color | Character |
|-------|-----------|----------------|-----------|
| OPEN DRIVE | 9:30 - 10:00 | Orange | High volatility, initial directional push |
| MORNING | 10:00 - 11:30 | Green | Trend continuation or reversal, best signal quality |
| MIDDAY | 11:30 - 14:00 | Gray | Low volume, mean reversion, chop risk |
| POWER HOUR | 14:00 - 15:30 | Cyan | Institutional repositioning, renewed volume |
| CLOSE | 15:30 - 16:00 | Yellow | Final positioning, 0DTE gamma effects |
| PRE/POST | Outside RTH | Gray | Extended hours |

### TTM Squeeze

Detects Bollinger Band compression inside Keltner Channels — a precursor to volatility expansion:

- **SQUEEZE ON**: BB inside KC, compression building (red dots)
- **FIRED**: BB expanded outside KC this bar (green dots)
- **OFF**: Normal volatility (gray dots)

Squeeze momentum direction (from linear regression of midline deviation) feeds into the bias engine when a squeeze has fired within the last 3 bars.

### NYSE TICK Index

Real-time market breadth pulled from `USI:TICK`. Six-tier classification:

| Range | Label | Interpretation |
|-------|-------|---------------|
| > 1000 | EXTREME BULL | Broad-based buying, institutional demand |
| > 500 | BULLISH | Healthy bullish breadth |
| > 200 | LEAN BULL | Slight positive breadth |
| -200 to 200 | NEUTRAL | Balanced |
| < -500 | BEARISH | Healthy bearish breadth |
| < -1000 | EXTREME BEAR | Broad-based selling |

Contributes to the bias score when TICK exceeds the bullish/bearish thresholds.

### ATR Regime Classification

ATR is ranked against its own recent history using a percentile calculation over a configurable lookback (default 100 bars):

| Percentile | Regime | Interpretation |
|-----------|--------|---------------|
| >= 90th | EXTREME | Outsized moves, widen stops, reduce size |
| >= 70th | HIGH | Active trending, normal-to-wide stops |
| >= 30th | NORMAL | Standard conditions |
| < 30th | LOW | Compressed volatility, squeeze likely building |

---

## Bias Engine — Weighted Scoring

The 5-minute variant introduces a **weighted scoring system** that differentiates structural market conditions from confirmation signals. Structural conditions represent the primary trend framework; confirmation conditions validate the direction.

### Structural Conditions (2x weight when weighted mode enabled)

| # | Condition | Bullish | Bearish |
|---|-----------|---------|---------|
| 1 | EMA Alignment | Fast > Mid > Slow | Fast < Mid < Slow |
| 2 | Price vs VWAP | Above VWAP | Below VWAP |
| 3 | Price vs Anchor EMA | Above 15m 50 EMA | Below 15m 50 EMA |

### Confirmation Conditions (1x weight always)

| # | Condition | Bullish | Bearish |
|---|-----------|---------|---------|
| 4 | RSI Bias | RSI > 55 | RSI < 45 |
| 5 | DI Spread | DI+ > DI- | DI+ < DI- |
| 6 | HTF Confirmation | Price above 15m slow EMA | Price below 15m slow EMA |
| 7 | NYSE TICK | TICK > bullish threshold | TICK < bearish threshold |
| 8 | Squeeze Momentum | Positive momentum (fired recently) | Negative momentum (fired recently) |

### Score Ranges

**Weighted mode** (default): Max = 3x2 + 5x1 = +/-11

| Score | Label | Interpretation |
|-------|-------|---------------|
| +7 to +11 | STRONG BULL | High-conviction bullish bias — look for long setups on pullbacks |
| +4 to +6 | LEAN BULL | Moderate bullish lean — trend present, watch for confirmation |
| +1 to +3 | WEAK BULL | Slight bullish tilt — low conviction |
| 0 | NEUTRAL | Choppy / no directional bias — avoid |
| -1 to -3 | WEAK BEAR | Slight bearish tilt |
| -4 to -6 | LEAN BEAR | Moderate bearish lean |
| -7 to -11 | STRONG BEAR | High-conviction bearish bias |

**Equal mode**: All conditions 1x. Max = +/-8. Thresholds shift proportionally.

### Bias Trend Tracking

The indicator compares the current bias score to its 3-bar simple moving average to classify the bias trajectory:

- **IMPROVING**: Bias is increasing (moving more bullish or less bearish)
- **STABLE**: Bias is holding steady
- **FADING**: Bias is decreasing (moving more bearish or less bullish)

This answers the question "is the wind shifting?" without requiring a bias sign change.

---

## Dashboard Layout

Compact 3-column, 18-row table optimized for small chart tiles:

| Row | Field | Col 1 | Col 2 | Col 3 |
|-----|-------|-------|-------|-------|
| 0 | Header | "Market Monitor - 5m" (merged) | — | — |
| 1 | Bias | Label | Score / Max | Bias Label |
| 2 | Trend | Label | IMPROVING/STABLE/FADING | WTD/EQL |
| 3 | Price | Label | Current Price | Day Change % |
| 4 | EMA | Label | Trend State | Lengths |
| 5 | RSI | Label | Value | State |
| 6 | ADX | Label | Value | Strength |
| 7 | DI | Label | +DI / -DI | Dominance |
| 8 | ATR | Label | Value (% of Price) | Regime |
| 9 | VWAP | Label | Distance | Extended/Near |
| 10 | RelVol | Label | Ratio | Spike/Above/Below |
| 11 | HTF | Label | Above/Below/At | EMA Reference |
| 12 | Squeeze | Label | State | Momentum Dir |
| 13 | TICK | Label | Value | Classification |
| 14 | PD | Label | Position vs PD H/L | — |
| 15 | OR | Label | Position vs OR | OR High / Low |
| 16 | Session | Label | HOD/LOD Position | H/L Values |
| 17 | PD VWAP | Label | Value | Above/Below |

Default: **Normal** size, **Bottom Right** positioning.

---

## Alerts

Nine pre-built alert conditions:

| # | Alert | Condition |
|---|-------|-----------|
| 1 | Strong Bullish Bias | Score >= +7 (weighted) or +5 (equal) |
| 2 | Strong Bearish Bias | Score <= -7 (weighted) or -5 (equal) |
| 3 | Bias Crossed Neutral | Score crosses zero |
| 4 | Volume Spike (>2x) | Relative volume > 2x average |
| 5 | RSI Overbought | RSI > 70 |
| 6 | RSI Oversold | RSI < 30 |
| 7 | Squeeze Fired | TTM Squeeze transitions from ON to OFF |
| 8 | TICK Extreme Bull | NYSE TICK > 1000 |
| 9 | TICK Extreme Bear | NYSE TICK < -1000 |

All alerts include `{{ticker}}` and `{{interval}}` placeholders for multi-symbol alert routing.

---

## `request.security` Call Budget

| Call | Timeframe | Purpose |
|------|-----------|---------|
| 1 | Daily | Previous Day High / Low / Close |
| 2 | 15m | HTF Slow EMA (used for HTF confirmation + Anchor EMA plot) |
| 3 | Current | NYSE TICK Index (`USI:TICK`) |
| 4 | Daily | Day Open (dashboard day change %) |

**Total: 4 calls** — well within TradingView's 40-call limit with substantial headroom for running 8+ instances simultaneously.

---

## Performance Notes

- **Object counts**: Max labels: 200, max lines: 300, max boxes: 100. Well under Pine Script limits.
- **Computation**: One loop (ATR percentile ranking over configurable lookback, default 100 bars). All other calculations are vectorized. The percentile loop runs once per bar and is lightweight.
- **Dashboard**: Only rendered on `barstate.islast` to avoid per-bar table overhead.
- **Squeeze/TICK**: Optional modules that can be disabled to reduce `request.security` calls and computation if running many instances.

---

## Configuration Guide

### Scoring Mode

**Weighted (default)**: Structural conditions score 2 points each, confirmation conditions score 1 point each. This emphasizes that price being above VWAP and the 15m anchor EMA is more significant than RSI being slightly bullish. Recommended for most users.

**Equal**: All conditions score 1 point. Simpler to interpret. Use if you prefer a more democratic scoring system.

### Background Tint Threshold

Default is 4 (weighted mode). This triggers the subtle green/red background when the bias score reaches roughly 30% of maximum. Increase to 7+ for a "strong conviction only" tint, or decrease to 2 for more sensitive tinting.

### Opening Range Duration

Default 15 minutes (3 bars on 5m). For more conservative OR definitions, increase to 30 minutes (6 bars). The 15-minute default captures the initial volatility flush and provides actionable breakout/breakdown levels for the rest of the session.

### ATR Percentile Lookback

Default 100 bars (~1.3 sessions on 5m RTH). Increase to 200+ for a longer-term volatility context that spans multiple days. Decrease to 50 for a more responsive regime classification.

### NYSE TICK Thresholds

Default +/-500. For SPY/QQQ monitoring, these defaults work well. For individual stocks, TICK is less directly relevant — consider disabling it or widening thresholds to +/-800.

### TTM Squeeze Parameters

Default BB 20/2.0, KC 20/1.5 are standard TTM Squeeze settings. The KC multiplier of 1.5 (vs the typical 1.5) provides a balanced squeeze detection — not too sensitive, not too conservative.

---

## Limitations

- **5-minute optimized**: While it will technically run on other timeframes, the session phase detection, OR duration defaults, HTF mapping (fixed to 15m), and scoring parameters are calibrated for 5-minute bars. For other timeframes, use the general-purpose Market Monitor.
- **VWAP is session-anchored**: Only meaningful on intraday charts. Auto-hides on Daily+ but the bias engine substitutes the slow EMA.
- **TICK data availability**: `USI:TICK` is NYSE-specific. May not be available on all TradingView plans or for non-US market hours.
- **Prior Day VWAP latching**: The pdVwap value is captured at the new session boundary from the previous bar's VWAP. If the indicator is loaded mid-session, the value will be `na` until the next session start.
- **Bias score is a heuristic**: It quantifies directional lean, not signal quality. A score of +9 means strong structural alignment — it does not mean "enter now."
- **ATR percentile loop**: Runs a `for` loop of configurable length (default 100). On very large lookback values (500+), this adds marginal per-bar computation, but should be negligible for 8 instances.
- **Previous day levels use `lookahead_on`**: Appropriate for reference levels but means they appear before the prior day closes during replay/backtesting.
- **HTF confirmation uses `lookahead_off`**: Introduces 1-bar lag on the 15m timeframe by design to avoid future data leakage.
