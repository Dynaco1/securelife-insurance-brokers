# SecureLife Backend Service

FastAPI application for the SecureLife Intelligent Lead Pipeline, providing REST APIs, LangGraph conversational agent execution, PyMuPDF OCR document processing, deterministic qualification scoring, and PostgreSQL database persistence.

For complete project documentation, architecture design, and frontend setup, refer to the root [README.md](../README.md).

---

## Quick Reference Commands

### Running with Docker Compose (Recommended)

From this directory:

```bash
# Build and start PostgreSQL and Backend containers
docker compose up -d --build

# Run Alembic migrations to latest schema
docker exec securelife_backend_dev uv run alembic upgrade head

# Seed database with test users and leads
docker exec securelife_backend_dev uv run python -m app.db.seed

# Run backend test suite
docker exec securelife_backend_dev uv run pytest
```

### Running Locally with uv

```bash
# Install dependencies into virtual environment
uv sync

# Configure environment variables
cp .env.example .env

# Run database migrations
uv run alembic upgrade head

# Seed initial test data
uv run python -m app.db.seed

# Start development server with hot-reload
uv run uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

---

## API Documentation

When the backend is running:
- **Swagger Interactive UI**: [http://localhost:8000/docs](http://localhost:8000/docs)
- **ReDoc UI**: [http://localhost:8000/redoc](http://localhost:8000/redoc)
