# Provider Adapters

## Typed adapter interface (Python)

```python
from __future__ import annotations
from typing import Protocol, Any

class StockAdapter(Protocol):
    """Provider-agnostic interface for stock data."""

    def quote(self, ticker: str) -> dict[str, Any]:
        """Latest price, volume, bid/ask, market cap."""
        ...

    def historical(
        self,
        tickers: list[str],
        period: str,
        interval: str,
    ) -> dict[str, Any]:
        """OHLCV bars over a date range."""
        ...

    def fundamentals(
        self,
        ticker: str,
        statement: str,
        period: str,
    ) -> dict[str, Any]:
        """Financial statements: income, balance_sheet, cash_flow."""
        ...

    def corporate_actions(self, ticker: str) -> dict[str, Any]:
        """Dividends, splits, buybacks."""
        ...

    def options_chain(
        self,
        ticker: str,
        expiration: str,
    ) -> dict[str, Any]:
        """Calls/puts with greeks and open interest."""
        ...

    def screen(self, criteria: dict[str, Any]) -> list[dict[str, Any]]:
        """Filter universe by criteria; return ranked results."""
        ...
```

## yfinance adapter

```python
import yfinance as yf

class YFinanceAdapter:
    def quote(self, ticker: str) -> dict:
        t = yf.Ticker(ticker)
        info = t.info
        return {
            "ticker": ticker,
            "price": info.get("currentPrice") or info.get("regularMarketPrice"),
            "volume": info.get("volume"),
            "market_cap": info.get("marketCap"),
            "bid": info.get("bid"),
            "ask": info.get("ask"),
        }

    def historical(self, tickers: list[str], period: str = "6mo", interval: str = "1d") -> dict:
        data = yf.download(tickers, period=period, interval=interval, group_by="ticker")
        return {t: data[t].to_dict("records") for t in tickers}

    def fundamentals(self, ticker: str, statement: str = "income", period: str = "annual") -> dict:
        t = yf.Ticker(ticker)
        stmt_map = {"income": t.financials, "balance_sheet": t.balance_sheet, "cash_flow": t.cashflow}
        df = stmt_map.get(statement)
        return df.to_dict() if df is not None else {}

    def corporate_actions(self, ticker: str) -> dict:
        t = yf.Ticker(ticker)
        return {
            "dividends": t.dividends.to_dict() if t.dividends is not None else {},
            "splits": t.splits.to_dict() if t.splits is not None else {},
        }

    def options_chain(self, ticker: str, expiration: str) -> dict:
        t = yf.Ticker(ticker)
        chain = t.option_chain(expiration)
        return {
            "calls": chain.calls.to_dict("records"),
            "puts": chain.puts.to_dict("records"),
        }

    def screen(self, criteria: dict) -> list[dict]:
        # yfinance has no server-side screener; fetch universe + filter locally
        raise NotImplementedError("yfinance does not support server-side screening")
```

## Alpha Vantage adapter

```python
import os, requests

class AlphaVantageAdapter:
    BASE = "https://www.alphavantage.co/query"
    KEY = os.environ.get("ALPHAVANTAGE_API_KEY", "")

    def quote(self, ticker: str) -> dict:
        r = requests.get(self.BASE, params={
            "function": "GLOBAL_QUOTE", "symbol": ticker, "apikey": self.KEY,
        }).json().get("Global Quote", {})
        return {"ticker": ticker, "price": r.get("05. price"), "volume": r.get("06. volume")}

    def historical(self, tickers: list[str], period: str = "1y", interval: str = "daily") -> dict:
        # Alpha Vantage returns one ticker at a time
        result = {}
        for t in tickers:
            r = requests.get(self.BASE, params={
                "function": "TIME_SERIES_DAILY", "symbol": t,
                "outputsize": "compact", "apikey": self.KEY,
            }).json().get("Time Series (Daily)", {})
            result[t] = [{"date": k, **v} for k, v in r.items()]
        return result

    def fundamentals(self, ticker: str, statement: str = "income", period: str = "annual") -> dict:
        func_map = {"income": "INCOME_STATEMENT", "balance_sheet": "BALANCE_SHEET", "cash_flow": "CASH_FLOW"}
        r = requests.get(self.BASE, params={
            "function": func_map.get(statement, "INCOME_STATEMENT"),
            "symbol": ticker, "apikey": self.KEY,
        }).json()
        reports = r.get("annualReports", r.get("quarterlyReports", []))
        return reports[0] if reports else {}

    def corporate_actions(self, ticker: str) -> dict:
        r = requests.get(self.BASE, params={
            "function": "DIVIDENDS", "symbol": ticker, "apikey": self.KEY,
        }).json()
        return {"dividends": r.get("data", [])}

    def options_chain(self, ticker: str, expiration: str) -> dict:
        raise NotImplementedError("Alpha Vantage does not provide options chains")

    def screen(self, criteria: dict) -> list[dict]:
        raise NotImplementedError("Alpha Vantage does not support custom screening")
```

## Polygon.io adapter

```python
import os, requests

class PolygonAdapter:
    BASE = "https://api.polygon.io"
    KEY = os.environ.get("POLYGON_API_KEY", "")

    def _get(self, path: str, params: dict = {}) -> dict:
        params["apiKey"] = self.KEY
        return requests.get(f"{self.BASE}{path}", params=params).json()

    def quote(self, ticker: str) -> dict:
        r = self._get(f"/v2/snapshot/locale/us/markets/stocks/tickers/{ticker}")
        t = r.get("ticker", {})
        return {"ticker": ticker, "price": t.get("min", {}).get("c"), "volume": t.get("todaysVolume")}

    def historical(self, tickers: list[str], period: str = "2024-01-01", interval: str = "day") -> dict:
        end = "2024-12-31"
        result = {}
        for t in tickers:
            r = self._get(f"/v2/aggs/ticker/{t}/range/1/day/{period}/{end}")
            result[t] = r.get("results", [])
        return result

    def fundamentals(self, ticker: str, statement: str = "income", period: str = "annual") -> dict:
        # Polygon.io financials are premium; basic endpoint:
        r = self._get(f"/vX/reference/financials", {"ticker": ticker, "type": statement, "period": period})
        return r.get("results", [{}])[0] if r.get("results") else {}

    def corporate_actions(self, ticker: str) -> dict:
        r = self._get(f"/v3/reference/dividends", {"ticker": ticker})
        return {"dividends": r.get("results", [])}

    def options_chain(self, ticker: str, expiration: str) -> dict:
        r = self._get(f"/v3/snapshot/options/{ticker}", {"expiration_date": expiration})
        return {"options": r.get("results", [])}

    def screen(self, criteria: dict) -> list[dict]:
        raise NotImplementedError("Polygon.io does not support server-side screening")
```

## Switching providers via constructor

```python
import os

def get_adapter(provider: str = None) -> StockAdapter:
    provider = provider or os.environ.get("STOCK_PROVIDER", "yfinance")
    adapters = {
        "yfinance": YFinanceAdapter,
        "alpha_vantage": AlphaVantageAdapter,
        "polygon": PolygonAdapter,
    }
    return adapters[provider]()

# Usage:
# STOCK_PROVIDER=polygon python analyze.py --ticker AAPL
```