# MaintOps Developer Stack

Local Docker environment for MaintOps, a portfolio project for maintenance operations management. This repository brings the Laravel API, Vue console, Realtime gateway, Analytics API, MySQL, PostgreSQL, Redis, and Mailpit together with one Compose file so the environment can be cloned and run consistently on another machine.

This repository contains environment orchestration only: Docker Compose, local defaults, and operational documentation. The application source code lives in Git submodules.

## Included Services

- Laravel API from `maintops-api-laravel`.
- Vue/Vite console from `maintops-web-vue`.
- Node/Express Realtime Gateway from `maintops-realtime-node`.
- FastAPI Analytics API from `maintops-analytics-fastapi`.
- MySQL 8.4 with a local Docker volume.
- PostgreSQL 17 with a local Docker volume for the analytics read model.
- Redis 7 with append-only persistence and Redis Streams.
- Mailpit for local email inspection.

## Requirements

- Docker Engine or Docker Desktop with Docker Compose v2.
- Git.
- Access to the submodule repositories when cloning from a private remote.

## Clone

```bash
git clone --recurse-submodules https://github.com/cofran91/maintops-stack.git
cd maintops-stack
```

If the repository was cloned without submodules:

```bash
git submodule update --init --recursive
```

## Start The Full Environment

You do not need to create a `.env` file for local development. `compose.yaml` already includes practical defaults. Copy `.env.example` to `.env` only when you need to change ports, local credentials, CORS origins, service token secrets, or the Analytics service key.

```bash
docker compose config
docker compose up -d --build
docker compose ps
```

The first run builds the images, installs application dependencies inside the containers, waits for MySQL, PostgreSQL, and Redis, runs Laravel migrations, loads the base seeders, runs Analytics migrations, imports the initial Analytics read model from Laravel, starts the Laravel queue worker and scheduler, starts the realtime gateway, starts the Analytics worker, and serves the Vue console.

## Services

| Service | URL | Purpose |
| --- | --- | --- |
| Vue frontend | http://localhost:5173 | MaintOps Console web app. |
| Laravel API | http://localhost:8000/api/v1 | Transactional API. |
| Laravel health check | http://localhost:8000/up | Backend availability check. |
| Realtime health check | http://localhost:3000/ready | Socket.IO gateway and Redis readiness check. |
| Analytics API | http://localhost:8001 | Administrative metrics, forecasts, risks, and recommendations. |
| Analytics health check | http://localhost:8001/health | FastAPI process liveness check. |
| Analytics readiness check | http://localhost:8001/ready | FastAPI and PostgreSQL readiness check. |
| Mailpit | http://localhost:8025 | Local email inbox. |

MySQL is published on localhost only:

```text
Host: 127.0.0.1
Port: 3307
Database: maintops
User: maintops
Password: maintops-local-password
```

Redis is available only inside the Compose network as `redis:6379`.

Analytics PostgreSQL is published on localhost only:

```text
Host: 127.0.0.1
Port: 5433
Database: maintops_analytics
User: maintops_analytics
Password: maintops-analytics-password
```

## Runtime Flow

The Vue console authenticates against Laravel and requests a short-lived service token from `POST /api/v1/auth/service-token` with `audience: "realtime"`. The Realtime Gateway validates that token, joins the browser to authorized Socket.IO rooms, consumes operational events from Redis Streams, and broadcasts updates to connected users.

Laravel publishes operational events to the shared stream configured by `MAINTOPS_EVENTS_STREAM`. The stack sets this stream to `maintops:events` for both Laravel and the Realtime Gateway.

Realtime rooms are derived only from Laravel-signed token claims. Users always receive `user:<id>` and allowed `role:<role>` rooms; users with a workshop scope also receive `workshop:<id>` and workshop presence updates. This allows both workshop-scoped operational notifications and future administrative notifications such as user-management events.

The Analytics API uses its own PostgreSQL read model. On stack startup, `analytics-migrations` applies Alembic migrations and `analytics-initial-sync` imports Laravel's internal snapshot through `GET /api/v1/internal/analytics/initial-sync/{resource}` using `OPERATIONS_ANALYTICS_SERVICE_KEY`. After that, `analytics-worker` consumes the same Redis Stream as realtime and keeps the read model updated.

The Vue console requests a short-lived service token from Laravel with `audience: "analytics"` before calling FastAPI. Analytics validates that token with the shared `SERVICE_TOKEN_SECRET`; the browser never calls FastAPI with the Laravel session token directly.

## Demo Account

The base seeder creates an administrative user:

```text
email: admin@maint.test
password: password
role: super_admin
```

## Quick Verification

```bash
curl http://localhost:8000/up
curl http://localhost:3000/ready
curl http://localhost:8001/health
curl http://localhost:8001/ready
```

Then open `http://localhost:5173` and sign in with the demo account. The frontend should connect to the realtime gateway after login, and the Analytics screen should be available to administrative users.

Useful logs:

```bash
docker compose logs -f laravel queue scheduler realtime analytics-api analytics-worker frontend
```

## Stop Or Reset

Stop the environment without deleting data:

```bash
docker compose down
```

Delete MySQL, PostgreSQL, and Redis data and start again:

```bash
docker compose down -v
docker compose up -d --build
```

## Submodules

The application projects are included as Git submodules:

- `maintops-api-laravel`: https://github.com/cofran91/maintops-api-laravel
- `maintops-web-vue`: https://github.com/cofran91/maintops-web-vue
- `maintops-realtime-node`: https://github.com/cofran91/maintops-realtime-node
- `maintops-analytics-fastapi`: https://github.com/cofran91/maintops-analytics-fastapi

Each application repository can also be opened independently if you want to review its own setup, architecture notes, commands, and documentation. Application changes should be committed in their own repositories first. This stack stores the exact submodule revisions plus the local environment configuration needed to run them together.
