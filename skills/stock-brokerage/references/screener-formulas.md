# Screener Formulas

## Standard screening pipeline

```
fetch universe → calculate fields → filter by thresholds → rank by score → present results
```

1. **Universe:** define the starting set (all stocks, a sector, an index, a market).
2. **Calculate:** compute the needed metrics for each stock.
3. **Filter:** apply threshold rules (e.g. P/E < 15, market cap > $1B).
4. **Rank:** sort survivors by a score (e.g. lowest P/E, highest ROE).
5. **Present:** show top N with key context.

---

## Common screeners

### Value

| Screener | Formula | Inputs | Interpretation | Example threshold |
|----------|---------|--------|----------------|-------------------|
| P/E | `market_cap / net_income` | Price, EPS (TTM) | Price per dollar of earnings | < 15 |
| P/B | `market_cap / book_value` | Price, book value per share | Price per dollar of assets | < 1.5 |
| P/S | `market_cap / revenue` | Price, revenue per share | Price per dollar of sales | < 2 |
| PEG | `P/E / EPS_growth_rate` | P/E, EPS growth % | P/E adjusted for growth | < 1 |
| EV/EBIT | `(market_cap + debt - cash) / EBIT` | EV, operating earnings | Firm value per operating dollar | < 10 |
| EV/EBITDA | `(market_cap + debt - cash) / EBITDA` | EV, EBITDA | Same, but adds back D&A | < 12 |
| Dividend Yield | `annual_dividend / share_price` | Dividend, price | Cash yield from dividends | > 3% |

### Quality

| Screener | Formula | Inputs | Interpretation | Example threshold |
|----------|---------|--------|----------------|-------------------|
| ROE | `net_income / shareholders_equity` | Net income, equity | Return on shareholder capital | > 15% |
| ROA | `net_income / total_assets` | Net income, total assets | Efficiency of asset use | > 8% |
| ROIC | `NOPAT / invested_capital` | Operating income, tax rate, invested capital | Return on all capital | > 12% |
| Gross Margin | `(revenue - COGS) / revenue` | Revenue, COGS | Pricing power | > 40% |
| Operating Margin | `operating_income / revenue` | Operating income, revenue | Profitability after ops costs | > 15% |
| Net Margin | `net_income / revenue` | Net income, revenue | Bottom-line profitability | > 10% |

### Growth

| Screener | Formula | Inputs | Interpretation | Example threshold |
|----------|---------|--------|----------------|-------------------|
| EPS Growth | `(EPS_t - EPS_{t-1}) / abs(EPS_{t-1})` | Current, prior EPS | Earnings growth rate | > 15% |
| Revenue Growth | `(rev_t - rev_{t-1}) / rev_{t-1}` | Current, prior revenue | Top-line growth | > 10% |
| Debt-to-Equity | `total_debt / shareholders_equity` | Debt, equity | Leverage relative to equity | < 0.5 |
| Current Ratio | `current_assets / current_liabilities` | Current assets, liabilities | Short-term liquidity | > 1.5 |

### Momentum

| Screener | Formula | Inputs | Interpretation | Example threshold |
|----------|---------|--------|----------------|-------------------|
| RSI(14) | `100 - 100 / (1 + RS)`, RS = avg_gain/avg_loss (14 periods) | Close prices (14 periods) | Overbought (>70), oversold (<30) | < 30 (oversold entry) |
| MACD Crossover | `MACD = EMA(12) - EMA(26)`; signal = `EMA(9) of MACD` | Close prices | Bullish when MACD crosses above signal | Crossover + price > SMA(200) |
| Moving-average crossover | Short SMA crosses long SMA | Close prices | Golden cross (50/200) = bullish | 50-day > 200-day |
| Volume spike | `volume_today / avg_volume_20d` | Volume (1 day, 20-day avg) | Unusual activity | > 2× |
| 52-week range | `(price - 52w_low) / (52w_high - 52w_low)` | Price, 52-week high/low | Position in annual range | > 0.8 (near highs) |

### Income

| Screener | Formula | Inputs | Interpretation | Example threshold |
|----------|---------|--------|----------------|-------------------|
| Dividend Yield | `annual_dividend / share_price` | Dividend, price | Cash yield | > 3% |
| Payout Ratio | `dividends_paid / net_income` | Dividends, net income | Sustainability | < 60% |

---

## Niche / quant screeners

### Piotroski F-Score (9-point checklist)

