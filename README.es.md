# MaintOps Developer Stack

Ambiente local con Docker para MaintOps, un proyecto de portafolio enfocado en la gestión de operaciones de mantenimiento. Este repositorio integra la API Laravel, la consola Vue, el gateway Realtime, MySQL, Redis y Mailpit con un solo archivo Compose para que el entorno pueda clonarse y ejecutarse de forma consistente en otra máquina.

Este repositorio contiene únicamente la orquestación del ambiente: Docker Compose, valores locales por defecto y documentación operativa. El código fuente de las aplicaciones vive en submódulos Git.

## Servicios Incluidos

- API Laravel desde `maintops-api-laravel`.
- Consola Vue/Vite desde `maintops-web-vue`.
- Gateway Realtime Node/Express desde `maintops-realtime-node`.
- MySQL 8.4 con un volumen local de Docker.
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

No necesitas crear un archivo `.env` para desarrollo local. `compose.yaml` ya incluye valores por defecto prácticos. Copia `.env.example` a `.env` solo cuando necesites cambiar puertos, credenciales locales, orígenes CORS o secretos compartidos de realtime.

```bash
docker compose config
docker compose up -d --build
docker compose ps
```

La primera ejecución construye las imágenes, instala las dependencias de aplicación dentro de los contenedores, espera MySQL y Redis, ejecuta las migraciones de Laravel, carga los seeders base, inicia el worker de cola, inicia el scheduler de Laravel y levanta el gateway realtime.

## Servicios

| Servicio | URL | Propósito |
| --- | --- | --- |
| Frontend Vue | http://localhost:5173 | Aplicación web MaintOps Console. |
| API Laravel | http://localhost:8000/api/v1 | API transaccional. |
| Health check Laravel | http://localhost:8000/up | Verificación de disponibilidad del backend. |
| Health check Realtime | http://localhost:3000/ready | Verificación del gateway Socket.IO y Redis. |
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

## Flujo De Ejecución

La consola Vue autentica contra Laravel y solicita un token realtime de corta duración en `POST /api/v1/auth/realtime-token`. El gateway Realtime valida ese token, une el navegador a salas autorizadas de Socket.IO, consume eventos operativos desde Redis Streams y envía actualizaciones a los usuarios conectados.

Laravel publica eventos operativos en el stream compartido configurado por `MAINTOPS_EVENTS_STREAM`. El stack fija este stream como `maintops:events` tanto para Laravel como para el gateway Realtime.

Las rooms realtime se derivan únicamente de claims firmados por Laravel. Los usuarios siempre reciben rooms `user:<id>` y `role:<role>` permitidas; los usuarios con alcance de taller también reciben `workshop:<id>` y eventos de presencia de taller. Esto permite tanto notificaciones operativas por taller como futuras notificaciones administrativas, por ejemplo eventos de gestión de usuarios.

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
```

Luego abre `http://localhost:5173` e inicia sesión con la cuenta demo. El frontend debería conectarse al gateway realtime después del login.

Logs útiles:

```bash
docker compose logs -f laravel queue scheduler realtime frontend
```

## Detener O Reiniciar

Detener el ambiente sin borrar datos:

```bash
docker compose down
```

Borrar los datos de MySQL y Redis e iniciar de nuevo:

```bash
docker compose down -v
docker compose up -d --build
```

## Submódulos

Los proyectos de aplicación están incluidos como submódulos Git:

- `maintops-api-laravel`: https://github.com/cofran91/maintops-api-laravel
- `maintops-web-vue`: https://github.com/cofran91/maintops-web-vue
- `maintops-realtime-node`: https://github.com/cofran91/maintops-realtime-node

Cada repositorio de aplicación también puede abrirse de forma independiente para revisar su instalación, notas de arquitectura, comandos y documentación propia. Los cambios de aplicación deben commitearse primero en sus propios repositorios. Este stack guarda las revisiones exactas de los submódulos y la configuración local necesaria para ejecutarlos juntos.
