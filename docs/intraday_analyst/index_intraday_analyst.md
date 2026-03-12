# Index Intraday Analyst

**Platform**: TradingView
**Language**: Pine Script v6
**Chart**: CBOE Cash Indices (SPX, NDX, RUT, DJX) — 1-minute, 5-minute, 10-minute, 15-minute (adaptive)
**Theme**: Dark theme (vibrant, high-contrast palette)

---

## Overview

Intraday overlay for CBOE cash indices — SPX, NDX, RUT, DJX, and any other exchange-listed cash index that lacks volume data on TradingView.

CBOE cash indices have **no volume data**. This indicator is architected entirely around that constraint. Every component is either price-derived or pulled from external companion symbols (VIX/VXN/RVX, NYSE TICK). Volume-dependent tools (VWAP, OBV, relative volume, footprint) are replaced with purpose-built alternatives.

---

## Supported Indices + Volatility Index Mapping

| Cash Index | Description | Vol Index Symbol |
|:-----------|:------------|:----------------|
| SPX | S&P 500 | CBOE:VIX |
| NDX | Nasdaq-100 | CBOE:VXN |
| RUT | Russell 2000 | CBOE:RVX |
| DJX | Dow Jones | CBOE:VXD |

The volatility index symbol is a configurable input. When switching from SPX to NDX, change `CBOE:VIX` to `CBOE:VXN` in the Volatility Index settings group. Everything else adapts automatically.

---

## Relationship to the Equity Intraday Analyst (Stocks/ETFs)

| Component | Index Intraday Analyst (this script) | Equity Intraday Analyst |
|:----------|:------------------------------------|:-----------------|
| Volume data | None (CBOE constraint) | Yes (full volume stack) |
| Fair value reference | Session TWAP (time-weighted) | Session VWAP (volume-weighted) |
| Volume confirmation | Not available | OBV trend (CONFIRMING / DIVERGING) |
| Volume regime | ATR percentile only | Relative Volume (SPIKE / ABOVE / NORMAL / DRY) |
| Prior day fair value | Not available | Prior Day VWAP Close |
| Volatility index | ON by default, configurable symbol | OFF by default (opt-in for SPY/QQQ) |
| TICK breadth | ON by default | ON by default |
| Scoring max (weighted) | +/-13 | +/-15 (adds OBV condition) |
| Target instruments | CBOE cash indices (no volume) | Any equity or ETF with volume |

**Use the Equity Intraday Analyst** for: SPY, QQQ, IWM, NVDA, TSLA, GOOGL, AAPL, MSFT, AMD, or any instrument with volume data.

**Use the Index Intraday Analyst** (this script) for: SPX, NDX, RUT, DJX, or any instrument without volume data.

---

## Volume Substitution Strategy

Rather than disabling volume-based features, this indicator substitutes them:

| Replaced Component | Index Substitute | Rationale |
|:-------------------|:-----------------|:----------|
| Session VWAP | **Session TWAP** with std dev bands | Running mean of typical price (HLC/3) anchored to session start. Equal time weighting instead of volume weighting. |
| OBV trend | **Volatility index context** (level + slope + intraday change) | VIX/VXN/RVX provides fear/complacency context that OBV gives on volume-capable instruments. |
| Relative volume | **ATR percentile + BB width percentile** | Volatility regime classification replaces volume regime classification. |
| Volume footprint | **Stochastic K/D** | Fast momentum confirmation filling the footprint delta slot. |

---

## Components

### Session TWAP and Bands

**Time-Weighted Average Price** — running mean of typical price (HLC/3) anchored to each RTH session start. Standard deviation bands provide extension context.

TWAP treats every bar equally regardless of participation. In reality, more volume transacts at certain prices — but since we have no volume data, time-weighting is the best available proxy for session fair value.

### Volatility Index Context Module

The companion volatility index (VIX for SPX, VXN for NDX, etc.) provides context that raw price action cannot.

**Three dimensions tracked:**

1. **Level** — LOW (<15) / NORMAL (15-20) / ELEVATED (20-25) / HIGH (25-30) / EXTREME (>30)
2. **Slope** — RISING (fear increasing, bearish) / FLAT / FALLING (fear decreasing, bullish)
3. **Intraday change %** — today's vol index movement vs daily open

**Scoring integration**: Contributes +/-1. Bullish: vol index below elevated AND slope falling. Bearish: above elevated AND slope rising.

### All Other Components

EMA Ribbon, HTF Anchor EMA, RSI (with divergence detection), MACD, ADX/DMI, Stochastic, ATR regime, BB width percentile, TTM Squeeze, NYSE TICK, session levels, phase detection, regime classifier, signal engine, and dashboard are architecturally identical across both Intraday Analyst variants. See the Equity Intraday Analyst documentation for detailed descriptions.

---

## Scoring Engine

### Structural Conditions (2x weight when weighted mode enabled)

| # | Condition | Bullish (+2) | Bearish (-2) |
|:--|:----------|:------------|:------------|
| 1 | EMA Stack | Fast > Mid > Slow | Fast < Mid < Slow |
| 2 | Price vs Session TWAP | Above | Below |
| 3 | Price vs HTF Anchor EMA | Above | Below |

### Confirmation Conditions (1x weight always)

| # | Condition | Bullish (+1) | Bearish (-1) |
|:--|:----------|:------------|:------------|
| 4 | ADX + DI Direction | ADX > threshold AND DI+ > DI- | ADX > threshold AND DI- > DI+ |
| 5 | RSI Position | RSI > 55 | RSI < 45 |
| 6 | MACD vs Signal | Above signal | Below signal |
| 7 | NYSE TICK | SMA > bull threshold | SMA < bear threshold |
| 8 | Squeeze Momentum | Positive (recently fired) | Negative |
| 9 | Volatility Index | Low + falling | Elevated + rising |
| 10 | 50 EMA Slope | Positive | Negative |

**Weighted max: +/-13. Equal max: +/-10.**

---

## `request.security()` Budget

| # | Call | Purpose |
|:--|:-----|:--------|
| 1 | Prior Day High (Daily) | Key level |
| 2 | Prior Day Low (Daily) | Key level |
| 3 | Prior Day Close (Daily) | Key level |
| 4 | HTF Anchor EMA | Structural scoring |
| 5 | NYSE TICK close (1m) | Breadth |
| 6 | NYSE TICK SMA (1m) | Smoothed breadth |
| 7 | Vol Index close (current TF) | Vol index level |
| 8 | Vol Index daily open | Vol index intraday change |

Max 8 calls. Disabling TICK saves 2, disabling vol index saves 2.

---

## Known Limitations

1. **No volume data**: CBOE cash indices provide no volume. TWAP, not VWAP. No OBV, no relative volume, no footprint. By design.
2. **TWAP is not VWAP**: Equal time weighting vs volume weighting. TWAP is a reasonable proxy for session fair value but lacks volume-weighted precision.
3. **Volatility index granularity**: VIX/VXN update frequently but are derived from options prices. During fast moves, the vol index may lag slightly.
4. **Pre-market levels require extended hours**: Data availability depends on your TradingView plan.
5. **TICK data RTH only**: NYSE TICK returns N/A outside 9:30-16:00 ET.
6. **No backtest capability**: `indicator()`, not `strategy()`.
7. **RVX/VXD availability**: Depending on your TradingView data plan, some secondary volatility indices may have limited data.