**Inputs:** net income, operating cash flow, ROA, long-term debt, current ratio, shares outstanding, gross margin, asset turnover.

| # | Criterion | Score 1 if... |
|---|-----------|----------------|
| 1 | Profitability: Net income > 0 | Positive earnings |
| 2 | Profitability: Operating cash flow > 0 | Positive cash generation |
| 3 | Profitability: ROA improving | ROA_t > ROA_{t-1} |
| 4 | Profitability: Cash flow > net income | Accruals are negative (quality of earnings) |
| 5 | Leverage: Long-term debt decreasing | LT_debt_t < LT_debt_{t-1} |
| 6 | Liquidity: Current ratio increasing | CR_t > CR_{t-1} |
| 7 | Liquidity: No dilution | Shares_t ≤ Shares_{t-1} |
| 8 | Efficiency: Gross margin improving | GM_t > GM_{t-1} |
| 9 | Efficiency: Asset turnover improving | AT_t > AT_{t-1} |

**Score:** 0–9. F-Score ≥ 7 = strong; ≤ 3 = weak.

### Magic Formula (Joel Greenblatt)

**Inputs:** EBIT, enterprise value, invested capital.

| Step | Action |
|------|--------|
| 1 | Compute **EBIT/EV** for each stock (earnings yield). |
| 2 | Compute **ROIC** for each stock. |
| 3 | Rank all stocks by EBIT/EV (highest = rank 1). |
| 4 | Rank all stocks by ROIC (highest = rank 1). |
| 5 | Add the two ranks. Lowest combined rank = best. |
| 6 | Exclude financials and utilities (optional). |

**Interpretation:** stocks that are cheap (high EBIT/EV) and high-quality (high ROIC) rank highest.

### Acquirer's Multiple (Tobias Carlisle)

**Inputs:** EBIT, enterprise value.

**Formula:** `Enterprise Value / EBIT`

**Interpretation:** lower = cheaper. Rank by lowest Acquirer's Multiple. Similar to Magic Formula but uses only the value factor.

### Altman Z-Score (bankruptcy prediction)

**Inputs:** working capital, retained earnings, EBIT, market cap, total liabilities, sales, total assets.

**Formula:**

```
Z = 1.2×(WC/TA) + 1.4×(RE/TA) + 3.3×(EBIT/TA) + 0.6×(MCap/TL) + 1.0×(Sales/TA)
```

| Zone | Interpretation |
|------|----------------|
| Z > 2.99 | Safe — low bankruptcy risk |
| 1.81 < Z < 2.99 | Grey zone — caution |
| Z < 1.81 | Distress — high bankruptcy risk |

### Beneish M-Score (earnings manipulation)

**Inputs:** current and prior year values for: DSRI, GMI, AQI, SGI, DEPI, SGAI, TATA, LVGI.

**Formula:**

```
M = -4.84 + 0.92×DSRI + 0.528×GMI + 0.404×AQI + 0.892×SGI
    + 0.115×DEPI - 0.172×SGAI + 4.679×TATA - 0.327×LVGI
```

Where each index compares current vs prior year ratios (e.g. `DSRI = (DSO_current / DSO_prior)`).

**Interpretation:** M-Score > -1.78 suggests likely earnings manipulation.

### Graham Number

**Inputs:** EPS, book value per share, maximum P/E = 15, maximum P/B = 1.5.

**Formula:**

```
Graham Number = √(22.5 × EPS × BVPS)
```

**Interpretation:** fair value ceiling per Benjamin Graham. If price < Graham Number, the stock is undervalued by this metric.

### Net-Net Working Capital (NNWC)

**Inputs:** cash, receivables, inventory, total liabilities, shares outstanding.

**Formula:**

```
NNWC = cash + 0.75×receivables + 0.5×inventory - total_liabilities
NNWC_per_share = NNWC / shares_outstanding
```

**Interpretation:** if price < NNWC_per_share, the stock trades below its liquidation value (deep value).

### ROIC – WACC Spread

**Inputs:** ROIC, WACC.

**Formula:** `ROIC - WACC`

**Interpretation:** positive spread = value creation. Wider spread = stronger competitive advantage. Compare across years to assess durability.

### Free Cash Flow Yield

**Inputs:** FCF, market cap.

**Formula:** `FCF / market_cap`

**Interpretation:** cash return on equity value. Higher = cheaper on a cash basis. Compare to bond yields and sector averages.

### Calmar Ratio

