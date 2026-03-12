# Equity Intraday Analyst

**Platform**: TradingView
**Language**: Pine Script v6
**Chart**: Any liquid US equity or ETF — 1-minute, 5-minute, 10-minute, 15-minute (adaptive)
**Theme**: Dark theme (vibrant, high-contrast palette)

---

## Overview

Ticker-agnostic intraday overlay for any liquid US equity or ETF. Works on NVDA, MSFT, TSLA, GOOGL, AAPL, AMD, SPY, QQQ, IWM, and any other instrument with volume data on TradingView.

This is the general-purpose companion to the Index Intraday Analyst (which handles the no-volume constraint of CBOE cash indices (SPX, NDX, RUT) with TWAP and enhanced price-only momentum). Both instruments have volume, VWAP, OBV, and relative volume are native to this script.

---

## Relationship to Index Intraday Analyst

| Component | Index Intraday Analyst | Equity Intraday Analyst (this script) |
|:----------|:--------------------|:-------------------------------|
| Volume data | None (CBOE constraint) | Yes (full volume stack) |
| Fair value reference | Session TWAP (time-weighted) | Session VWAP (volume-weighted) |
| Volume confirmation | Not available | OBV trend (CONFIRMING / DIVERGING) |
| Volume regime | ATR percentile only | Relative Volume (SPIKE / ABOVE / NORMAL / DRY) |
| Prior day fair value | Not available | Prior Day VWAP Close |
| VIX context | ON by default (configurable symbol) | OFF by default (opt-in for SPY/QQQ) |
| TICK breadth | ON by default | ON by default |
| Scoring max (weighted) | +/-13 | +/-15 (adds OBV condition) |
| Target instruments | CBOE cash indices (SPX, NDX, RUT) | Any equity or ETF with volume |

---

## Why One Script Instead of Two (Stocks vs ETFs)

The architectural differences between individual stocks (NVDA, TSLA) and ETFs (SPY, QQQ) are configuration-level, not architecture-level:

- Both have volume data (VWAP, OBV, relative volume all work)
- NYSE TICK is useful context for both (directly relevant for ETFs, indirect sentiment for stocks)
- VIX is most relevant for SPY/QQQ but is already an opt-in toggle
- Liquidity differences are handled by ATR-based thresholds, not separate code paths

A single script with smart defaults and toggleable modules avoids maintaining two nearly-identical scripts. Use the input settings to configure per-instrument:

**For ETFs (SPY, QQQ, IWM):** Enable VIX Context, keep TICK at 500/-500 thresholds.

**For Stocks (NVDA, TSLA, GOOGL):** Leave VIX off, widen TICK thresholds to 800/-800 (or disable if not useful for your workflow).

---

## Components

### EMA Ribbon (9 / 21 / 50)

Three EMAs with dynamic cloud fill. Bullish alignment (green), bearish (red), mixed (yellow). The 50 EMA slope (5-bar rate of change) is tracked as a scoring condition.

### HTF Anchor EMA

Higher-timeframe 50 EMA via `request.security()`. Auto-adapts: 1m->5m, 5m->15m, 10m->30m, 15m->60m. Structural scoring condition.

### Session VWAP + Bands

Real volume-weighted average price anchored to each RTH session, with configurable standard deviation bands. The key advantage over TWAP (used in the Index variant) is that VWAP weights price by actual participation — high-volume price levels have more influence on the fair-value line.

**Prior Day VWAP Close**: latched at each session boundary from the prior bar's VWAP value. Institutional fair value reference from the prior session.

### OBV Trend Confirmation

On-Balance Volume with a 20-period EMA. The slope of the OBV EMA (5-bar rate of change) is compared to price direction:

| OBV State | Condition | Interpretation |
|:----------|:----------|:--------------|
| CONFIRMING | Price and OBV slope moving same direction | Volume supports the trend |
| DIVERGING | Price and OBV slope moving opposite directions | Warning: volume contradicts price |
| NEUTRAL | No clear directional alignment | Indeterminate |

OBV slope contributes +/-1 to the scoring engine when enabled.

### Relative Volume

Current bar's volume divided by a 20-bar SMA baseline:

| Classification | Ratio | Interpretation |
|:---------------|:------|:--------------|
| SPIKE | > 2.0x | Institutional activity, breakout catalyst |
| ABOVE | > 1.2x | Healthy participation |
| NORMAL | 0.8 - 1.2x | Typical activity |
| DRY | < 0.8x | Low participation, chop risk |

Displayed in the dashboard for context. Not directly in the scoring engine (volume conditions are captured via OBV).

