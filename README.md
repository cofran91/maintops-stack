# MaintOps Developer Stack

Local Docker environment for MaintOps, a portfolio project for maintenance operations management. This repository brings the Laravel API, Vue console, Realtime gateway, MySQL, Redis, and Mailpit together with one Compose file so the environment can be cloned and run consistently on another machine.

This repository contains environment orchestration only: Docker Compose, local defaults, and operational documentation. The application source code lives in Git submodules.

## Included Services

- Laravel API from `maintops-api-laravel`.
- Vue/Vite console from `maintops-web-vue`.
- Node/Express Realtime Gateway from `maintops-realtime-node`.
- MySQL 8.4 with a local Docker volume.
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

You do not need to create a `.env` file for local development. `compose.yaml` already includes practical defaults. Copy `.env.example` to `.env` only when you need to change ports, local credentials, CORS origins, or shared realtime secrets.

```bash
docker compose config
docker compose up -d --build
docker compose ps
```

The first run builds the images, installs application dependencies inside the containers, waits for MySQL and Redis, runs Laravel migrations, loads the base seeders, starts the Laravel queue worker and scheduler, and starts the realtime gateway.

## Services

| Service | URL | Purpose |
| --- | --- | --- |
| Vue frontend | http://localhost:5173 | MaintOps Console web app. |
| Laravel API | http://localhost:8000/api/v1 | Transactional API. |
| Laravel health check | http://localhost:8000/up | Backend availability check. |
| Realtime health check | http://localhost:3000/ready | Socket.IO gateway and Redis readiness check. |
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

## Runtime Flow

The Vue console authenticates against Laravel and requests a short-lived realtime token from `POST /api/v1/auth/realtime-token`. The Realtime Gateway validates that token, joins the browser to authorized Socket.IO rooms, consumes operational events from Redis Streams, and broadcasts updates to connected users.

Laravel publishes operational events to the shared stream configured by `MAINTOPS_EVENTS_STREAM`. The stack sets this stream to `maintops:events` for both Laravel and the Realtime Gateway.

Realtime rooms are derived only from Laravel-signed token claims. Users always receive `user:<id>` and allowed `role:<role>` rooms; users with a workshop scope also receive `workshop:<id>` and workshop presence updates. This allows both workshop-scoped operational notifications and future administrative notifications such as user-management events.

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
```

Then open `http://localhost:5173` and sign in with the demo account. The frontend should connect to the realtime gateway after login.

Useful logs:

```bash
docker compose logs -f laravel queue scheduler realtime frontend
```

## Stop Or Reset

Stop the environment without deleting data:

```bash
docker compose down
```

Delete MySQL and Redis data and start again:

```bash
docker compose down -v
docker compose up -d --build
```

## Submodules

The application projects are included as Git submodules:

- `maintops-api-laravel`: https://github.com/cofran91/maintops-api-laravel
- `maintops-web-vue`: https://github.com/cofran91/maintops-web-vue
- `maintops-realtime-node`: https://github.com/cofran91/maintops-realtime-node

Each application repository can also be opened independently if you want to review its own setup, architecture notes, commands, and documentation. Application changes should be committed in their own repositories first. This stack stores the exact submodule revisions plus the local environment configuration needed to run them together.
