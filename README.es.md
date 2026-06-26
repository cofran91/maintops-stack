# MaintOps Developer Stack

Ambiente local con Docker para MaintOps, un proyecto de portafolio enfocado en la gestión de operaciones de mantenimiento. Este repositorio integra la API Laravel, la consola Vue, el gateway Realtime, la API Analytics, MySQL, PostgreSQL, Redis y Mailpit con un solo archivo Compose para que el entorno pueda clonarse y ejecutarse de forma consistente en otra máquina.

Este repositorio contiene únicamente la orquestación del ambiente: Docker Compose, valores locales por defecto y documentación operativa. El código fuente de las aplicaciones vive en submódulos Git.

## Servicios Incluidos

- API Laravel desde `maintops-api-laravel`.
- Consola Vue/Vite desde `maintops-web-vue`.
- Gateway Realtime Node/Express desde `maintops-realtime-node`.
- API Analytics FastAPI desde `maintops-analytics-fastapi`.
- MySQL 8.4 con un volumen local de Docker.
- PostgreSQL 17 con un volumen local de Docker para el modelo de lectura analítico.
- Redis 7 con persistencia append-only y Redis Streams.
- Mailpit para inspección local de correos.

## Requisitos

- Docker Engine o Docker Desktop con Docker Compose v2.
- Git.
- Acceso a los repositorios de los submódulos si se clona desde un remoto privado.

## Clonar

```bash
git clone --recurse-submodules https://github.com/cofran91/maintops-stack.git
cd maintops-stack
```

Si el repositorio fue clonado sin submódulos:

```bash
git submodule update --init --recursive
```

## Iniciar El Ambiente Completo

No necesitas crear un archivo `.env` para desarrollo local. `compose.yaml` ya incluye valores por defecto prácticos. Copia `.env.example` a `.env` solo cuando necesites cambiar puertos, credenciales locales, orígenes CORS, secretos de tokens de servicio o la service key de Analytics.

```bash
docker compose config
docker compose up -d --build
docker compose ps
```

La primera ejecución construye las imágenes, instala las dependencias de aplicación dentro de los contenedores, espera MySQL, PostgreSQL y Redis, ejecuta las migraciones de Laravel, carga los seeders base, ejecuta las migraciones de Analytics, importa el modelo de lectura inicial desde Laravel, inicia el worker de cola, inicia el scheduler de Laravel, levanta el gateway realtime, inicia el worker de Analytics y sirve la consola Vue.

## Servicios

| Servicio | URL | Propósito |
| --- | --- | --- |
| Frontend Vue | http://localhost:5173 | Aplicación web MaintOps Console. |
| API Laravel | http://localhost:8000/api/v1 | API transaccional. |
| Health check Laravel | http://localhost:8000/up | Verificación de disponibilidad del backend. |
| Health check Realtime | http://localhost:3000/ready | Verificación del gateway Socket.IO y Redis. |
| API Analytics | http://localhost:8001 | Métricas administrativas, pronósticos, riesgos y recomendaciones. |
| Health check Analytics | http://localhost:8001/health | Verificación de vida del proceso FastAPI. |
| Readiness Analytics | http://localhost:8001/ready | Verificación de disponibilidad de FastAPI y PostgreSQL. |
| Mailpit | http://localhost:8025 | Bandeja de correos local. |

MySQL se publica solo en localhost:

```text
Host: 127.0.0.1
Port: 3307
Database: maintops
User: maintops
Password: maintops-local-password
```

Redis queda disponible solo dentro de la red de Compose como `redis:6379`.

PostgreSQL de Analytics se publica solo en localhost:

```text
Host: 127.0.0.1
Port: 5433
Database: maintops_analytics
User: maintops_analytics
Password: maintops-analytics-password
```

## Flujo De Ejecución

La consola Vue autentica contra Laravel y solicita un token de servicio de corta duración en `POST /api/v1/auth/service-token` con `audience: "realtime"`. El gateway Realtime valida ese token, une el navegador a salas autorizadas de Socket.IO, consume eventos operativos desde Redis Streams y envía actualizaciones a los usuarios conectados.

Laravel publica eventos operativos en el stream compartido configurado por `MAINTOPS_EVENTS_STREAM`. El stack fija este stream como `maintops:events` tanto para Laravel como para el gateway Realtime.

Las rooms realtime se derivan únicamente de claims firmados por Laravel. Los usuarios siempre reciben rooms `user:<id>` y `role:<role>` permitidas; los usuarios con alcance de taller también reciben `workshop:<id>` y eventos de presencia de taller. Esto permite tanto notificaciones operativas por taller como futuras notificaciones administrativas, por ejemplo eventos de gestión de usuarios.

La API Analytics usa su propio modelo de lectura en PostgreSQL. Al iniciar el stack, `analytics-migrations` aplica las migraciones Alembic y `analytics-initial-sync` importa el snapshot interno de Laravel desde `GET /api/v1/internal/analytics/initial-sync/{resource}` usando `OPERATIONS_ANALYTICS_SERVICE_KEY`. Después de eso, `analytics-worker` consume el mismo Redis Stream que realtime y mantiene actualizado el modelo de lectura.

La consola Vue solicita a Laravel un token de servicio de corta duración con `audience: "analytics"` antes de llamar a FastAPI. Analytics valida ese token con el `SERVICE_TOKEN_SECRET` compartido; el navegador no llama a FastAPI directamente con el token de sesión de Laravel.

## Cuenta Demo

El seeder base crea un usuario administrativo:

```text
email: admin@maint.test
password: password
role: super_admin
```

## Verificación Rápida

```bash
curl http://localhost:8000/up
curl http://localhost:3000/ready
curl http://localhost:8001/health
curl http://localhost:8001/ready
```

Luego abre `http://localhost:5173` e inicia sesión con la cuenta demo. El frontend debería conectarse al gateway realtime después del login, y la pantalla Analytics debería estar disponible para usuarios administrativos.

Logs útiles:

```bash
docker compose logs -f laravel queue scheduler realtime analytics-api analytics-worker frontend
```

## Detener O Reiniciar

Detener el ambiente sin borrar datos:

```bash
docker compose down
```

Borrar los datos de MySQL, PostgreSQL y Redis e iniciar de nuevo:

```bash
docker compose down -v
docker compose up -d --build
```

## Submódulos

Los proyectos de aplicación están incluidos como submódulos Git:

- `maintops-api-laravel`: https://github.com/cofran91/maintops-api-laravel
- `maintops-web-vue`: https://github.com/cofran91/maintops-web-vue
- `maintops-realtime-node`: https://github.com/cofran91/maintops-realtime-node
- `maintops-analytics-fastapi`: https://github.com/cofran91/maintops-analytics-fastapi

Cada repositorio de aplicación también puede abrirse de forma independiente para revisar su instalación, notas de arquitectura, comandos y documentación propia. Los cambios de aplicación deben commitearse primero en sus propios repositorios. Este stack guarda las revisiones exactas de los submódulos y la configuración local necesaria para ejecutarlos juntos.
