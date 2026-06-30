# MaintOps Developer Stack

Spanish documentation: [README.es.md](README.es.md).

MaintOps is a multi-service portfolio project for vehicle maintenance operations. This repository is the reproducible local environment for the full demo: it brings the Laravel API, Vue web console, Node realtime gateway, FastAPI analytics service, MySQL, PostgreSQL, Redis, and Mailpit together with one Docker Compose file.

This repository does not own application source code. The applications live in Git submodules and can be reviewed independently. The stack owns orchestration, local defaults, service wiring, submodule revisions, and the operating guide for running the complete system.

## Portfolio Purpose

The stack is designed to show how the individual services work together as one product:

- Laravel owns authentication, authorization, transactional data, domain workflows, audit records, scheduled automation, localized API responses, operational events, and emails.
- Vue owns the browser experience: bilingual authenticated console, role-aware navigation, CRUD workflows, realtime notifications, and analytics views.
- Node owns realtime delivery: signed-token Socket.IO authentication, authorized rooms, Redis Streams consumption, presence, event deduplication, and dead-letter handling.
- FastAPI owns analytics: PostgreSQL read model, initial sync from Laravel, Redis Streams projection, observed metrics, forecasts, risk alerts, recommendations, and structured codes that the bilingual console can translate.

Each service can be studied on its own, but the intended review path is the full stack because several capabilities only make sense when the services are connected.

## What The Product Simulates

MaintOps simulates the daily operation of a vehicle maintenance company.

The product is not only a CRUD demo. It models how a service advisor receives a customer vehicle, how maintenance work is organized into an order, how work items are approved and scheduled, how technicians execute assigned work, and how administrators monitor the operation through dashboards, realtime notifications, email, and analytics.

Main business concepts:

| Concept | Meaning In The Demo |
| --- | --- |
| Owner | Customer who owns one or more vehicles. Owners are domain records, not interactive console users. |
| Vehicle | Vehicle that receives maintenance. It belongs to an owner and stores data such as license plate and odometer. |
| Workshop | Physical or operational location where maintenance work is performed. |
| Technician | User who performs assigned maintenance work. |
| Vehicle system | Area of expertise such as engine, brakes, electrical, cooling, tires, or suspension. Workshops and technicians can be scoped by supported systems. |
| Maintenance task | Work that can be performed, such as checking brakes, changing oil, or diagnosing an electrical issue. |
| Maintenance plan | Set of recurring tasks recommended by time or mileage. |
| Maintenance order | Main work request for a vehicle. It groups the maintenance work that should be reviewed, approved, scheduled, and executed. |
| Order item | Individual piece of work inside an order. Items can be approved, rejected, scheduled, started, completed, or cancelled. |
| Approval | Customer-facing decision about which order items should be performed. |
| Scheduling | Assignment of approved work to a workshop, technician, and planned time. |

The expected functional flow is:

1. An administrator prepares operational data: users, roles, workshops, technicians, vehicle systems, maintenance tasks, and maintenance plans.
2. An advisor registers or reviews an owner and vehicle.
3. An advisor creates a maintenance order for that vehicle.
4. The system can add recommended order items from due maintenance plans and active vehicle-specific issues.
5. The owner approves or rejects each proposed item. Accepted items move forward; rejected items remain as decision history.
6. Approved items are scheduled to a workshop and technician based on capability, working hours, availability, and expected duration.
7. A technician signs in and sees only assigned work.
8. The technician starts and completes assigned order items.
9. The order can be completed and delivered.
10. Realtime notifications, owner-facing email, dashboard data, audit records, and analytics update from the same operational changes.

You do not need to create this whole flow from scratch on first review. The seeders include demo data across the main lifecycle states so reviewers can inspect existing orders, users, workshops, tasks, notifications, and analytics immediately.

## Included Services

| Service | Project | Responsibility |
| --- | --- | --- |
| Laravel API | `maintops-api-laravel` | Transactional backend, auth, policies, state machines, scheduling, audit trail, emails, and integration contracts. |
| Vue console | `maintops-web-vue` | Browser console for operations, maintenance records, realtime activity, and analytics. |
| Realtime gateway | `maintops-realtime-node` | Socket.IO gateway that consumes Laravel operational events from Redis Streams. |
| Analytics API | `maintops-analytics-fastapi` | Read-only analytics service with its own PostgreSQL read model. |
| MySQL | Docker image | Laravel transactional database. |
| PostgreSQL | Docker image | Analytics read model database. |
| Redis | Docker image | Shared Redis Streams transport and realtime support. |
| Mailpit | Docker image | Local email inbox for password recovery and operational owner emails. |

## Requirements

- Docker Engine or Docker Desktop with Docker Compose v2.
- Git.
- Network access to GitHub to clone this repository and its public submodules.

## Clone

```bash
git clone --recurse-submodules https://github.com/cofran91/maintops-stack.git
cd maintops-stack
```

If the repository was cloned without submodules:

```bash
git submodule update --init --recursive
```

The submodules use public HTTPS URLs so the stack can be cloned without SSH setup. Developers who prefer SSH can configure Git locally with an `insteadOf` rule.

## Start The Full Demo

You do not need to create a `.env` file for local development. `compose.yaml` already includes practical defaults. Copy `.env.example` to `.env` only when you need to change ports, local credentials, CORS origins, queue names, or shared service-token settings.

```bash
docker compose config
docker compose up -d --build
docker compose ps
```

