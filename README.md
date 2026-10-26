# Rudo.ai

> "Red Sudo on ur system" — an AI Security Assessment System: one dashboard, four scanning agents, and an AI brain that plans, analyzes and reports.

![CI](https://github.com/mvzen-ash/rudo.ai/actions/workflows/ci_checks.yml/badge.svg)
**Repo:** https://github.com/mvzen-ash/rudo.ai

## What it is

One dashboard, four specialized scanning agents, and an AI brain that plans each scan, analyzes the results, maps attack paths, and answers questions through the in-app **Ask Rudo.ai** chat. Built for an IT employee at any organization — small company to bank — who needs a full, prioritized security report without being a pentest expert.

| Agent | Does |
|---|---|
| **Network** | Maps the network, runs known attack checks, draws it for the specialist |
| **Web App** | Same idea, scoped to the company's own web app |
| **Advanced** | Deeper scan profile with more features |
| **Vulnerability Searching** | Pulls CVE/severity from trusted sources, produces the fixed-template technical report |

## Tech stack

| Layer | Choice |
|---|---|
| Frontend | React 19 + Vite, built in Docker, served by Nginx, installable as a PWA |
| Backend | Python 3.11 + FastAPI + Uvicorn |
| Database | PostgreSQL 15 |
| Queue / workers | Redis 7 + Celery 5 |
| AI | Ollama (llama3) on the host machine, Groq API as cloud fallback |
| Web server | Nginx — reverse proxy, static files, WebSocket, PWA headers |
| Orchestration | Docker Compose |
| CI/CD | GitHub Actions |

## Network

```
Port 80    (Nginx)       → serves frontend + proxies /api/* to backend
Port 8000  (FastAPI)     → internal only
Port 5432  (PostgreSQL)  → internal only
Port 6379  (Redis)       → internal only
Port 11434 (Ollama)      → on the HOST machine, outside Docker
```

## Repo structure

```
rudo.ai/
├── app/                        # Frontend (React + Vite)
├── backend/
│   └── rudo/                   # FastAPI app, agents, AI brain
├── tests/
├── docs/
├── scripts/
│   ├── run_platform.sh / .bat
│   └── setup_local_ollama.sh / .bat
├── .github/workflows/ci_checks.yml
├── docker-compose.yml
├── docker-compose.dev.yml
├── nginx.conf
├── .env.example
├── .gitignore
└── README.md
```

## Getting started

```bash
git clone https://github.com/mvzen-ash/rudo.ai.git
cd rudo.ai
cp .env.example .env
./scripts/setup_local_ollama.sh
docker compose up --build -d
# open http://localhost
```

## Key environment variables

```env
POSTGRES_USER=rudo
POSTGRES_PASSWORD=change-me
POSTGRES_DB=rudo_db
DATABASE_URL=postgresql+asyncpg://rudo:change-me@postgres:5432/rudo_db
REDIS_URL=redis://redis:6379/0
SECRET_KEY=change-me
GROQ_API_KEY=your-groq-api-key
OLLAMA_BASE_URL=http://host.docker.internal:11434/v1
HIBP_API_KEY=your-haveibeenpwned-api-key
APP_ENV=development
CORS_ORIGINS=http://localhost:5173,http://localhost:80
```

## Working rules

- `main` stays protected and green; build on short-lived branches off `develop`, merge once CI passes.
- One commit = one working slice (one agent, one endpoint, one screen).
- Keep `backend/rudo/` import paths and `.env` keys consistent as the codebase grows.

## CI

`.github/workflows/ci_checks.yml`:
1. `frontend-check` — lint + build the React app
2. `backend-check` — compile + pytest the FastAPI app
3. `docker-build-check` — full `docker compose up` + `curl /api/health`
