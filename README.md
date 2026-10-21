# StockPilot AI — Intelligent Inventory & Retail Operations Agent

StockPilot AI is an AI-powered inventory intelligence platform that helps identify stock risks, detect slow-moving and dead-stock candidates, estimate short-term demand, and generate replenishment recommendations.

Built with FastAPI, Streamlit, SQLAlchemy, and optional Google Gemini integration, the application combines automated inventory monitoring, sales analytics, demand estimation, and human-in-the-loop decision-making in one dashboard.

**Project Status:** Working local MVP with automated scheduled monitoring, manual risk checks, demand forecasting baselines, AI-assisted investigations, replenishment proposals, and Docker support.

**Note:** The current demo uses synthetic inventory and sales data. Monitoring works automatically on the application's available data; live synchronization with a real e-commerce store is not yet implemented.

## Features

- **Inventory Dashboard:** View products, stock levels, categories, prices, and sales metrics.
- **Automated Risk Monitoring:** Periodically checks inventory and sales data for stock-out risk, low stock, slow-moving products, and potential dead stock.
- **Manual Risk Checks:** Trigger inventory monitoring from the dashboard or REST API.
- **Demand Forecasting Baseline:** Estimates demand for the next seven days using recent sales history.
- **Sales Trend Comparison:** Compares recent sales with the previous 30-day period.
- **AI-Assisted Investigation:** Uses Google Gemini when configured to summarize product risks and recommendations based on available evidence.
- **Rules-Based Fallback:** Continues to provide deterministic investigation results when Gemini is not configured or an AI request fails.
- **Replenishment Recommendations:** Calculates suggested replenishment quantities and creates draft proposals.
- **Human-in-the-Loop Workflow:** Allows users to approve or reject replenishment proposals.
- **Interactive Dashboard:** Provides views for inventory, risk alerts, investigations, approvals, and demand baselines.
- **REST API:** FastAPI endpoints with interactive Swagger documentation.
- **Database Support:** SQLite for local development and PostgreSQL through Docker Compose.
- **Automated Tests:** Pytest tests and a GitHub Actions workflow.
- **Shopify Integration Scaffold:** Includes a read-only integration foundation for future store connectivity.

## Tech Stack

| Area | Technologies |
|---|---|
| Programming Language | Python |
| Backend | FastAPI, Uvicorn, Pydantic |
| API | REST APIs |
| Frontend / Dashboard | Streamlit |
| Data Access | SQLAlchemy |
| Database | SQLite, PostgreSQL |
| Data Analysis | Pandas |
| Visualization | Plotly |
| AI Integration | Google Gemini API (optional) |
| Inventory Monitoring | APScheduler, rule-based risk detection |
| Forecasting | 30-day moving-average demand baseline |
| Integration Scaffold | Shopify Admin REST API client |
| Testing | Pytest, HTTPX |
| Containerization | Docker, Docker Compose |
| CI | GitHub Actions |
| Version Control | Git, GitHub |

## How It Works

1. The application loads a sample product catalog and synthetic sales history.
2. The monitoring service calculates recent sales, average daily sales, estimated stock coverage, and time since the last recorded sale.
3. Rule-based checks identify potential inventory risks and create or update risk alerts.
4. Scheduled monitoring runs at a configurable interval, while users can also trigger checks manually.
5. The demand endpoint estimates the next seven days of demand using recent sales history.
6. The investigation agent combines product metrics and active alerts to generate recommendations. Google Gemini can provide an AI-generated summary when configured.
7. Users can create replenishment proposals and approve or reject them through the dashboard.

Approving a proposal records the decision only. The application does not automatically place supplier orders.

## Architecture

```text
              Streamlit Dashboard
                       |
                       | REST API Requests
                       v
                 FastAPI Backend
                       |
          +------------+-------------+
          |            |             |
          v            v             v
     Inventory &   Risk Monitor   Investigation
     Sales Metrics + Scheduler       Agent
          |            |             |
          +------------+-------------+
                       |
                       v
                 SQLAlchemy ORM
                       |
               +-------+--------+
               |                |
               v                v
           SQLite DB       PostgreSQL DB
               |
               v
       Products, Sales, Risk Alerts,
       Forecasts and Replenishment Actions
```

## Repository Structure

```text
StockPilot-AI-Intelligent-Inventory-Retail-Operations-Agent/
├── app/
│   ├── integrations/
│   │   └── shopify.py
│   ├── agent.py
│   ├── config.py
│   ├── database.py
│   ├── logic.py
│   ├── main.py
│   ├── models.py
│   ├── schemas.py
│   └── seed.py
├── dashboard/
│   └── app.py
├── tests/
│   ├── test_api.py
│   └── test_logic.py
├── .github/
│   └── workflows/
│       └── tests.yml
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── start_windows.bat
├── .env.example
├── .gitignore
└── README.md
```

