# Soy Put Hedge — Black-Scholes Pricing & Backtesting

Commodity hedge strategy for a Brazilian soy producer using a European put option priced via Black-Scholes, with parameters calibrated on a training set and validated on an out-of-sample backtest period.

## Overview

This project models a **price protection strategy** for soy denominated in BRL, covering the full pipeline from data collection to hedge performance evaluation. The put strike is set at the average BRL price over the training period, and the option is priced using volatility estimated exclusively from that same window.

Key results:

| Metric | Value |
|--------|-------|
| Strike K | R$ 5,847.83 |
| Put premium | R$ 373.62 (6.88% of spot) |
| Net floor guaranteed | R$ 5,474.21 |
| Probability of exercise (N(-d2)) | 55.7% |
| Realized outcome | Put expired worthless (+10.5% market) |
| Floor respected | ✅ Yes |
| Break-even price | R$ 5,806.28 |

> The hedge decision was rational ex-ante: 55.7% probability of exercise at inception. The market rose 10.5% during the test period, so the full premium (R$ 373.62) was the realized cost.

## Methodology

**Data collection**
- Soy futures (ZS=F) and USD/BRL exchange rate (BRL=X) downloaded via `yfinance`
- BRL price computed daily: `V_BRL = Price_USD × FX_rate`
- 501 aligned observations (April 2024 – April 2026)

**Train/test split**
- Chronological split: 18 months training (377 sessions), 6 months test (124 sessions)
- All parameters calibrated exclusively on the training set — no lookahead

**Parameterization (training set)**
- Daily log-returns: `r_t = ln(P_t / P_{t-1})`
- Annualized volatility: `σ = σ_daily × √252 = 22.61%`
- Strike: `K = mean(Soja_BRL)` over training period = R$ 5,847.83
- S0 = last price of training period = R$ 5,432.66 → put born in-the-money

**Option pricing — Black-Scholes**
- European put: `P = K·e^(−rT)·N(−d2) − S·N(−d1)`
- Risk-free rate: SELIC annualized = 12.9% p.a.
- T = 124 trading days / 252 = 0.49 years

**Backtesting**
- Put marked to market daily using Black-Scholes with decreasing T
- Producer result tracked as: `S_t + Put_value_t − Premium`
- Floor validation confirmed across all 124 sessions

**Scenario analysis**
- Payoff computed across 6 scenarios from −20% to +20% relative to S0
- Break-even identified at R$ 5,806.28

## Key Findings

The put provided full downside protection throughout the test period — the floor of R$ 5,474.21 was never breached. In a severe drop scenario (−20%), the hedge would have protected R$ 1,128/bushel. The cost of protection (6.23% of the realized sale price) was the only drag in the favorable market outcome that materialized.

## Stack

- Python 3.x
- `yfinance` — commodity and FX data
- `scipy.stats` — Black-Scholes normal distribution
- `numpy`, `pandas` — data processing
- `matplotlib` — payoff diagrams and backtesting charts

## Structure

```
soy-put-hedge-black-scholes-backtesting/
├── notebook.ipynb          # Full pipeline (5 blocks)
├── requirements.txt
└── README.md
```

## How to Run

```bash
pip install -r requirements.txt
jupyter notebook notebook.ipynb
```

> Data is fetched live at runtime — no static files required.

## Context

Developed as part of the Applied Computing course at **FGV EESP** (MSc in Data Science, 2026).
