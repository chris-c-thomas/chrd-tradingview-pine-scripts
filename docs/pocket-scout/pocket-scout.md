# Pocket Scout — SPY/QQQ Mobile Intraday

Mobile-first intraday directional bias overlay purpose-built for iPhone
viewing of liquid US equity ETFs.

---

## Design Philosophy

The desktop indicators in this suite (Intraday Analyst, Trend Compass,
SPY 0DTE Scalper) use 18-26 row dashboards optimized for full-screen
viewing and multi-chart layouts. On an iPhone in portrait, a 20-row table
at normal text size consumes roughly 60% of the chart — the fix is not
shrinking text (illegible at a glance) but **ruthless information
hierarchy**.

Pocket Scout compresses directional context into **8 rows at `size.huge`
text** so the full picture reads in under 2 seconds with one glance,
leaving ~70% of the screen for price action. The `Huge` default sizing,
merged 2-cell banner row, and vivid color coding are all mobile-first
decisions that would be overkill on desktop.

If you can't make a directional decision in 2 seconds with one glance,
the dashboard failed.

---

## Scope

Ticker-agnostic within liquid US equity ETFs:

| Symbol Family | Vol Index (auto) |
|---|---|
| SPY, DIA | CBOE:VIX |
| QQQ, TQQQ, SQQQ, QLD, PSQ | CBOE:VXN |
| IWM, TNA, TZA | CBOE:RVX |
| Any other symbol | Falls back to CBOE:VIX (or user override) |

Intended timeframes: **1m / 5m / 15m**. The HTF Anchor EMA adapts:
1m → 15m, 5m → 1h, 15m → 4h. On daily+ charts most components
degrade gracefully but the tool is not designed for that use case.

---

## Components

### EMA Ribbon (9 / 21 / 50)

Three EMAs. Bullish alignment = Fast > Mid > Slow; Bearish = Fast < Mid < Slow.
No cloud fill (deliberate — clouds clutter small screens).

### Session VWAP

Auto-anchored per session via `ta.vwap(hlc3)`. No standard deviation bands
(mobile clutter discipline). Position measured in **ATR units** rather than
raw dollars so distance is scale-invariant across SPY vs QQQ vs IWM.

### HTF Anchor EMA (adaptive)

The slow EMA pulled from a higher timeframe provides structural context
for the current chart. `lookahead_off` prevents repainting.

| Chart TF | HTF Anchor |
|---|---|
| ≤ 1m | 15m |
| ≤ 5m | 1h |
| ≤ 15m | 4h |
| Higher | Daily |

### RSI (14) + MACD (12/26/9)

Single-line combined momentum readout on the dashboard. RSI value + MACD
direction arrow. The dashboard color reflects the **sum** of both momentum
scoring contributions.

### Volatility Index Slope

Current vol index level + intraday % change from today's open. Falling
vol index = bullish bias for equities (scoring inverts). 0.5% deadband
prevents flip-flopping around the zero line.

### Prior Day H/L/C + Session HOD/LOD

Five reference levels plotted with `plot.style_linebr` so they break
cleanly instead of extending across prior bars. The **Level Proximity**
row reports the single nearest level in ATR distance:

| ATR Distance | Label |
|---|---|
| ≤ 0.25 | `AT {level}` (yellow) |
| ≤ 0.75 | `NEAR {level}` (cyan) |
| > 0.75 | `MID RANGE` (dim) |

---

## Scoring Engine

Six conditions. Structural conditions count 2x in weighted mode, 1x in
equal mode.

**Structural (2x weighted):**

| # | Condition | Bull (+2) | Bear (-2) |
|---|---|---|---|
| 1 | EMA Stack | Fast > Mid > Slow | Fast < Mid < Slow |
| 2 | Price vs VWAP | Above | Below |
| 3 | Price vs HTF Anchor | Above | Below |

**Confirmation (1x always):**

| # | Condition | Bull (+1) | Bear (-1) |
|---|---|---|---|
| 4 | RSI | > 55 | < 45 |
| 5 | MACD | Line > Signal | Line < Signal |
| 6 | Vol Index Slope | Falling (< -0.5%) | Rising (> +0.5%) |

**Weighted max: ±9. Equal max: ±6.**

### Classification

| Score | Label | Banner |
|---|---|---|
| ≥ +STRONG | `STRONG BULL` | Solid green |
| ≥ +MODERATE | `MOD BULL` | Translucent green |
| -MOD < s < +MOD | `NEUTRAL` | Gray |
| ≤ -MODERATE | `MOD BEAR` | Translucent red |
| ≤ -STRONG | `STRONG BEAR` | Solid red |

