# GMXV2-Risk-Solvency-Analysis
DuneSQL-driven framework for real-time GMX V2 risk management by tracking OI, Solvency, and Trader PnL.

[View Live Dashboard on Dune](https://dune.com/kemasan/gmx-v2-risk-and-solvency-analysis) 

### 🎯 Project Overview
In GMX V2, Liquidity Providers (LPs) act as the "House." They provide the collateral that traders use to gamble on price movements. If traders win too much, the LPs lose money. If the market becomes too one-sided, the pool faces "Imbalance Risk."

### The Business Question: How risky are individual GMX V2 markets today, and how much stress can the system handle if traders win at scale?

This project rebuilds the financial state of each market using on-chain data. It converts raw blockchain events into clear risk indicators.
The result is a real-time system that shows whether liquidity pools can stay solvent during strong market moves.

### 📈 Key Metrics & Why They matters
  #### MARKET ACTIVITY
  #### Open Interest (OI): Total value of active, outstanding bets (Longs + Shorts).
  High OI amplifies volatility and liquidation risk.
  #### Stress Solvency Ratio: Pool Liquidity ÷ (Net Exposure × Shock).
  Simulates losses under price shocks (5%, 10%, 20%)
  #### Market Skew: Directional imbalance between Longs and Shorts.
  Strong skew increases directional risk.
  #### Pool Liquidity: Total collateral backing each market.
  Determines how much loss the pool can absorb.

  #### PROFITABILITY
  #### Rolling Trader Wins (7D): 7-day sum of positive realized PnL.
  Measures recent pressure on LP capital.
  #### Liquidity vs. Trader Wins: Pool Liquidity ÷ 7D Trader Wins.
  Shows whether recent profits are sustainable. 

  #### Risk & Solvency
  #### Stress Solvency Ratio: Pool Liquidity ÷ (Net Exposure × Shock)
  Simulates losses under price shocks (5%, 10%, 20%).
  #### Stress Liability: Estimated loss under simulated shock.
  Quantifies downside risk in USD terms.
  #### Risk Regime: Classification based on solvency (High Risk, Watchlist, Healthy, No Exposure).
  Makes risk visible at a glance.
  #### System-Wide OI at Risk: % of total OI in insolvent markets.
  Measures protocol-level tail risk.

### 🛠️ Technical Implementation (DuneSQL)
This project is built in DuneSQL using GMX V2 on-chain event tables. The main goal is to reconstruct daily market state (OI, liquidity, stress metrics) from raw updates.
#### Joins (to connect the data correctly)  
Used for:
    - Join Open Interest events to market metadata (markets_data) to get market_name, long_token, short_token.
    - Join reconstructed OI state with reconstructed pool liquidity state on: day + chain + market
    - Join trader wins (from position_decrease) into the same daily market frame.

##### CTEs (to make logic readable + modular)
Each metric is built step-by-step:
    - params: stores dashboard inputs (shock size, start date).
    - calendar: creates a daily date spine (so missing days don’t break charts).
    - oi_ CTEs*: build daily OI per market and forward-fill.
    - pool_ CTEs*: build daily pool liquidity and forward-fill.
    - trader_ CTEs*: compute daily + rolling trader wins.
    - final SELECT: calculates stress solvency + risk regime labels.

#### Window Functions (to “rebuild state” from events)
Used mainly for forward-filling and rolling sums:
    - LAST_VALUE(... IGNORE NULLS) OVER (...) → carries the latest known value forward to days with no events.
    - SUM(...) OVER (ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) → computes 7-day rolling trader wins.

#### Filtering with WHERE (to keep queries fast + relevant)
Used for:
    - block_time >= start_ts → limits analysis to recent history (default 90 days).
    - token filter in pool updates → only keeps long_token and short_token for each market.

#### CASE WHEN logic (if-then-else rules)
Used for:
    - Chain naming normalization (avalanche_c → Avalanche)
    - Safe division (NULLIF(...,0) to avoid divide-by-zero)
    - Handling “no exposure” markets (OI = 0)
    - Creating risk categories:
        - High Risk (solvency < 1)
        - Watchlist (solvency 1–3)
        - Healthy (solvency > 3)

### 📚 Research Influence
Informed by GMX community research and DeFi risk discussions.
All metrics and models were independently reconstructed, validated, and implemented using on-chain data.

### 📊 Why the Solvency Ratio matters for GMX V2
The Solvency Ratio is the ultimate indicator of whether the Liquidity Providers are at risk of a "run on the bank." In GMX V2, the isolated pool design means that if one pool (like DOGE/USD) becomes insolvent, it does not affect the others, making this metric vital for choosing which pools to provide liquidity for.

[GMX V2 Open Interest and Pool Value relationship](https://www.youtube.com/watch?v=6PEn6iEuFGA)

This video explains how GMX V2 manages the relationship between pool collateral and open interest limits, which is the foundation of the solvency ratio calculated in this project.
