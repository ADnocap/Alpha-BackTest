
# Alpha Backtesting Engine for Russell 1000

## Overview

This engine evaluates the performance of alpha signals (predictive vectors) on US large-cap equities by constructing optimal portfolios that maximize expected returns while maintaining strict neutrality constraints. The system supports multiple trading frequencies and incorporates realistic market microstructure considerations.

## Core Concept

The fundamental premise is simple: given an alpha vector $`\boldsymbol{\alpha}_t \in \mathbb{R}^n`$ where:
- $`\alpha_{t,i} > 0`$ indicates a bullish signal for stock $`i`$
- $`\alpha_{t,i} < 0`$ indicates a bearish signal for stock $`i`$  
- $`n`$ is the number of stocks in the investable universe

The engine constructs portfolio positions $`\boldsymbol{h}_t`$ that maximize the expected profit from the alpha signal while respecting neutrality and risk constraints.

## Mathematical Framework

### Objective Function

At each rebalancing period $`t`$, the portfolio optimization problem is:

```math
\max_{\boldsymbol{h}_t} \quad \boldsymbol{h}_t^\top \boldsymbol{\alpha}_t - \lambda \cdot \mathrm{Cost}(\boldsymbol{h}_t, \boldsymbol{h}_{t-1})
```

Where:

- $`\boldsymbol{h}_t = (h_{t,1}, \ldots, h_{t,n})^\top`$ is the position vector (in dollars or shares)
- $`\boldsymbol{\alpha}_t`$ is the alpha forecast for the next period
- $`\lambda`$ is the risk aversion parameter
- $`\mathrm{Cost}(\cdot)`$ captures transaction costs and market impact

### Neutrality Constraints

#### Market Neutrality

```math
\sum_{i=1}^n h_{t,i} = 0
```

The portfolio must be dollar-neutral (equal long and short exposure).

#### Sector Neutrality

For each sector $`s \in \mathcal{S}`$:

```math
\sum_{i \in \mathrm{sector}(s)} h_{t,i} = 0
```

Ensures zero net exposure to each GICS sector.

#### Factor Neutrality

For common risk factors (size, value, momentum, etc.):

```math
\boldsymbol{h}_t^\top \boldsymbol{f}_k \approx 0, \quad \forall k \in \mathcal{F}
```

Where $`\boldsymbol{f}_k`$ is the factor loading vector for factor $`k`$.

### Transaction Cost Model

```math
\mathrm{Cost}(\boldsymbol{h}_t, \boldsymbol{h}_{t-1}) = \sum_{i=1}^n \left[ c_i \cdot |h_{t,i} - h_{t-1,i}| + \gamma_i \cdot (h_{t,i} - h_{t-1,i})^2 \right]
```

Where:

- $`c_i`$ is the linear transaction cost (commissions + fixed spread)
- $`\gamma_i`$ captures market impact (proportional to trade size squared)

### Position and Leverage Constraints

```math
|h_{t,i}| \leq h_{\max,i} \quad \forall i \quad \text{(position limits)}
```

```math
\sum_{i=1}^n |h_{t,i}| \leq L \quad \text{(leverage limit)}
```

### Trading Frequencies
- **Hourly**: High-frequency alpha signals, suitable for microstructure-based strategies
- **Daily**: Standard frequency for most quantitative strategies
- **Weekly**: Lower turnover, reduced transaction costs
- **Monthly**: Long-term factor strategies

### Risk Management
- **Maximum position sizing** per stock (default: 2% of portfolio)
- **Gross leverage limits** (default: 2x, i.e., 100% long + 100% short)
- **Turnover constraints** to manage trading costs
- **Stop-loss mechanisms** for individual positions

### Neutralization Hierarchy
1. **Market neutral**: Dollar-neutral by construction
2. **Sector neutral**: 11 GICS Level 1 sectors
3. **Industry neutral** (optional): 24 GICS Level 2 industry groups  
4. **Factor neutral** (optional): Fama-French factors, momentum, volatility

### Market Microstructure Modeling
- **Bid-ask spread costs**: Based on historical spreads or stock liquidity
- **Market impact**: Square-root or linear models calibrated to average daily volume
- **Slippage**: Random execution noise around VWAP
- **Trading capacity constraints**: Max percentage of daily volume

### Performance Attribution

Decomposes PnL into:

- **Alpha contribution**: $`\sum_t \boldsymbol{h}_t^\top \boldsymbol{r}_{t+1}`$ where $`\boldsymbol{r}_{t+1}`$ are realized returns
- **Factor exposures**: Unintended bets on systematic factors
- **Transaction costs**: Explicit and implicit costs
- **Timing effects**: Differences between intended and executed trades

## Performance Metrics

### Return Metrics
- **Cumulative Returns**: Total strategy performance over backtest period
- **Annualized Return**: $`(1 + R_{\text{total}})^{252/T} - 1`$ for daily trading
- **Excess Return**: Strategy returns minus risk-free rate

### Risk-Adjusted Metrics

**Sharpe Ratio**:

```math
\mathrm{SR} = \frac{\mathbb{E}[R_t - r_f]}{\sigma(R_t - r_f)} \times \sqrt{252}
```

**Information Ratio**:

```math
\mathrm{IR} = \frac{\mathbb{E}[R_t - R_{t}^{\text{benchmark}}]}{\sigma(R_t - R_{t}^{\text{benchmark}})} \times \sqrt{252}
```

**Calmar Ratio**:

```math
\mathrm{Calmar} = \frac{\text{Annualized Return}}{\text{Maximum Drawdown}}
```

### Risk Metrics
- **Maximum Drawdown**: Largest peak-to-trough decline
- **Value at Risk (VaR)**: 95% and 99% confidence levels
- **Conditional VaR (CVaR)**: Expected loss beyond VaR threshold
- **Volatility**: Annualized standard deviation of returns

### Trading Metrics

**Turnover**: As percentage of portfolio

```math
\mathrm{TO}_t = \frac{1}{2}\sum_{i=1}^n |h_{t,i} - h_{t-1,i}|
```

- **Holding Period**: Average time a position is maintained
- **Win Rate**: Percentage of profitable trades
- **Profit Factor**: Gross profits / Gross losses

### Alpha Quality Metrics

**IC (Information Coefficient)**: Correlation between alpha predictions and realized returns

```math
\mathrm{IC}_t = \mathrm{corr}(\boldsymbol{\alpha}_t, \boldsymbol{r}_{t+1})
```

- **IC Stability**: Standard deviation of rolling IC
- **Alpha Decay**: How quickly alpha signal loses predictive power
```