Default thresholds: STRONG = 6, MODERATE = 3 (weighted mode).
For equal mode, drop STRONG to 4.

---

## Dashboard Layout

Eight rows, two columns, rows 0-1 merged full-width:

| Row | Left Col | Right Col | Notes |
|---|---|---|---|
| 0 | `PKT SCOUT  SPY  5m` (merged) | — | Header, dark slate bg |
| 1 | `▲  STRONG BULL  +7/9` (merged) | — | **Banner**, vivid colored bg |
| 2 | Price | Day % | Color = sign of day change |
| 3 | `VWAP ABOVE` | `+0.42 ATR` | Distance in ATRs |
| 4 | `EMA 9/21/50` | `BULLISH` | Stack state |
| 5 | `Momentum` | `RSI 62.3  MACD ▲` | Combined line |
| 6 | `VIX` | `14.2 ▼ -3.1%` | Level + intraday slope |
| 7 | `Level` | `NEAR PDH` | Nearest in ATR units |

### Sizing

| Size | Use Case |
|---|---|
| `Normal` | Desktop cross-check |
| `Large` | iPad |
| `Huge` (default) | **iPhone portrait** |

Default position is **Top Right** because TradingView's mobile toolbar
consumes ~80pt at the bottom of the iPhone screen. On desktop you can
flip to Bottom Right if preferred.

---

## `request.security()` Budget

| # | Call | Purpose |
|---|---|---|
| 1 | HTF Anchor EMA | Structural bias condition |
| 2 | Prior Day tuple [H, L, C, open] | Reference levels + day % |
| 3 | Vol index close (current TF) | Vol level |
| 4 | Vol index daily open | Vol intraday slope |

**Total: 4 calls.** Well under the 40-call limit with headroom for
running multiple instances simultaneously.

---

## Alerts

| # | Alert | Condition |
|---|---|---|
| 1 | STRONG BULL | `biasScore` crosses above `(STRONG - 0.5)` |
| 2 | STRONG BEAR | `biasScore` crosses below `-(STRONG - 0.5)` |
| 3 | Crossed PDH | `close` crosses above Prior Day High |
| 4 | Crossed PDL | `close` crosses below Prior Day Low |

All include `{{ticker}}` and `{{interval}}` placeholders for multi-symbol
routing.

---

## What Pocket Scout Does NOT Do

Deliberate exclusions to preserve the mobile information budget:

- No divergence detection (use Intraday Analyst on desktop)
- No TTM Squeeze (use Intraday Analyst on desktop)
- No ADX / DI tiers (use Trend Compass on desktop)
- No NYSE TICK (network latency on mobile + adds security calls)
- No signal arrows / entry labels (use SPY 0DTE Scalper on desktop)
- No Opening Range boxes (visual clutter on small screens)
- No multi-timeframe EMA stack (Trend Compass territory)
- No prior week / 52w levels (not intraday-actionable)

Pocket Scout is a **pocket companion**, not a replacement for the desktop
toolkit. Use it on your phone when you're away from the desk. Use the
full indicators when you're at the desk.

---

## Limitations

1. **Vol index requires data subscription** — VIX is free on TradingView;
   VXN and RVX may require a CBOE data plan.
2. **Session VWAP on daily+** — `ta.vwap` on non-intraday timeframes
   resets per daily bar and is not meaningful. VWAP-based scoring
   contributes 0 in that case.
3. **HOD/LOD during extended hours** — Session tracking uses `time("D")`
   which includes ETH. If you prefer RTH-only HOD/LOD, disable extended
   hours on the chart.
4. **Not a signal generator** — This is a bias/context tool. It tells
   you "which way is the wind blowing," not "enter now." Combine with
   your own entry/exit rules.

---

## Version History

### v1.0.0 (initial release)

- Adaptive 1m / 5m / 15m timeframe support
- Symbol auto-detect for SPY/QQQ/IWM/DIA families
- 6-condition weighted scoring engine (structural 2x / confirmation 1x)
- 8-row mobile-optimized dashboard with merged banner row
- Session VWAP in ATR-normalized distance
- HTF Anchor EMA with TF-adaptive mapping
- Session HOD/LOD tracking
- Level proximity detection in ATR units
- 4 high-signal alerts
