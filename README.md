# GMXV2-Risk-Solvency-Analysis
DuneSQL-driven framework for real-time GMX V2 risk management by tracking OI, Solvency, and Trader PnL.

[View Live Dashboard on Dune](https://dune.com/kemasan/gmx-v2-risk-and-solvency-analysis) 

### 🎯 Project Overview
In GMX V2, Liquidity Providers (LPs) act as the "House." They provide the collateral that traders use to gamble on price movements. If traders win too much, the LPs lose money. If the market becomes too one-sided, the pool faces "Imbalance Risk."

### The Business Question: "What is the current risk profile of our liquidity pools, and do we have enough collateral to remain solvent if traders win big?"

This project provides a real-time monitoring solution for GMX V2 isolated GM pools, allowing stakeholders to identify market anomalies and capital efficiency at a glance.

### 📈 Key Metrics & Why They matters
  #### Open Interest (OI): Total value of active, outstanding bets (Longs + Shorts).
  Why it matters: Risk Exposure: High OI means high volatility. If BTC moves 10%, a $10M OI means $1M is shifting in/out of the pool.
  #### Cumulative PnL: The historical "Scoreboard" of trader profits vs. losses.
  Why it matters: Pool Growth: Negative = House wins (Pool grows). Positive = Traders win (Pool shrinks).
  #### Solvency Ratio: The "Safety Buffer" ($Total Assets \div Total Trader Debt$).
  Why it matters: Survival: A ratio of 10.0 means for every $1 traders win, $10 is in the vault. A ratio near 1.0 is a "Red Alert.
  #### Market Skew: Directional imbalance between Longs and Shorts.
  Why it matters: Exposure: If 90% are Long and BTC moons, the House loses. A 50/50 skew is the goal for "Delta-Neutral" stability.

### 🛠️ Technical Implementation (DuneSQL)
This project is built using the DuneSQL framework, leveraging advanced window functions to reconstruct the historical state of the protocol from event logs.


### 📊 Why the Solvency Ratio matters for GMX V2
The Solvency Ratio is the ultimate indicator of whether the Liquidity Providers are at risk of a "run on the bank." In GMX V2, the isolated pool design means that if one pool (like DOGE/USD) becomes insolvent, it does not affect the others, making this metric vital for choosing which pools to provide liquidity for.

[GMX V2 Open Interest and Pool Value relationship](https://www.youtube.com/watch?v=6PEn6iEuFGA)

This video explains how GMX V2 manages the relationship between pool collateral and open interest limits, which is the foundation of the solvency ratio calculated in this project.
