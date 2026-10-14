# SecureLife Intelligent Lead Pipeline & Broker CRM

An enterprise-grade insurance lead qualification, customer relationship management (CRM), and intelligent document processing (IDP) platform built for modern brokerage teams.

---

## Architecture Overview

```
                      ┌───────────────────────────────────────┐
                      │    Prospect Web Chat Widget (FRD-01)  │
                      │    Broker Pipeline Portal   (FRD-05)  │
                      │    React 19 + Redux-Saga + Tailwind   │
                      └──────────────────┬────────────────────┘
                                         │ HTTP REST / SSE
                                         ▼
                      ┌───────────────────────────────────────┐
                      │          FastAPI ASGI Gateway         │
                      │   JWT Auth · CORS · Pydantic v2       │
                      └──────────────────┬────────────────────┘
                                         │
        ┌────────────────────────────────┼────────────────────────────────┐
        ▼                                ▼                                ▼
┌───────────────┐              ┌──────────────────┐             ┌───────────────────┐
│  LangGraph    │              │    PyMuPDF +     │             │  Deterministic    │
│  Dialogue     │              │  OCR Extraction  │             │  Scoring & Rules  │
│  State Machine│              │   Engine (FRD-03)│             │  (FRD-02, FRD-04) │
└───────┬───────┘              └────────┬─────────┘             └─────────┬─────────┘
        │                               │                                 │
        └───────────────────────────────┼─────────────────────────────────┘
                                        ▼
                      ┌───────────────────────────────────────┐
                      │    PostgreSQL 15+ Relational Storage  │
                      │    Leads · Documents · Analyses · Logs│
                      └───────────────────────────────────────┘
```

---

## Service Endpoints & Default Ports

