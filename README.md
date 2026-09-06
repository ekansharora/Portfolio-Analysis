# Portfolio Analytics Pipeline & Dashboard

A small end-to-end data pipeline that pulls daily market data, computes
standard risk/return metrics, and serves them through an interactive
Streamlit dashboard.

## Why this exists

Built to demonstrate: ETL design, financial metrics implementation from
first principles (no black-box libraries for the math), test coverage
on the numerical core, and a clean separation between data layer /
compute layer / presentation layer.

## Architecture

```
tickers --> data_fetch.py --> storage.py (SQLite) --> metrics.py --> storage.py
                                     |
                                     v
                            dashboard/app.py (Streamlit)
```

- **`pipeline/data_fetch.py`** — pulls OHLCV data via `yfinance`.
- **`pipeline/storage.py`** — SQLite persistence (`prices`, `metrics` tables). The dashboard never touches the network, only this DB.
- **`pipeline/metrics.py`** — pure functions: annualized return/volatility, Sharpe, Sortino, max drawdown, beta, correlation matrix. No I/O — fully unit tested.
- **`pipeline/run.py`** — CLI that wires the above together.
- **`dashboard/app.py`** — Streamlit UI: price history, cumulative returns, latest metrics table, correlation heatmap.

## Setup

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Usage

1. Run the pipeline to fetch data and compute metrics:

```bash
python -m pipeline.run --tickers AAPL,MSFT,GOOG --start 2020-01-01 --benchmark SPY
```

2. Launch the dashboard:

```bash
streamlit run dashboard/app.py
```

## Tests

```bash
pytest -v
```

Tests run against seeded synthetic price series (`tests/conftest.py`) so
they're deterministic and don't depend on network access — useful both
for CI and for reviewers running this without an API key.

## Metrics implemented

| Metric | Notes |
|---|---|
| Annualized return | Geometric, compounded over trading days |
| Annualized volatility | Std dev of daily returns, scaled by √252 |
| Sharpe ratio | Configurable risk-free rate |
| Sortino ratio | Downside deviation only |
| Max drawdown | Peak-to-trough on price series |
| Beta | Covariance with a chosen benchmark ticker |
| Correlation matrix | Across all selected tickers |

## Possible extensions

- Swap SQLite for Postgres + Airflow/Prefect scheduling
- Add position sizing / portfolio-level (weighted) metrics, not just per-asset
- Add a backtesting module for simple strategies (moving average crossover, etc.)
- Dockerize + deploy dashboard (Streamlit Community Cloud / Fly.io)

## License

MIT — see `LICENSE`.