### RSI, MACD, ADX/DMI, Stochastic, ATR, BB Width, TTM Squeeze, NYSE TICK, VIX Context

Identical architecture to the Index Intraday Analyst. See that documentation for detailed descriptions of each component.

### Session Levels, Phase Detection, Regime Classifier

Identical to Index Intraday Analyst, with the addition of Prior Day VWAP Close as a plotted reference level.

---

## Scoring Engine

### Structural Conditions (2x weight when weighted mode enabled)

| # | Condition | Bullish (+2) | Bearish (-2) |
|:--|:----------|:------------|:------------|
| 1 | EMA Stack | Fast > Mid > Slow | Fast < Mid < Slow |
| 2 | Price vs Session VWAP | Above | Below |
| 3 | Price vs HTF Anchor EMA | Above | Below |

### Confirmation Conditions (1x weight always)

| # | Condition | Bullish (+1) | Bearish (-1) |
|:--|:----------|:------------|:------------|
| 4 | ADX + DI Direction | ADX > threshold AND DI+ > DI- | ADX > threshold AND DI- > DI+ |
| 5 | RSI Position | RSI > 55 | RSI < 45 |
| 6 | MACD vs Signal | Above signal | Below signal |
| 7 | OBV Slope | Rising | Falling |
| 8 | NYSE TICK | SMA > bull threshold | SMA < bear threshold |
| 9 | Squeeze Momentum | Positive (recently fired) | Negative |
| 10 | VIX Context | Low + falling | Elevated + rising |
| 11 | 50 EMA Slope | Positive | Negative |

### Score Ranges (weighted mode, max +/-15 with VIX enabled, +/-14 without)

| Score | Label | Interpretation |
|:------|:------|:--------------|
| +7 to max | STRONG BULL | High-conviction bullish. All major components aligned. |
| +4 to +6 | LEAN BULL | Moderate bullish lean. |
| +1 to +3 | WEAK BULL | Slight bullish tilt. |
| 0 | NEUTRAL | No directional bias. |
| -1 to -3 | WEAK BEAR | Slight bearish tilt. |
| -4 to -6 | LEAN BEAR | Moderate bearish lean. |
| -7 to min | STRONG BEAR | High-conviction bearish. |

---

## `request.security()` Budget

| # | Call | Purpose |
|:--|:-----|:--------|
| 1 | Prior Day High (Daily) | Key level |
| 2 | Prior Day Low (Daily) | Key level |
| 3 | Prior Day Close (Daily) | Key level |
| 4 | HTF Anchor EMA | Structural scoring condition |
| 5 | NYSE TICK close (1m) | Breadth (if enabled) |
| 6 | NYSE TICK SMA (1m) | Smoothed breadth (if enabled) |
| 7 | VIX close (current TF) | VIX level (if enabled) |
| 8 | VIX daily open | VIX intraday change (if enabled) |

Max 8 calls, well within the 40-call limit. All optional modules degrade gracefully when disabled.

---

## Tuning Profiles

### Conservative / Sniper

```
Min Signal Score:      9
Signal Cooldown:       10 (1m) / 5 (5m) / 3 (15m)
ADX No-Trend:          20
Signal Window:         1000-1530
VWAP Distance Filter:  2.0
Require ADX:           ON
```

### Balanced (Default)

```
Min Signal Score:      7
Signal Cooldown:       auto
ADX No-Trend:          18
Signal Window:         0945-1545
VWAP Distance Filter:  2.5
Require ADX:           ON
```

### Aggressive

```
Min Signal Score:      5
Signal Cooldown:       3 (1m) / 2 (5m) / 1 (15m)
ADX No-Trend:          15
Signal Window:         0935-1555
VWAP Distance Filter:  3.5
Require ADX:           OFF
```

---

## Known Limitations

1. **No volume = use Index variant**: If applied to SPX or any other instrument without volume, VWAP will be zero/flat, OBV will be zero, and relative volume will be meaningless. Use the Index Intraday Analyst for volumeless instruments.
2. **VWAP is session-anchored**: Only meaningful on intraday charts. Auto-hides on Daily+.
3. **TICK data RTH only**: NYSE TICK returns N/A outside 9:30-16:00 ET.
4. **VIX relevance varies**: Directly relevant for SPY, moderately for QQQ/IWM, marginally for individual stocks.
5. **No backtest capability**: `indicator()`, not `strategy()`.
6. **Single-chart focus**: No cross-symbol correlation. For multi-chart reconnaissance, use the Market Monitor family.
7. **Pre-market levels require extended hours**: Enable in chart settings.
8. **Prior Day VWAP latching**: Value captured at session boundary. If loaded mid-session, will be `na` until next session start.