| Service | URL | Purpose |
|---|---|---|
| **Frontend Web App** | [http://localhost:5173](http://localhost:5173) | Public conversational intake chat & Broker Portal |
| **Broker CRM Dashboard** | [http://localhost:5173/dashboard](http://localhost:5173/dashboard) | Pipeline Kanban, Leads Directory & Review Queue |
| **Backend API Gateway** | [http://localhost:8000](http://localhost:8000) | FastAPI REST service |
| **Interactive API Docs (Swagger)** | [http://localhost:8000/docs](http://localhost:8000/docs) | OpenAPI interactive schema and test console |
| **Alternative API Docs (ReDoc)** | [http://localhost:8000/redoc](http://localhost:8000/redoc) | Clean API documentation reference |
| **PostgreSQL Database** | `localhost:5432` | Relational database (Default DB: `securelife`) |

---

## Prerequisites

Before starting, ensure you have the following installed on your machine:

* **Docker & Docker Compose** (Docker Desktop on Windows/macOS or Docker Engine on Linux)
* *(Optional for native development)*:
  * **Python 3.11+** with [`uv`](https://docs.astral.sh/uv/) package manager
  * **Node.js 20+** and **npm**

---

## Step-by-Step Installation (Docker Compose — Recommended)

### 1. Environment Configuration

Copy the example environment files for both backend and frontend:

```bash
# Backend environment configuration
cp backend/.env.example backend/.env

# Frontend environment configuration
cp frontend/.env.example frontend/.env
```

> **Note on LLM API Keys**:
> By default, the application runs with `LLM_PROVIDER=mock` for testing without external API dependencies.
> To enable live LLM inference with Groq or OpenAI, edit `backend/.env`:
> ```env
> LLM_PROVIDER=groq
> GROQ_API_KEY=your_groq_api_key_here
> ```

---

### 2. Build and Launch Containers

Start the PostgreSQL database and backend service:

```bash
cd backend
docker compose up -d --build
```

Start the frontend service:

```bash
cd ../frontend
docker compose up -d --build
```

Verify that all three containers are healthy and running:

```bash
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

Expected output:
```
NAMES                     STATUS                    PORTS
securelife_backend_dev    Up (healthy)              0.0.0.0:8000->8000/tcp
securelife_postgres_dev   Up (healthy)              0.0.0.0:5432->5432/tcp
securelife_frontend_dev   Up                        0.0.0.0:5173->5173/tcp
```

---

## Database Migrations (Alembic)

Database tables are managed version-by-version using Alembic migrations.

### Run Migrations to Latest Schema (`head`)

**Inside Docker container:**
```bash
docker exec securelife_backend_dev uv run alembic upgrade head
```

*Alternative via Docker Compose (from `backend/` directory):*
```bash
cd backend
docker compose exec backend uv run alembic upgrade head
```

*Alternative for local native development:*
```bash
cd backend
uv run alembic upgrade head
```

### Roll Back Migrations

To downgrade by one migration step:
```bash
docker exec securelife_backend_dev uv run alembic downgrade -1
```

### Create a New Database Migration

If you update SQLAlchemy models in `backend/app/db/models.py`:
```bash
docker exec securelife_backend_dev uv run alembic revision --autogenerate -m "add_new_table_or_field"
```

---

## Database Seeding Instructions

The seed script creates initial team accounts (Brokers, Agents, Admins) and sample insurance leads across all pipeline stages with messages, notes, and documents.

### Run the Seed Script

**Inside Docker container:**
```bash
docker exec securelife_backend_dev uv run python -m app.db.seed
```

*Alternative for local native development:*
```bash
cd backend
uv run python -m app.db.seed
```

Expected output:
```
[*] Checking for existing database records...
[*] Cleared old seed data.
[+] Seeded 3 team users.
[+] Seeded sample leads across pipeline stages.
[+] Seeding complete! Database is ready for development.
```

> **Tip for Clean Reseed**:
> By design, `seed.py` skips generation if records already exist. To perform a fresh reseed from scratch:
> ```bash
> # Roll back all migrations to base, re-apply, and reseed
> docker exec securelife_backend_dev uv run alembic downgrade base
> docker exec securelife_backend_dev uv run alembic upgrade head
> docker exec securelife_backend_dev uv run python -m app.db.seed
> ```

---

## Pre-Configured Test Accounts

After running the seed script, log in to the Broker Portal at [http://localhost:5173/dashboard](http://localhost:5173/dashboard):

| Role | Email | Password | Full Name | Capabilities |
|---|---|---|---|---|
| **Senior Broker** | `broker@securelife.com` | `password123` | Rajesh Sharma | Full pipeline access, lead reassignment, document underwriting approval |
| **Underwriting Agent** | `agent@securelife.com` | `password123` | Pooja Verma | Lead intake, document OCR verification, notes |
| **Admin** | `admin@securelife.com` | `password123` | Vikram Mehta | Full system and user administration |

---

## Local Native Setup (Without Docker)

If you prefer to run services directly on your host machine:

### 1. Backend Setup
```bash
cd backend

# Install dependencies using uv
uv sync

# Copy environment variables
cp .env.example .env

# Run database migrations
uv run alembic upgrade head

# Seed initial test data
uv run python -m app.db.seed

# Start FastAPI development server with auto-reload
uv run uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

### 2. Frontend Setup
```bash
cd frontend

# Install npm dependencies
npm install

# Start Vite development server
npm run dev
```

Visit [http://localhost:5173](http://localhost:5173) in your browser.

---

## Testing & Quality Assurance

### Run Backend Unit & Integration Tests (pytest)
```bash
# In Docker
docker exec securelife_backend_dev uv run pytest

# Locally
cd backend
uv run pytest
```

### Run Frontend Tests, Linting & Build Validation
```bash
cd frontend

# Run all Vitest unit & component tests
npm test -- --run

# Run ESLint check
npm run lint

# Run TypeScript compilation & production build check
npm run build
```

---

## Project Directory Map

```
securelife-insurance-brokers/
├── backend/
│   ├── alembic/              # Alembic database migrations and version files
│   ├── alembic.ini           # Alembic configuration
│   ├── app/
│   │   ├── api/routers/      # REST API endpoints (leads, chat, documents, analysis, auth)
│   │   ├── core/             # Configuration, security, JWT auth, logging
│   │   ├── db/               # SQLAlchemy models (Lead, Document, Analysis, etc.) & seed script
│   │   ├── services/         # Business logic (qualification, OCR, analysis, LangGraph chat)
│   │   └── main.py           # FastAPI application entrypoint
│   ├── docker/               # Dockerfile.dev for backend
│   ├── docker-compose.yml    # Docker Compose definition (FastAPI + PostgreSQL)
│   └── pyproject.toml        # Python project dependencies managed via uv
├── frontend/
│   ├── src/
│   │   ├── features/
│   │   │   ├── chat/         # Conversational intake widget (FRD-01)
│   │   │   ├── crm/          # Broker Pipeline Kanban, Leads Table, Review Queue (FRD-05)
│   │   │   ├── documents/    # Document Uploader, PDF Viewer, Extraction Editor (FRD-03)
│   │   │   └── analysis/     # Coverage Gap & Savings Decision Cards (FRD-04)
│   │   ├── store/            # Redux Toolkit root reducer and Redux-Saga root
│   │   ├── utils/            # Status styles & design tokens (statusStyles.ts)
│   │   ├── App.tsx           # Main application routing & auth gate
│   │   └── main.tsx          # React application root
│   ├── docker-compose.yml    # Docker Compose definition for frontend
│   └── package.json          # Node dependencies and scripts
└── docs/                     # Full PRD and Functional Requirement Documents (FRD-01 to FRD-05)
```

---

## Documentation Index

| Document | Purpose |
|---|---|
| [**Functional Overview**](docs/FUNCTIONAL_OVERVIEW.md) | Business workflows, target personas, FRD-01 to FRD-05 functional breakdown, and ROI metrics |
| [**System Design & Scalability**](docs/SYSTEM_DESIGN_AND_SCALABILITY.md) | High-level system architecture, component design, database scaling, async worker pool, and P95 SLAs |
| [**Architecture Evaluation & Failure Modes**](docs/ARCHITECTURE_EVALUATION_AND_FAILURE_MODES.md) | Problem decomposition, tech stack justifications, and in-depth failure mode mitigations |
| [**Product Requirements Document (PRD)**](docs/PRD.md) | Complete product scope, user personas, MVP goals, and non-goals |
| [**FRD-01: Conversational Chatbot**](docs/FRD-01-lead-capture-chatbot.md) | LangGraph dialogue state machine, slot filling, and handoff specifications |
| [**FRD-02: Data Storage & CRM Layer**](docs/FRD-02-data-storage.md) | PostgreSQL schema, deterministic scoring engine, and core REST APIs |
| [**FRD-03: Document Processing (IDP)**](docs/FRD-03-document-processing.md) | PyMuPDF rasterization, OCR extraction, confidence scoring, and review queue |
| [**FRD-04: Intelligent Analysis**](docs/FRD-04-intelligent-analysis.md) | Coverage gap detector, savings calculator, and executive broker dossier |
| [**FRD-05: Broker Dashboard & Portal**](docs/FRD-05-dashboard.md) | Multi-view pipeline (Kanban, Funnel, Table), leads directory, and journey timeline |

