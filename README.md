# AI E-commerce Operations Agent — Forecastly

An inventory intelligence and operations MVP designed to help e-commerce and retail teams identify stock risks, spot slow-moving or dead-stock candidates, estimate near-term demand, and prepare replenishment proposals for human review.

The project combines a **FastAPI REST API**, a **Streamlit operations dashboard**, SQLAlchemy-backed data storage, scheduled inventory monitoring, and an optional Google Gemini-powered investigation agent. It includes synthetic demo products and sales history so the application can be explored locally without a real store connection or paid AI API.

> **Project status:** Runnable local MVP / portfolio project. Demo mode uses synthetic data. Gemini is optional. Shopify integration is a read-only integration scaffold. The application does not place supplier orders.

## Key Features

- **Inventory Overview:** View seeded products, stock levels, categories, prices, and sales metrics.
- **Automated Risk Monitoring:** Periodically checks inventory and sales signals for stock-out risk, low stock, slow-moving products, and dead-stock candidates.
- **Manual Monitoring:** Trigger a risk check from the dashboard or REST API.
- **Demand Baseline:** Estimates next-seven-day demand using a transparent 30-day moving-average baseline and compares recent sales with the previous 30-day period.
- **Evidence-Grounded Investigation Agent:** Summarizes product risks using recorded inventory, sales metrics, and active alerts. It uses Google Gemini when configured and falls back to deterministic rules if no key is set or the AI request fails.
- **Replenishment Proposals:** Calculates a suggested replenishment quantity and creates a pending proposal.
- **Human Review Workflow:** Approve or reject proposals in the dashboard. Approval only records a decision; it does not send a purchase order.
- **Interactive Dashboard:** Streamlit tabs for inventory, risk alerts, agent investigations, approvals, and demand baselines.
- **REST API and Swagger UI:** FastAPI endpoints with interactive API documentation.
- **Persistence Options:** SQLite by default for simple local setup, with PostgreSQL available through Docker Compose.
- **Tests and CI:** Pytest tests and a GitHub Actions workflow.
- **Optional Shopify Scaffold:** A read-only Admin REST API client scaffold for future integration with a real store.

## Tech Stack

| Area | Technologies |
|---|---|
| Programming Language | Python |
| Backend / API | FastAPI, REST APIs, Pydantic, Uvicorn |
| Data Access | SQLAlchemy |
| Database | SQLite by default; PostgreSQL with Docker Compose |
| Dashboard | Streamlit |
| Data Analysis / Visualization | Pandas, Plotly |
| AI / Agent | Google Gemini API (optional), evidence-grounded rules fallback |
| Scheduled Monitoring | APScheduler |
| Integrations | Shopify read-only client scaffold |
| Testing | Pytest, HTTPX |
| DevOps | Docker, Docker Compose, GitHub Actions |
| Version Control | Git, GitHub |

## How It Works

1. The application loads a small demo catalog and synthetic sales history.
2. The monitoring logic calculates recent sales, average daily units, stock coverage, and days since the last recorded sale.
3. Rule-based checks create or update risk alerts and mark no-longer-applicable open alerts as resolved.
4. A scheduled job repeats monitoring at a configurable interval; users can also trigger it manually.
5. The demand endpoint estimates next-seven-day units from the last 30 days and reports the change against the preceding 30-day average.
6. The investigation agent combines product data, sales metrics, and open alerts into evidence and recommendations. If configured, Gemini can summarize that evidence; otherwise, rules-based output is used.
7. A user can create a draft replenishment proposal and approve or reject it. No supplier order is sent automatically.

## Architecture

```text
                 Streamlit Dashboard
                         |
                         | HTTP REST requests
                         v
                    FastAPI API
                         |
           +-------------+-------------+
           |             |             |
           v             v             v
     Inventory &     Risk Monitor   Investigation
     Sales Metrics   + Scheduler       Agent
           |             |             |
           +-------------+-------------+
                         |
                         v
                   SQLAlchemy ORM
                         |
              +----------+----------+
              |                     |
              v                     v
          SQLite local        PostgreSQL Docker
              |
              v
       Products, Sales, Risks,
       Replenishment Actions
```

## Repository Structure

```text
ai-ecommerce-operations-agent/
├── app/
│   ├── integrations/
│   │   └── shopify.py       # Read-only Shopify integration scaffold
│   ├── agent.py             # Investigation and replenishment proposals
│   ├── config.py            # Environment-based configuration
│   ├── database.py          # SQLAlchemy database setup
│   ├── logic.py             # Sales metrics, forecasting, risk rules
│   ├── main.py              # FastAPI routes and monitoring scheduler
│   ├── models.py            # Database models
│   ├── schemas.py            # API schemas
│   └── seed.py               # Synthetic demo catalog and sales data
├── dashboard/
│   └── app.py                # Streamlit dashboard
├── tests/
│   ├── test_api.py
│   └── test_logic.py
├── .github/workflows/
│   └── tests.yml             # CI workflow
├── docker-compose.yml
├── Dockerfile
├── requirements.txt
├── start_windows.bat         # Windows local startup helper
└── README.md
```

