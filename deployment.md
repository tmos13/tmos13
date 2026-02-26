# Deployment Guide

> **Coming soon.** This guide is under active development.

Railway, Vercel, Docker, and self-hosted deployment options.

---

## What's Coming

- **Docker** — Production-ready `docker-compose.yml` with engine + Redis
- **Railway** — One-click deploy template
- **Vercel** — Frontend deployment with backend on Railway/Docker
- **Self-hosted** — Bare metal and VM deployment guide
- **Air-gapped** — On-premise deployment with local LLM inference (Ollama)
- **Environment variables** — Complete reference for all configuration options
- **SSL/TLS** — HTTPS configuration for production
- **Scaling** — Horizontal scaling with Redis-backed session state

---

## In the Meantime

The engine runs as a standard FastAPI application:

```bash
# Development
uvicorn engine.app:app --reload --port 8000

# Docker
docker build -t tmos13-engine .
docker run -p 8000:8000 --env-file .env tmos13-engine

# Production (any container platform)
uvicorn engine.app:app --host 0.0.0.0 --port $PORT
```

Minimum environment:

```env
ANTHROPIC_API_KEY=sk-ant-...
TMOS13_ENV=production
TMOS13_PACK=your_pack_id
```

See the [Getting Started](getting-started.md) guide for a quick local setup.

---

## Contact

Questions about deployment? Reach out at dev@tmos13.ai.
