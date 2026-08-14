---
name: stock-brokerage
description: Analyze equities and market data using provider-agnostic patterns that work with Yahoo Finance, broker APIs, Alpha Vantage, Polygon.io, and other data providers. Use when asked about stock prices, tickers, fundamentals, technical indicators, earnings, dividends, options, screening, or portfolio analysis.
---

# Stock Brokerage

Analyze equities and market data without coupling to a specific provider. Every concept here maps to any data API via a small adapter interface.

## When to use

- Stock prices, tickers, fundamentals, technical indicators, earnings, dividends.
- Options chains and options analysis.
- Stock screening and filtering.
- Portfolio analysis and valuation.
- Comparing stocks across exchanges.

**Not for** executing trades. This skill analyzes data; it does not interact with brokerage accounts or place orders. Output is not investment advice.

## Quick process

1. Identify ticker(s) — resolve names to ticker symbols via `get_stock_list` if needed.
2. Identify the data need — price, fundamentals, options, screening?
3. Choose a provider — match the adapter to the available API key and data needs.
4. Fetch — call the adapter method.
5. Validate — check for empty/missing data, wrong date ranges, rate limit errors.
6. Present — summarize, highlight anomalies, include context and disclaimer.

## Provider-agnostic adapter

A small interface with methods that map to universal data concepts, not provider-specific endpoints:

| Method | What it fetches |
|--------|-----------------|
| `quote(ticker)` | Latest price, bid/ask, volume, market cap |
| `historical(tickers, period, interval)` | OHLCV bars over a date range |
| `fundamentals(ticker, statement, period)` | Financial statements (income, balance sheet, cash flow) |
| `corporate_actions(ticker)` | Dividends, splits, buybacks |
| `options_chain(ticker, expiration)` | Calls/puts, greeks, open interest |
| `screen(criteria)` | Filter a universe by a set of rules |

See `references/provider-adapters.md` for typed interfaces and skeletons for yfinance, Alpha Vantage, and Polygon.io.

## Data categories

| User request | Universal concept |
|--------------|-------------------|
| "What's the price of X?" | Quote (latest price, volume, bid/ask) |
| "Show me the last 6 months" | Historical OHLCV |
| "What's the P/E ratio?" | Fundamental — valuation metric |
| "When is the next dividend?" | Corporate action — dividend schedule |
| "What are the options for X?" | Options chain |
| "Find me cheap stocks" | Screener — filter + rank |
| "Is X undervalued?" | Fundamental + valuation + peer comparison |
| "How did X perform today?" | Intraday OHLCV |

## Screening concept

A screener is a **filter plus a rank**:

- **Filter:** pass/fail criteria (e.g. P/E < 15, market cap > $1B).
- **Rank:** sort surviving stocks by a score (e.g. lowest P/E first, highest ROE first).
- **Formula:** the calculation that produces the filter or rank value from raw data.

Screening requires: a universe (all stocks to consider), formulas for each criterion, threshold values, and a ranking strategy.

See `references/screener-formulas.md` for common and niche/quant screeners with exact formulas.

## Named providers

| Provider | Access | Notes |
|----------|--------|-------|
| yfinance | Free (unofficial Yahoo API) | Broad coverage, no API key; may be rate-limited |
| Alpha Vantage | Free tier + paid | Fundamentals + technical; API key required |
| Polygon.io | Free tier + paid | Real-time/delayed; good for intraday |
| Twelve Data | Free tier + paid | Technical and historical; clean API |
| Interactive Brokers | Account required | Real-time data + execution |
| Alpaca | Account required | Commission-free; good for US equities |
| Schwab/OpenAPI | Account required | TD Ameritrade successor |
| FRED | Free | Economic indicators (macro context) |
| Quandl / NASDAQ Data Link | Free + paid | Alternative datasets |

See `references/provider-adapters.md` for adapter skeletons.

## Universal cautions

- **Ticker format varies** by exchange (e.g. `BBCA.JK` for IDX, `0700.HK` for HKEX).
- **Timezone and market hours** — timestamps may be in exchange-local time or UTC.
- **Adjusted vs unadjusted prices** — splits and dividends shift historical values.
- **Rate limits** — respect provider limits; implement retries with backoff.
- **Missing/empty data** — check for `None`/NaN before computing ratios.
- **Survivorship bias** — delisted stocks are absent from current lists; filter carefully.

## Presenting results

- Summarize key numbers first, not a wall of raw data.
- Use markdown tables for multi-stock comparisons.
- Highlight anomalies (outliers, missing data, extreme values).
- Provide context (sector averages, peer comparison, historical trends).
- Include disclaimer: not investment advice.

## References

- `references/provider-adapters.md` — typed adapter interface and provider skeletons.
- `references/data-concepts.md` — definitions and formulas for common data fields.
- `references/screener-formulas.md` — common and niche/quant screeners with exact formulas and steps.