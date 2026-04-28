# 📧 Sandesh Email Service - Public Setup

The **Sandesh Email Service** lets you run a complete notification platform using prebuilt Docker images.

It includes:
- Subscriber management
- Templates
- Event trigger API
- Queue-based processing (Redis worker)
- Web UI + REST API

This repository is **deployment-only**. You don't need source code to run it.

---

## 🚀 Quick Start

### Prerequisites
- Docker
- Docker Compose

### Setup

1. Copy env file:
   ```bash
   cp .env.example .env
   ```
   PowerShell:
   ```powershell
   Copy-Item .env.example .env
   ```

2. Update `.env` values (especially image names, DB URL, and JWT secret).

3. Start:
   ```bash
   docker compose up -d
   ```

4. Open:
   - Frontend: `http://localhost:3000`
   - Backend health: `http://localhost:8000/health`
   - API docs: `http://localhost:8000/docs`

---

## 📦 Required Files in This Public Repo

- `docker-compose.yml`
- `.env.example`
- `README.md`

---

## 🧾 .env.example

```env
# Images
SANDESH_BACKEND_IMAGE=rohithakur0208/sandesh-email-backend:latest
SANDESH_FRONTEND_IMAGE=rohithakur0208/sandesh-email-frontend:latest

# Postgres
POSTGRES_USER=postgres
POSTGRES_PASSWORD=password
POSTGRES_DB=emails
POSTGRES_INITDB_ARGS=--encoding=UTF-8 --lc-collate=C --lc-ctype=C
POSTGRES_EXPOSED_PORT=5433
DATABASE_URL=postgresql://postgres:password@postgres:5432/emails

# Redis
REDIS_URL=redis://redis:6379/0
REDIS_EXPOSED_PORT=6379

# API / App
API_HOST=0.0.0.0
API_PORT=8000
API_EXPOSED_PORT=8000
PYTHONPATH=/app
PYTHONUNBUFFERED=1

# Auth
JWT_SECRET_KEY=change-this-to-a-long-random-secret
JWT_ALGORITHM=HS256
JWT_ACCESS_TOKEN_EXPIRE_MINUTES=1440
API_KEYS=1234

# Worker Queue
QUEUE_WORKER_CONCURRENCY=8
QUEUE_POLL_TIMEOUT_SECONDS=3
QUEUE_MAX_RETRIES=5
QUEUE_RETRY_BACKOFF_SECONDS=2

# Frontend
FRONTEND_EXPOSED_PORT=3000
REACT_APP_API_URL=http://localhost:8000

# Compose
COMPOSE_PROJECT_NAME=sandesh-email-service
```

---

## 🐳 docker-compose.yml

```yaml
services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_INITDB_ARGS: ${POSTGRES_INITDB_ARGS}
    ports:
      - "${POSTGRES_EXPOSED_PORT:-5433}:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 8
      start_period: 30s
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    command: ["redis-server", "--appendonly", "yes"]
    ports:
      - "${REDIS_EXPOSED_PORT:-6379}:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 10
    restart: unless-stopped

  backend:
    image: ${SANDESH_BACKEND_IMAGE}
    environment:
      DATABASE_URL: ${DATABASE_URL}
      REDIS_URL: ${REDIS_URL}
      API_HOST: ${API_HOST}
      API_PORT: ${API_PORT}
      PYTHONPATH: ${PYTHONPATH}
      PYTHONUNBUFFERED: ${PYTHONUNBUFFERED}
      JWT_SECRET_KEY: ${JWT_SECRET_KEY}
      JWT_ALGORITHM: ${JWT_ALGORITHM}
      JWT_ACCESS_TOKEN_EXPIRE_MINUTES: ${JWT_ACCESS_TOKEN_EXPIRE_MINUTES}
      API_KEYS: ${API_KEYS}
    ports:
      - "${API_EXPOSED_PORT:-8000}:${API_PORT}"
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:${API_PORT}/health"]
      interval: 20s
      timeout: 5s
      retries: 8
      start_period: 30s
    restart: unless-stopped

  worker:
    image: ${SANDESH_BACKEND_IMAGE}
    command: ["python", "worker.py"]
    environment:
      DATABASE_URL: ${DATABASE_URL}
      REDIS_URL: ${REDIS_URL}
      PYTHONPATH: ${PYTHONPATH}
      PYTHONUNBUFFERED: ${PYTHONUNBUFFERED}
      QUEUE_WORKER_CONCURRENCY: ${QUEUE_WORKER_CONCURRENCY}
      QUEUE_POLL_TIMEOUT_SECONDS: ${QUEUE_POLL_TIMEOUT_SECONDS}
      QUEUE_MAX_RETRIES: ${QUEUE_MAX_RETRIES}
      QUEUE_RETRY_BACKOFF_SECONDS: ${QUEUE_RETRY_BACKOFF_SECONDS}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      backend:
        condition: service_healthy
    restart: unless-stopped

  frontend:
    image: ${SANDESH_FRONTEND_IMAGE}
    ports:
      - "${FRONTEND_EXPOSED_PORT:-3000}:80"
    depends_on:
      backend:
        condition: service_healthy
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
```

---

## 🧠 First-Time Platform Setup

After containers are up:

1. Open UI and register first user.
2. Configure email integration in UI (SES/SMTP credentials).
3. Create templates.
4. Create subscribers.
5. Trigger events using API or SDK.

> Integration credentials are configured in the app UI, not required in `.env`.

---

## 🐍 Python SDK

Install:

```bash
pip install sandesh-sdk
```

Usage:

```python
from sandesh.sdk import Sandesh

client = Sandesh(
    base_url="http://localhost:8000",
    bearer_token="YOUR_API_KEY_OR_JWT",
)

response = client.events_trigger(
    {
        "name": "welcome-template",
        "to": {"subscriberId": "user-123"},
        "payload": {"name": "Rohit"},
    }
)

print(response)
```

---

## 🔌 API Example

```bash
curl --location 'http://localhost:8000/v1/events/trigger' \
  --header 'Authorization: ApiKey YOUR_API_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "name": "welcome-template",
    "to": { "subscriberId": "user-123" },
    "payload": {
      "name": "Rohit"
    }
  }'
```

---

## 🛠️ Operations

Start:
```bash
docker compose up -d
```

Logs:
```bash
docker compose logs -f
```

Restart:
```bash
docker compose restart
```

Stop:
```bash
docker compose down
```

Stop + remove volumes:
```bash
docker compose down -v
```

---

## ❗Troubleshooting

- Backend unhealthy:
  - `docker compose logs backend`
  - verify Postgres and `DATABASE_URL`
- Worker not processing:
  - `docker compose logs worker`
  - verify Redis and `REDIS_URL`
- Frontend can't call API:
  - verify `API_EXPOSED_PORT` and `REACT_APP_API_URL`

---

## 🔐 Security

- Never commit real `.env`
- Use strong secrets (`JWT_SECRET_KEY`)
- Use HTTPS + reverse proxy for internet-facing deployment

---

## 📄 License

This deployment repo is licensed under the **MIT License**.
