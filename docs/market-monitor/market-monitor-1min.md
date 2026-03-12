# Market Monitor - 1m

General-purpose directional-bias overlay for multi-chart watchlist monitoring.

---

## Design Philosophy

This indicator is purpose-built for the "6-8 charts per window" monitoring workflow. It prioritizes:

- **Glanceability**: A single composite bias score (-5 to +5) tells you direction instantly via the dashboard and optional background tint.
- **Low visual noise**: No signal arrows, no entry/exit markers. Just structure, trend context, and momentum state.
- **Timeframe adaptability**: Works from 1-minute through Daily. VWAP auto-disables on non-intraday charts; the bias engine swaps its anchor from VWAP to the slow EMA accordingly.
- **Lightweight footprint**: Minimal object counts, no volume profile approximation, no opening range boxes. Designed to run on 8 instances simultaneously without hitting TradingView limits.

---

## Components

### EMA Ribbon (9 / 21 / 50)

Three EMAs with a cloud fill between fast and slow. Default lengths are deliberately broader than the 0DTE scalper variants (9/21/50 vs 9/15/21) to provide meaningful trend context across timeframes without excessive whipsawing on 15m or 1H charts.

- **Bullish alignment**: Fast > Mid > Slow (green cloud)
- **Bearish alignment**: Fast < Mid < Slow (red cloud)
- **Mixed**: Any other configuration (yellow cloud)

### VWAP + Bands

Session-anchored VWAP with configurable standard deviation bands. Auto-detects intraday timeframes and hides entirely on Daily/Weekly/Monthly charts where VWAP has no meaning.

### Previous Day High / Low / Close

Always-on reference levels pulled via `request.security` on the Daily timeframe. These are the most universally relevant levels across all timeframes — institutional order flow clusters around them.

### Bias Engine

Five binary conditions scored +1 (bullish) or -1 (bearish), producing a composite score from -5 to +5:

| # | Condition | Bullish (+1) | Bearish (-1) |
|---|-----------|-------------|-------------|
| 1 | EMA Alignment | Fast > Mid > Slow | Fast < Mid < Slow |
| 2 | Price vs Anchor | Above VWAP (intraday) / Above Slow EMA (daily+) | Below VWAP / Below Slow EMA |
| 3 | RSI Bias | RSI > 55 | RSI < 45 |
| 4 | DI Spread | DI+ > DI- | DI+ < DI- |
| 5 | HTF Confirmation | Price above higher-TF slow EMA | Price below higher-TF slow EMA |

**Interpretation**:

- +3 to +5: Strong bullish bias — look for long setups on pullbacks
- +1 to +2: Lean bullish — trend present but not overwhelming
- 0: Neutral / choppy — avoid directional bias
- -1 to -2: Lean bearish
- -3 to -5: Strong bearish bias — look for short setups on bounces

### HTF EMA Confirmation

Automatically maps the current timeframe to the next-higher timeframe and pulls the slow EMA:

| Current TF | HTF Used |
|-----------|----------|
| 1m | 5m |
| 3m | 15m |
| 5m | 15m |
| 15m | 1H |
| 30m | 1H |
| 1H | 4H |
| 4H | Daily |
| Daily | Weekly |

This prevents getting caught in micro-timeframe noise when the higher structure is moving against you.

### Relative Volume

Current bar volume divided by the 20-period SMA of volume. Thresholds:

- **>2.0x**: SPIKE (green) — institutional activity likely
- **>1.2x**: ABOVE average (yellow)
- **<1.2x**: BELOW average (gray)

### Background Tint

Optional subtle background coloring that triggers when the absolute bias score meets the configured threshold (default: 2). Green tint for bullish, red for bearish, transparent for neutral. At 94% transparency, it's visible but doesn't interfere with price action reading.

---

## Dashboard Layout

Compact 3-column, 12-row table optimized for small chart tiles:

| Row | Field | Col 1 | Col 2 |
|-----|-------|-------|-------|
| 0 | Header | "Market Monitor - 1m" (merged) | — |
| 1 | Bias | Score (+/-) | Label |
| 2 | Price | Current | Day Change % |
| 3 | EMA | Trend State | Lengths |
| 4 | RSI | Value | Label |
| 5 | ADX | Value | Strength |
| 6 | DI | +DI / -DI | Dominance |
| 7 | ATR | Value | % of Price |
| 8 | VWAP/EMA Dist | Distance | Extended/Near |
| 9 | RelVol | Ratio | Label |
| 10 | HTF | Above/Below | EMA Length |
| 11 | Prev Day | Position | — |

Four size options (Tiny / Small / Normal / Large) and four corner positions. Default: **Normal** size, **Bottom Right** positioning.

---

## Recommended Timeframe Pairings

For a typical monitoring session with 6-8 charts:

| Use Case | Primary TF | HTF Confirmation |
|----------|-----------|-----------------|
| Intraday scalp watch | 1m or 5m | 5m or 15m |
| Intraday swing watch | 15m | 1H |
| Multi-day swing | 1H or 4H | 4H or Daily |
| Position / weekly review | Daily | Weekly |

The indicator self-adapts, so you can freely switch timeframes on any chart without reconfiguring.

---

## Alerts

Six pre-built alert conditions:

1. **Strong Bullish Bias** — score >= +3
2. **Strong Bearish Bias** — score <= -3
3. **Bias Crossed Neutral** — score crosses zero
4. **Volume Spike** — relative volume > 2x
5. **RSI Overbought** — RSI > 70
6. **RSI Oversold** — RSI < 30

All alerts include `{{ticker}}` and `{{interval}}` placeholders for multi-symbol alert routing.

---

## Performance Notes

- **Object counts**: Well under Pine Script limits. Max labels: 100, max lines: 200, max boxes: 50.
- **Security calls**: 3 total (`request.security` for PD levels, day open, and HTF EMA). Within TradingView's 40-call limit with substantial headroom.
- **Computation**: No loops, no arrays, no maps. Pure vectorized calculations. Should run smoothly across 8+ simultaneous instances.

---

## Limitations

- VWAP is session-anchored and only meaningful on intraday timeframes. The indicator auto-hides it on Daily+ but the bias engine compensates by substituting the slow EMA as the price anchor.
- The bias score is a directional heuristic, not a trading signal. It tells you "which way is the wind blowing" — not "enter now."
- Previous day levels use `lookahead=barmerge.lookahead_on` which is appropriate for historical reference levels but means they appear on the chart before the prior day has closed during replay/backtesting.
- HTF confirmation introduces a 1-bar lag on the higher timeframe by design (`lookahead_off`) to avoid future data leakage.
