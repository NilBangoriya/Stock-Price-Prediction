<div align="center">

# 📈 Stock Price Prediction & Portfolio Optimisation

**Prophet-powered next-day price forecasting with Markowitz mean-variance portfolio optimisation — automated end-to-end with a live Streamlit dashboard.**

![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)
![Prophet](https://img.shields.io/badge/Prophet-Time%20Series-0467DF?logo=meta&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-Dashboard-FF4B4B?logo=streamlit&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-Database-3ECF8E?logo=supabase&logoColor=white)
![Poetry](https://img.shields.io/badge/Poetry-Dependencies-60A5FA?logo=poetry&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Data-150458?logo=pandas&logoColor=white)
![SciPy](https://img.shields.io/badge/SciPy-Optimisation-8CAAE6?logo=scipy&logoColor=white)
![CircleCI](https://img.shields.io/badge/CircleCI-CI-343434?logo=circleci&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-Automation-2088FF?logo=githubactions&logoColor=white)
![pytest](https://img.shields.io/badge/pytest-Tested-0A9EDC?logo=pytest&logoColor=white)

</div>

---

## Overview

This project forecasts next-day closing prices for a portfolio of large-cap stocks using **Meta's Prophet** model, then feeds those forecasts into a **Markowitz mean-variance optimiser** to compute an optimal asset allocation. Results are persisted to **Supabase** and visualised through a **Streamlit** dashboard. A GitHub Actions workflow runs the full pipeline daily, and a second workflow deploys the dashboard to a VPS on every push to `main`.

> ⚠️ **Disclaimer:** Built for educational and demonstration purposes. Predictions are not financial advice — markets are noisy and no forecasting model guarantees future returns.

## How It Works

```
 Extract              Preprocess            Forecast              Optimise                Persist          Visualise
┌──────────┐        ┌─────────────┐       ┌────────────┐       ┌────────────────┐      ┌───────────┐     ┌───────────┐
│ yfinance │  ──▶   │ Align dates │  ──▶  │ Prophet     │  ──▶  │ Mean-variance   │ ──▶  │ Supabase  │ ──▶ │ Streamlit │
│ (daily   │        │ across all  │       │ per-ticker  │       │ optimisation    │      │ Postgres  │     │ dashboard │
│ OHLCV)   │        │ tickers     │       │ next-day    │       │ (SciPy SLSQP)   │      │ table     │     │           │
└──────────┘        └─────────────┘       │ forecast    │       └────────────────┘      └───────────┘     └───────────┘
                                           └────────────┘
```

1. **Extract** — Pull daily OHLCV data for each ticker via `yfinance` and derive daily returns.
2. **Preprocess** — Align every ticker to a common set of trading dates.
3. **Forecast** — Fit a separate Prophet model per ticker (with yearly/weekly seasonality and real NYSE trading holidays pulled from `pandas_market_calendars`) to predict the next day's price.
4. **Optimise** — Use the predicted returns as expected returns in a mean-variance objective (`return − ½·λ·variance`), solved with `scipy.optimize.minimize` (SLSQP) subject to per-asset allocation bounds and weights summing to 1.
5. **Persist** — Write predictions, returns, and portfolio weights to a Supabase table.
6. **Visualise** — Serve an interactive Streamlit dashboard reading live from Supabase.

## Features

- 📊 Per-ticker next-day price forecasting with Prophet, holiday-aware for the US market calendar
- ⚖️ Markowitz mean-variance portfolio optimisation with configurable risk aversion and allocation bounds
- 🗄️ Results persisted to Supabase for historical tracking
- 📈 Streamlit dashboard for visualising predictions, allocations, and recent price history
- 🤖 Fully automated daily pipeline via GitHub Actions (cron + manual dispatch)
- 🚀 One-push deployment to a VPS with an automatic `systemd` service restart
- ✅ CI on every commit — linting (ruff + black), type-checking (mypy), and tests with coverage (CircleCI)

## Tech Stack

| Layer | Tools |
|---|---|
| Language | Python 3.12 |
| Forecasting | Prophet, pandas-market-calendars |
| Optimisation | SciPy, NumPy |
| Data | yfinance, pandas |
| Database | Supabase (Postgres) |
| Dashboard | Streamlit, Plotly, Altair |
| Dependency management | Poetry |
| Testing | pytest, pytest-cov, pytest-mock |
| Code quality | ruff, black, mypy, pre-commit |
| CI/CD | CircleCI, GitHub Actions |
| Deployment | Hostinger VPS via SSH + systemd |

## Project Structure

```
Stock-Price-Prediction/
├── src/
│   ├── main.py              # Pipeline entry point (extract → predict → optimise → save)
│   ├── extractor.py         # yfinance data extraction
│   ├── processor.py         # Date alignment & prediction row appending
│   ├── model.py              # Prophet model wrapper with holiday calendar
│   ├── optimiser.py          # Mean-variance portfolio optimisation
│   ├── database.py           # Supabase read/write
│   ├── settings.py           # Tickers, dates, risk params, Prophet config
│   └── streamlit_app.py      # Live dashboard
├── tests/                    # Unit tests for each module
├── scripts/deploy.sh         # VPS deployment script (git pull + restart service)
├── .github/workflows/        # Daily optimisation cron + VPS deploy on push
├── .circleci/config.yml      # Lint, type-check, test pipeline
├── pyproject.toml            # Poetry dependencies & tool config
└── Makefile                  # Common dev commands
```

## Getting Started

### Prerequisites

- Python 3.12
- [Poetry](https://python-poetry.org/)
- A [Supabase](https://supabase.com/) project (for persistence and the dashboard)

### Installation

```bash
git clone https://github.com/NilBangoriya/Stock-Price-Prediction.git
cd Stock-Price-Prediction
make install-dev   # poetry install with dev dependencies
```

### Environment Variables

Create a `.env` file in the project root:

```env
SUPABASE_URL=your-supabase-project-url
SUPABASE_KEY=your-supabase-api-key
```

### Usage

```bash
make run         # Run the forecasting + optimisation pipeline once, save to Supabase
make dashboard   # Launch the Streamlit dashboard locally
make test        # Run the test suite with coverage
make check       # Run format, lint, type-check, and test together
```

Portfolio tickers, date ranges, risk aversion, and allocation bounds are all configurable in `src/settings.py`.

## Automation

| Workflow | Trigger | What it does |
|---|---|---|
| `daily-optimisation.yml` | Daily cron (09:00 UTC) + manual dispatch | Runs the full pipeline and writes results to Supabase |
| `deploy.yml` | Push to `main` | SSHes into a Hostinger VPS, pulls latest code, reinstalls dependencies if `pyproject.toml` changed, and restarts the Streamlit `systemd` service |
| CircleCI | Every commit | Lints (ruff, black), type-checks (mypy), and runs the test suite with coverage |

## Testing

```bash
make test
```

Covers the extraction, preprocessing, Prophet modelling, optimisation, and database modules under `tests/`.

## License

No license specified yet — add one (e.g. MIT) if you'd like others to reuse this code.