**Inputs:** annualized return, maximum drawdown.

**Formula:** `annualized_return / abs(max_drawdown)`

**Interpretation:** risk-adjusted return. Higher = better return per unit of drawdown risk.

### Dual Momentum (12-month return vs risk-free and benchmark)

**Inputs:** 12-month return of asset, 12-month return of benchmark (or risk-free rate).

**Rules:**

1. If asset 12-month return > risk-free rate → proceed to step 2; else go to cash.
2. If asset 12-month return > benchmark 12-month return → hold asset; else hold cash/benchmark.

**Interpretation:** combines absolute momentum (outperform risk-free) with relative momentum (outperform benchmark).

### Quality minus Junk Proxy

**Inputs:** ROIC, gross margin, debt/equity, earnings variability (std dev of EPS), investment rate (capex/revenue).

**Composite score (equal-weighted rank):**

| Factor | Direction | Ranking |
|--------|-----------|---------|
| ROIC | Higher = better | Rank ascending |
| Gross Margin | Higher = better | Rank ascending |
| Debt/Equity | Lower = better | Rank descending |
| Earnings variability | Lower = better | Rank descending |
| Investment rate (capex/revenue) | Lower = better (less wasteful investment) | Rank descending |

**Interpretation:** average the five ranks; lowest composite = highest quality.

### Low-Volatility Anomaly

**Inputs:** daily returns over 252 trading days.

**Formula:** `std_dev(daily_returns) × √252` (annualized volatility)

**Interpretation:** stocks with lower volatility tend to outperform on a risk-adjusted basis over long horizons. Filter for lowest vol; pair with momentum to avoid value traps.

### Short-Interest Ratio

**Inputs:** shares sold short, average daily volume.

**Formula:** `shares_sold_short / avg_daily_volume`

**Interpretation:** days-to-cover. High ratio (e.g. > 5) = heavy short interest; potential short squeeze if positive catalyst appears, or signal of fundamental concern.

### Insider Buy Ratio

**Inputs:** insider buy transactions (count or value), insider sell transactions (count or value).

**Formula:** `insider_buys / (insider_buys + insider_sells)` (by count or dollar value over trailing period)

**Interpretation:** ratio > 0.5 = more buying than selling; insiders believe stock is undervalued. Cluster of buys from multiple insiders is stronger signal.

### Earnings Surprise %

**Inputs:** actual EPS, consensus EPS estimate.

**Formula:** `(actual_EPS - consensus_EPS) / abs(consensus_EPS)`

**Interpretation:** positive = beat; negative = miss. Large positive surprises can drive post-earnings momentum. Consistent beats suggest management under-promises.

### Operating Leverage

**Inputs:** revenue growth, operating income growth.

**Formula:** `operating_income_growth / revenue_growth`

**Interpretation:** ratio > 1 = high operating leverage; small revenue gains produce large profit gains, but losses amplify in downturns.

### Financial Leverage

**Inputs:** total assets, shareholders' equity.

**Formula:** `total_assets / shareholders'_equity`

**Interpretation:** multiplier effect. Ratio of 3 means $1 of equity supports $3 of assets. Higher = more risk, more return potential.

### Cash Conversion Cycle

**Inputs:** cost of goods sold, inventory, revenue, accounts receivable, accounts payable.

**Formula:**

```
days_inventory = 365 / (COGS / avg_inventory)
days_receivable = 365 / (revenue / avg_receivable)
days_payable = 365 / (COGS / avg_payable)
CCC = days_inventory + days_receivable - days_payable
```

**Interpretation:** fewer days = faster cash recovery. Negative CCC (e.g. Amazon) = company collects from customers before paying suppliers.

---

## Data availability and fallbacks

- **Net debt** often unavailable; use `total_debt - cash_and_equivalents` or `total_liabilities - cash` as proxy.
- **NOPAT** may not be reported; compute `operating_income × (1 - effective_tax_rate)`.
- **Invested capital** varies by provider; use `total_assets - current_liabilities - excess_cash` or `shareholders_equity + net_debt`.
- **Shares outstanding** — check for diluted vs basic; diluted includes options/warrants.
- **Historical consensus EPS** — not available from most free providers; use actual EPS and note the limitation.
- **Short interest** — often delayed; data may be 2 weeks old.
- **Insider transactions** — SEC Form 4 filings; free sources (OpenInsider) have delays.

When a required input is unavailable, state the missing field and use the closest calculable proxy; do not halt.