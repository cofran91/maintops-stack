# MaintOps Developer Stack

Ambiente local con Docker para MaintOps, un proyecto de portafolio enfocado en la gestión de operaciones de mantenimiento. Este repositorio integra la API Laravel, la consola Vue, MySQL, Redis y Mailpit con un solo archivo Compose para que el entorno pueda clonarse y ejecutarse de forma consistente en otra máquina.

Este repositorio contiene únicamente la orquestación del ambiente: Docker Compose, valores locales por defecto y documentación operativa. El código fuente de las aplicaciones vive en submódulos Git.

## Servicios Incluidos

- API Laravel desde `maintops-api-laravel`.
- Consola Vue/Vite desde `maintops-web-vue`.
- MySQL 8.4 con un volumen local de Docker.
- Redis 7 con persistencia append-only.
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

## Iniciar El Ambiente

No necesitas crear un archivo `.env` para desarrollo local. `compose.yaml` ya incluye valores por defecto prácticos. Copia `.env.example` a `.env` solo cuando necesites cambiar puertos o credenciales locales.

```bash
docker compose config
docker compose up -d --build
docker compose ps
```

La primera ejecución construye las imágenes, instala las dependencias de aplicación dentro de los contenedores, espera MySQL y Redis, ejecuta las migraciones de Laravel y carga los seeders base.

## Servicios

| Servicio | URL | Propósito |
| --- | --- | --- |
| Frontend Vue | http://localhost:5173 | Aplicación web MaintOps Console. |
| API Laravel | http://localhost:8000/api/v1 | API transaccional. |
| Health check Laravel | http://localhost:8000/up | Verificación de disponibilidad del backend. |
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
```

Luego abre `http://localhost:5173` e inicia sesión con la cuenta demo.

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

Cada repositorio de aplicación también puede abrirse de forma independiente para revisar su instalación, notas de arquitectura, comandos y documentación propia. Los cambios de aplicación deben commitearse primero en sus propios repositorios. Este stack guarda las revisiones exactas de los submódulos y la configuración local necesaria para ejecutarlos juntos.