## Getting Started — Windows

### Prerequisites

- Python 3.11 or 3.12 recommended
- Git (optional, if cloning)
- Docker Desktop only if you want to use the PostgreSQL Docker setup
- Google Gemini API key only if you want the optional LLM-generated investigation summary

### Option 1: Quick Start (Recommended)

1. Download or clone the repository and extract it to a normal folder.
2. Open the project folder.
3. Double-click `start_windows.bat`, or run it from Command Prompt:

```cmd
start_windows.bat
```

The helper script is intended to create a virtual environment, install dependencies, seed demo data through application startup, and launch the API and dashboard in separate windows.

Open these URLs:

| Service | URL |
|---|---|
| Streamlit Dashboard | http://localhost:8501 |
| FastAPI Swagger UI | http://localhost:8000/docs |
| Health Check | http://localhost:8000/health |

If the dashboard opens before the API is ready, wait a few seconds and refresh.

### Option 2: Manual Local Setup

From the project root in Command Prompt:

```cmd
py -3.12 -m venv .venv
.venv\Scripts\activate
python -m pip install --upgrade pip
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```

Open a **second terminal** in the project root:

```cmd
.venv\Scripts\activate
streamlit run dashboard\app.py
```

The application uses SQLite by default, so no separate database setup is needed for a local demo. If the `py -3.12` launcher is unavailable, use an installed Python 3.11 or 3.12 interpreter to create the virtual environment.

### Option 3: Docker with PostgreSQL

Make sure Docker Desktop is running, then execute from the project root:

```cmd
docker compose up --build
```

The Compose setup starts PostgreSQL, the FastAPI API, and the Streamlit dashboard. PostgreSQL is exposed on host port `5433`; the API is on port `8000` and the dashboard on port `8501`.

Stop the services with:

```cmd
docker compose down
```

To preserve the database volume, do not add `-v`.

## Configuration

The application works in rules-only demo mode without an AI key. For optional Gemini-powered investigation summaries, configure these environment variables in a local `.env` file:

```env
GEMINI_API_KEY=your_gemini_api_key
GEMINI_MODEL=gemini-2.5-flash
```

The application also supports configuration such as `DATABASE_URL`, `MONITOR_INTERVAL_SECONDS`, `SAFETY_BUFFER_DAYS`, `LOW_STOCK_DAYS`, and `DEAD_STOCK_DAYS`. Check `app/config.py` and `docker-compose.yml` for defaults.

**Security:** Never commit `.env` files, API keys, database credentials, or real Shopify access tokens to GitHub. The supplied demo data is synthetic.

## API Endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/health` | API health response |
| `GET` | `/products` | List products |
| `GET` | `/products/{sku}` | Product details and sales metrics |
| `GET` | `/risks?status=open` | List open risk alerts; use `status=all` for all statuses |
| `POST` | `/monitor/run` | Run risk monitoring immediately |
| `POST` | `/forecast/{sku}` | Calculate the demand baseline |
| `POST` | `/agent/investigate/{sku}` | Investigate a product and return evidence and recommendations |
| `POST` | `/actions/replenishment/{sku}` | Create a draft replenishment proposal |
| `GET` | `/actions` | List replenishment proposals and decisions |
| `POST` | `/actions/{action_id}/decision` | Approve or reject a pending proposal |
| `POST` | `/demo/reset` | Restore the synthetic demo catalog and sales history |

Interactive request and response schemas are available at `http://localhost:8000/docs`.

## Testing

Activate the project's virtual environment, then run:

```cmd
pytest
```

The repository includes API and business-logic tests plus a GitHub Actions workflow. For production readiness, add broader integration tests, data-quality checks, forecast evaluation, and end-to-end dashboard tests.

## Limitations and Future Improvements

- This is a portfolio MVP, not a fully audited production system.
- Demo inventory and sales are synthetic, not live retailer data.
- The demand forecast is a simple moving-average baseline, not a trained predictive model; it does not account for promotions, holidays, seasonality, or demand censored by stockouts.
- Risk thresholds are configurable rule-based heuristics and should be validated for a real business.
- The Shopify module is a read-only integration scaffold; live synchronization requires store credentials, appropriate scopes, and validation against the store's current API version.
- The current demo does not provide production-grade user authentication, tenant isolation, database migrations, or a complete audit/security layer.
- Approvals record decisions only. No purchase order or supplier communication is sent.

Possible next steps include live inventory synchronization, role-based access, tenant isolation, background task queues, richer demand models, forecast evaluation, observability, alert delivery, and production deployment.

## Author

**Shruti Mawalkar**

Built as a portfolio project exploring AI-assisted retail operations, inventory analytics, REST API development, automated monitoring, demand baselines, and human-in-the-loop agent workflows.

## Project Links

- **GitHub Repository:** Add your repository URL here after uploading the project.
