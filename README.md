# Statistical Arbitrage in US Energy Equities
## Cointegration-Based Pairs Trading with Dynamic Hedge Ratio Estimation

**Georgia Institute of Technology | Spring 2025**

---

## Summary

Five-version research project developing a statistical arbitrage strategy 
on US energy sector equities (COP, MPC, PSX, CVX, XOM, VLO). 
Strict IS/OOS split: training 2015–2020, test 2021–2024, 
parameters frozen before OOS evaluation.

| Version | Method | IS Sharpe | OOS Sharpe |
|---|---|---|---|
| V1 | Static Engle-Granger + fixed hedge | 0.704 | 0.013 |
| V3 | Causal Kalman filter hedge | 1.023 | -0.111 |
| V4 | Kalman + HMM regime filter | 0.452 | -0.111 |
| V5 | Kalman + rolling cointegration gate | 0.423 | **+0.465** |

---

## Methodology

### Universe & Data
6 US energy sector equities, adjusted daily close prices via yfinance, 
2015–2024. Log-transformed. Strict temporal train/test split at 2020-12-31.

### Pair Selection (V1)
Engle-Granger cointegration test on all 15 pairs, IS only. 
2 pairs significant at p<0.05: COP/MPC (p=0.044), COP/PSX (p=0.050). 
ADF test confirmed spread stationarity. COP/MPC selected as primary pair 
given stronger economic motivation (crude producer vs petroleum refiner).

### Hedge Ratio Estimation
- **V1:** Static OLS beta estimated once on IS data (β=0.686)
- **V3/V5:** Causal Kalman filter — state-space model tracking 
  time-varying [β, α] updated daily using only past observations. 
  Beta drifted from ~1.0 (2015) to ~0.70 (2021) to ~0.82 (2024), 
  explaining V1's OOS failure as hedge ratio misspecification.

### Signal Construction
60-day rolling z-score of Kalman spread. Entry at ±2σ, exit at ±0.5σ. 
Positions shifted by 1 day to prevent look-ahead bias.

### Regime Filter (V5)
Rolling 126-day Engle-Granger cointegration p-value computed causally 
at each timestep. New positions only opened when p<0.10. 
Reduced time-in-market from 100% to 24.6% OOS, avoiding the 
2022-2023 energy sector dislocation driven by Russia-Ukraine supply shock.

---

## Key Findings

1. **Static models fail under regime change.** V1's OOS collapse was 
   due to hedge ratio drift, not cointegration breakdown.

2. **Kalman filter alone is insufficient.** Dynamic hedging improved IS 
   performance dramatically (Sharpe 1.023) but worsened OOS — the 
   2022-2023 energy dislocation created a trending regime that no 
   mean-reversion model survives.

3. **Real-time cointegration gating is the critical upgrade.** 
   Rolling p-value filter correctly identified non-cointegrated periods, 
   reducing max drawdown from -28% to -6.74% and recovering OOS Sharpe 
   to 0.465.

4. **HMM regime detection failed OOS.** The IS-trained HMM classified 
   99.6% of OOS days as tradeable — demonstrating that 
   geopolitically-driven structural breaks cannot be captured by 
   volatility-based regime models trained on pre-shock data.

---

## Limitations

- **Survivorship bias:** Universe restricted to currently-listed equities
- **Small universe:** 6 tickers, 1 primary pair — limited diversification
- **Execution assumptions:** End-of-day fills assumed; no market impact
- **Sparse OOS trades:** 24.6% time-in-market means few OOS observations
- **Single sector:** Results may not generalize beyond energy equities

---

## Further Work

- Expand to banks, airlines, semiconductors sectors
- Portfolio of pairs with correlation-adjusted position sizing
- Permutation test for statistical significance of IS Sharpe
- Stop-loss mechanism during drawdown periods
- Higher-frequency data to increase OOS trade count

---

## Stack
Python 3.13 | pandas | numpy | statsmodels | hmmlearn | matplotlib | yfinance
