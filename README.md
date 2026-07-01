# Toutiao Backend

FastAPI backend practice project for a news application.

## Features

- News APIs
- User APIs
- Favorite and history modules
- Async SQLAlchemy with MySQL
- Redis cache helpers
- CORS and custom exception handlers

## Run

```bash
uv sync
copy .env.example .env
uv run uvicorn main:app --reload
```

Set `ASYNC_DATABASE_URL` and Redis values in `.env` for your local environment.