The first run builds the images, installs application dependencies inside the containers, waits for MySQL, PostgreSQL, and Redis, runs Laravel migrations and seeders, runs Analytics migrations, imports the initial Analytics read model from Laravel, starts Laravel queue workers and scheduler, starts Realtime, starts the Analytics worker, and serves the Vue console.

## Service URLs

| Surface | URL | Use |
| --- | --- | --- |
| Vue frontend | http://localhost:5173 | Main MaintOps Console. |
| Laravel API | http://localhost:8000/api/v1 | Transactional API root. |
| Laravel health check | http://localhost:8000/up | Backend availability check. |
| Realtime readiness | http://localhost:3000/ready | Socket.IO gateway and Redis readiness check. |
| Analytics API | http://localhost:8001 | Read-only operational analytics. |
| Analytics health check | http://localhost:8001/health | FastAPI liveness check. |
| Analytics readiness | http://localhost:8001/ready | FastAPI and PostgreSQL readiness check. |
| Mailpit | http://localhost:8025 | Local inbox for demo emails. |

MySQL is published on localhost only:

```text
Host: 127.0.0.1
Port: 3307
Database: maintops
User: maintops
Password: maintops-local-password
```

Analytics PostgreSQL is published on localhost only:

```text
Host: 127.0.0.1
Port: 5433
Database: maintops_analytics
User: maintops_analytics
Password: maintops-analytics-password
```

Redis is available only inside the Compose network as `redis:6379`.

## Demo Account

The base seeders create demo users for the main roles. All demo accounts use:

```text
password: password
```

Recommended accounts:

| Role | Email | What To Review |
| --- | --- | --- |
| Super admin | `admin@maint.test` | Full local review, internal tools, API docs, audit, and unrestricted administration. |
| Admin | `admin.demo@maint.test` | Dashboard, catalogs, users, workshops, orders, audit, and Analytics. |
| Workshop manager | `manager.north@maint.test` | Workshop-scoped operation and restricted visibility. |
| Advisor | `advisor.north@maint.test` | Owner, vehicle, and maintenance order workflows. |
| Technician | `technician.engine@maint.test` | Assigned work only and executable item transitions. |

## Suggested Review Path

For a visual, step-by-step public demo guide, open:

```text
https://juandavidfranco.com/demo
```

Use the same review logic locally with these stack URLs:

- Vue console: `http://localhost:5173`
- Mailpit inbox: `http://localhost:8025`
- Laravel API docs: `http://localhost:8000/docs`
- Analytics API docs: `http://localhost:8001/docs`

For local review, start with `admin@maint.test` because it is the `super_admin` account and can inspect the full environment, internal tools, audit, and API documentation. Then switch to `admin.demo@maint.test`, `manager.north@maint.test`, `advisor.north@maint.test`, and `technician.engine@maint.test` to validate role-scoped behavior.

Use sample data only. Demo environments should not receive real customer data, vehicle data, addresses, passwords, phone numbers, documents, or personal email addresses. Mailpit is intentionally exposed in the local stack so reviewers can inspect generated demo emails.

## Runtime Flow

The Vue console authenticates against Laravel. Laravel remains the identity and authorization source of truth.

For realtime updates, the browser asks Laravel for a short-lived service token with `audience: "realtime"` and uses it during the Socket.IO handshake. The Node gateway validates the signed token, consumes Laravel operational events from Redis Streams, and delivers only authorized events to connected users.

For analytics, the browser asks Laravel for a short-lived service token with `audience: "analytics"` and uses it when calling FastAPI. Analytics validates the same shared service-token contract and reads from its own PostgreSQL model instead of reading Laravel's MySQL database directly.

Laravel publishes operational events to the shared Redis Stream configured by `MAINTOPS_EVENTS_STREAM`. Realtime and Analytics consume the same stream with separate consumer groups, so live UI delivery and analytical projection can evolve independently.

If you override `SERVICE_TOKEN_SECRET`, use the same value for Laravel, Realtime, and Analytics; otherwise Realtime sockets and Analytics requests will be rejected.

## Queues And Email

The stack runs three Laravel queue workers:

- `queue` processes default application jobs.
- `queue-events` publishes operational events to Redis Streams.
- `queue-mail` sends queued emails through Mailpit.

Mailpit captures application emails instead of sending them to the internet. It is used for password recovery emails and operational owner emails.

Password recovery is local by default:

1. Open `http://localhost:5173`.
2. Use the "Forgot your password?" link on the login screen.
3. Check the reset email in Mailpit at `http://localhost:8025`.
4. Open the reset link. Laravel generates reset links with `FRONTEND_PASSWORD_RESET_URL`, which defaults to `http://localhost:5173/reset-password`.

## Verification

```bash
curl http://localhost:8000/up
curl http://localhost:3000/ready
curl http://localhost:8001/health
curl http://localhost:8001/ready
```

Useful logs for long-running services:

```bash
docker compose logs -f laravel queue queue-events queue-mail scheduler realtime analytics-api analytics-worker frontend
```

If startup stops before the long-running services are healthy, inspect the one-shot initialization services:

```bash
docker compose logs laravel-init analytics-migrations analytics-initial-sync
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

Each application repository documents its own technology choices, folder structure, commands, tests, and integration boundaries. Application changes should be committed in their own repositories first. This stack stores the exact submodule revisions plus the local environment configuration needed to run them together.

## Deployment Boundary

Deployment-specific reverse proxy configuration is intentionally outside this repository. This stack is focused on the reproducible local environment used to review the portfolio demo.