## Getting Started

### Prerequisites

- Python 3.11 or 3.12 recommended
- Git (optional)
- Docker Desktop for the PostgreSQL Docker setup
- Google Gemini API key for optional AI-generated investigation summaries

### Option 1: Quick Start on Windows

1. Clone the repository or download and extract the project.
2. Open Command Prompt in the project directory.
3. Run:

```cmd
start_windows.bat
```

The startup script is intended to create the virtual environment, install dependencies, and launch the backend and dashboard.

Open the following URLs after the services start:

| Service | URL |
|---|---|
| Streamlit Dashboard | http://localhost:8501 |
| FastAPI Swagger UI | http://localhost:8000/docs |
| API Health Check | http://localhost:8000/health |

### Option 2: Manual Setup

Create a virtual environment and install dependencies from the project root:

```cmd
py -3.12 -m venv .venv
.venv\Scripts\activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Start the backend:

```cmd
uvicorn app.main:app --reload --port 8000
```

Open a second terminal in the project directory, activate the environment, and start the dashboard:

```cmd
.venv\Scripts\activate
streamlit run dashboard\app.py
```

The local configuration uses SQLite by default, so a separate database server is not required for the basic demo.

### Option 3: Docker

Make sure Docker Desktop is running, then execute:

```cmd
docker compose up --build
```

The Docker Compose setup starts the backend, dashboard, and PostgreSQL database.

Access the dashboard at `http://localhost:8501` and the API documentation at `http://localhost:8000/docs`.

To stop the services:

```cmd
docker compose down
```

To preserve the database volume, do not add the `-v` option.

## Configuration

The project supports rules-based operation without a Gemini API key.

For optional Gemini-powered investigations, configure the following environment variables in your local `.env` file:

```env
GEMINI_API_KEY=your_gemini_api_key
GEMINI_MODEL=gemini-2.5-flash
```

Other configuration options include:

- `DATABASE_URL`
- `MONITOR_INTERVAL_SECONDS`
- `SAFETY_BUFFER_DAYS`
- `LOW_STOCK_DAYS`
- `DEAD_STOCK_DAYS`

Check `app/config.py` and `docker-compose.yml` for the exact defaults used by the project.

**Security:** Never commit API keys, passwords, database credentials, or real store access tokens to GitHub.

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/health` | Check API health |
| GET | `/products` | List available products |
| GET | `/products/{sku}` | Retrieve product details and sales metrics |
| GET | `/risks?status=open` | List open inventory risk alerts |
| POST | `/monitor/run` | Trigger inventory monitoring |
| POST | `/forecast/{sku}` | Generate a demand baseline |
| POST | `/agent/investigate/{sku}` | Investigate product risks and recommendations |
| POST | `/actions/replenishment/{sku}` | Create a replenishment proposal |
| GET | `/actions` | List proposals and decisions |
| POST | `/actions/{action_id}/decision` | Approve or reject a proposal |
| POST | `/demo/reset` | Restore the synthetic demo dataset |

Use Swagger UI at `http://localhost:8000/docs` to explore the available endpoints and their request schemas.

## Testing

Activate the project's virtual environment and run:

```cmd
pytest
```

The repository includes tests for API behavior and core business logic, along with a GitHub Actions workflow.

## Limitations and Future Enhancements

- The current application uses synthetic inventory and sales data rather than live retailer data.
- Scheduled monitoring processes the data available in the application; real-time external store synchronization is not yet implemented.
- Demand estimation uses a simple moving-average baseline, not a trained machine-learning forecasting model.
- Risk detection relies on configurable rule-based thresholds.
- The Shopify client is an integration scaffold and requires store credentials and implementation of live synchronization.
- Gemini integration is optional; deterministic rules provide a fallback.
- Replenishment approvals are recorded in the application but do not submit purchase orders.
- Production deployment would require additional security, authentication, authorization, monitoring, and operational safeguards.

Potential future improvements include live e-commerce integrations, richer forecasting models, notifications, role-based access control, background processing, and production deployment.

## Author

**Shruti Mawalkar**

Built as a portfolio project exploring AI-assisted retail operations, inventory analytics, automated monitoring, demand estimation, REST API development, and human-in-the-loop decision workflows.

## Project Links

- **GitHub Repository:** https://github.com/ShrutiMawalkar/StockPilot-AI-Intelligent-Inventory-Retail-Operations-Agent
