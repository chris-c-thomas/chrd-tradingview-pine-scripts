# Market Monitor 15m v1.0.0

15-minute optimized directional-bias overlay for multi-chart watchlist monitoring.

---

## Design Philosophy

The 15-minute variant is the keystone of the intraday reconnaissance layer. It serves as the direct bridge between multi-day structural trends (evaluated by the Trend Compass) and high-frequency execution (handled by the 0DTE Scalpers).

Unlike the 1m and 5m Market Monitors which react aggressively to morning chop, the 15-minute timeframe absorbs intraday noise and establishes the definitive "session-level" bias. With exactly 26 bars per standard RTH session, this timeframe divides cleanly into institutional 1H and 4H boundaries, making its signals highly structured and statistically reliable.

---

## Key Differences from 5-Minute Variant

| Feature | 5-Minute Variant | 15-Minute Variant |
|---------|-----------------|-------------------|
| Bars per Session | 78 bars | **26 bars** |
| Anchor EMA | 15-Minute EMA | **1-Hour EMA** (4-bar fraction) |
| Opening Range | First 3-6 bars | **First 2 bars (30 min)** |
| Session Phasing | 5m intervals | **15m intervals** |
| ATR Pctl Lookback| 100 bars (~1.2 days) | **260 bars (~10 days)** |
| Goal | Rapid bias updates | **Stable session trend** |

---

## Components

### Weighted Scoring Engine (-11 to +11)

A composite directional bias score generated from 8 continuous assessments.

**Structural Alignment (2x Weight):**

1. **EMA Array** (9/21/50 alignment)
2. **VWAP Position** (Price vs Session VWAP)
3. **1H Anchor EMA** (Immediate institutional trend direction)

**Confirmation / Momentum (1x Weight):**

1. **RSI State** (>55 or <45)
2. **DMI Trend** (ADX > 20 and DI+ vs DI-)
3. **Squeeze Momentum** (Direction of the TTM Squeeze histogram)
4. **Short-Term Momentum** (Price vs 9 EMA)
5. **Market Breadth** (NYSE TICK > +200 or < -200)

### Session Phasing

Tracks the progression of the 26-bar session to establish temporal context:

- **OPEN DRIVE:** Bars 1-2 (09:30 - 10:00)
- **MORNING:** Bars 3-8 (10:00 - 11:30)
- **MIDDAY:** Bars 9-18 (11:30 - 14:00)
- **POWER HOUR:** Bars 19-25 (14:00 - 15:45)
- **CLOSE:** Bar 26 (15:45 - 16:00)

### TTM Squeeze Detection

Evaluates Bollinger Bands (2.0 SD) inside Keltner Channels (1.5 SD) over a 20-bar length.

- **COMPRESSED:** Volatility is dropping.
- **FIRED:** Volatility is expanding; a trend move is initiating.
- **EXPANDED:** Normal volatility state.

### NYSE TICK Integration (USI:TICK)

Measures aggregate market breadth (buying vs selling pressure across all NYSE equities). Extremely valuable on the 15-minute timeframe to distinguish a genuine index-wide trend day from a low-conviction range day. *Requires TradingView data access to `USI:TICK`.*

---

## Usage in Workflow

Load this indicator onto a grid layout (e.g., 6 charts: SPY, QQQ, IWM, AAPL, NVDA, TSLA) all set to 15-minute intervals.

1. **Scan the Backgrounds:** Look for charts with green or red background tints (indicates a bias score of +7 or -7, respectively).
2. **Evaluate the Triad:** If SPY shows a +9 on the 15m Market Monitor, switch to your dedicated SPY chart.
3. **Check Compass:** Verify the 1H and 4H `Trend Compass` agree with the 15m bias.
4. **Execute:** Wait for the `spy_0dte_scalper_15min` to trigger an entry signal in the direction of the bias.

---

## Alerts

Five static alert conditions for webhook or native TradingView routing:

1. **Strong Bullish Bias:** Score reaches +7 or higher.
2. **Strong Bearish Bias:** Score reaches -7 or lower.
3. **Bias Crossed Neutral:** Score crosses zero, indicating a momentum shift.
4. **Volume Spike:** Current 15m bar volume is > 2x the 20-bar SMA.
5. **Squeeze Fired:** TTM Squeeze transitions from Compressed to Fired.

---

## Limitations and Technical Notes

- **VWAP is Intraday Only:** VWAP logic evaluates to `na` on daily timeframes. The scoring engine gracefully falls back to the 50 EMA as the structural baseline if deployed on a Daily chart.
- **TICK Data:** If `USI:TICK` is unavailable in your region/feed, the scoring engine handles the `na` value cleanly and limits the maximum possible score to +/- 10.
- **`request.security()` Budget:** Uses exactly two calls: 1H Anchor EMA and US TICK data. Safe to deploy across 8+ charts simultaneously without hitting limits.
