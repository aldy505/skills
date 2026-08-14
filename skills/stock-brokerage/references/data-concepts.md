# Data Concepts — Definitions and Formulas

## Price and volume

**OHLCV** — Open, High, Low, Close, Volume. One bar per time period (day, hour, minute). The standard input for technical analysis.

**Adjusted Close** — Close price adjusted for splits and dividends. Use adjusted prices for historical analysis; use raw prices for day-to-day trading review.

## Valuation metrics

| Metric | Formula | Interpretation |
|--------|---------|----------------|
| **Market Cap** | `share_price × shares_outstanding` | Total equity value |
| **P/E (Price-to-Earnings)** | `market_cap / net_income` (TTM) | How much investors pay per dollar of earnings. Lower = cheaper relative to earnings. |
| **P/B (Price-to-Book)** | `market_cap / book_value` (total equity) | Price relative to net asset value. < 1 may signal undervaluation or distress. |
| **P/S (Price-to-Sales)** | `market_cap / revenue` (TTM) | Useful for unprofitable companies. |
| **PEG** | `P/E / EPS_growth_rate` | P/E adjusted for growth. < 1 often considered cheap. |
| **EV/EBIT** | `enterprise_value / EBIT` | Value of the whole firm (debt + equity) relative to operating earnings before interest. |
| **EV/EBITDA** | `enterprise_value / EBITDA` | Same as EV/EBIT but adds back depreciation/amortization; useful for capital-intensive businesses. |

**Enterprise Value** = `market_cap + total_debt - cash_and_equivalents`.

## Profitability metrics

| Metric | Formula | Interpretation |
|--------|---------|----------------|
| **Gross Margin** | `(revenue - COGS) / revenue` | Pricing power and production efficiency |
| **Operating Margin** | `operating_income / revenue` | Profitability after operating costs |
| **Net Margin** | `net_income / revenue` | Bottom-line profitability |
| **ROE** | `net_income / shareholders_equity` | Return on shareholder capital |
| **ROA** | `net_income / total_assets` | Efficiency of asset use |
| **ROIC** | `NOPAT / invested_capital` | Return on all capital invested (debt + equity) |

**NOPAT** = `operating_income × (1 - tax_rate)`.
**Invested Capital** = `total_assets - current_liabilities - excess_cash` (or `shareholders_equity + total_debt - cash`).

## Growth metrics

| Metric | Formula |
|--------|---------|
| **Revenue growth** | `(revenue_t - revenue_{t-1}) / revenue_{t-1}` |
| **EPS growth** | `(EPS_t - EPS_{t-1}) / abs(EPS_{t-1})` |
| **FCF growth** | `(FCF_t - FCF_{t-1}) / abs(FCF_{t-1})` |

Use TTM (trailing twelve months) or annual figures consistently. Avoid mixing quarterly and annual.

## Cash flow and quality metrics

| Metric | Formula | Interpretation |
|--------|---------|----------------|
| **Free Cash Flow (FCF)** | `operating_cash_flow - capex` | Cash available to distribute or reinvest |
| **FCF Yield** | `FCF / market_cap` | Cash return on equity value; higher = cheaper |
| **Dividend Yield** | `annual_dividend / share_price` | Cash return from dividends |
| **Payout Ratio** | `dividends_paid / net_income` | Sustainability of dividends |
| **Cash Conversion Cycle** | `days_inventory + days_receivable - days_payable` | How long cash is tied up in operations |

## Leverage metrics

| Metric | Formula | Interpretation |
|--------|---------|----------------|
| **Debt-to-Equity** | `total_debt / shareholders_equity` | Leverage relative to equity |
| **Current Ratio** | `current_assets / current_liabilities` | Short-term liquidity |
| **Net Debt** | `total_debt - cash_and_equivalents` | Debt burden after cash offset |

## Technical indicators

| Indicator | Formula | Interpretation |
|-----------|---------|----------------|
| **SMA(n)** | `sum(close, n) / n` | Simple moving average; trend direction |
| **EMA(n)** | `EMA_t = close × k + EMA_{t-1} × (1-k)`, `k = 2/(n+1)` | Exponential moving average; faster response than SMA |
| **RSI(14)** | `100 - 100 / (1 + RS)`, `RS = avg_gain / avg_loss` over 14 periods | Overbought (>70), oversold (<30) |
| **MACD** | `EMA(12) - EMA(26)` | Trend momentum; signal line = EMA(9) of MACD |
| **MACD Signal** | `EMA(9) of MACD` | Crossover with MACD signals trend change |
| **Bollinger Bands** | `SMA(20) ± 2 × stddev(20)` | Volatility envelope; price near band extremes may indicate reversal |

## Options greeks

| Greek | Measures | Rough formula |
|-------|----------|---------------|
| **Delta** | Price sensitivity to $1 move in underlying | `ΔC / ΔS` |
| **Gamma** | Rate of change of delta | `Δ²C / ΔS²` |
| **Theta** | Time decay per day | `-ΔC / Δt` |
| **Vega** | Sensitivity to 1-point change in IV | `ΔC / Δσ` |
| **Rho** | Sensitivity to 1% change in interest rate | `ΔC / Δr` |

## Earnings metrics

| Metric | Formula | Interpretation |
|--------|---------|----------------|
| **EPS** | `net_income / shares_outstanding` | Per-share profitability |
| **EPS (diluted)** | `net_income / diluted_shares` | Accounts for options, warrants, convertibles |
| **Earnings Surprise %** | `(actual_EPS - consensus_EPS) / abs(consensus_EPS)` | Beat or miss expectations |