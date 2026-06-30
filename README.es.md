# MaintOps Developer Stack

Documentación en inglés: [README.md](README.md).

MaintOps es un proyecto multi-servicio de portafolio para operaciones de mantenimiento vehicular. Este repositorio es el ambiente local replicable para ejecutar el demo completo: integra la API Laravel, la consola web Vue, el gateway realtime Node, el servicio de analítica FastAPI, MySQL, PostgreSQL, Redis y Mailpit con un solo archivo Docker Compose.

Este repositorio no contiene la lógica de aplicación. Las aplicaciones viven en submódulos Git y pueden revisarse de forma independiente. El stack contiene la orquestación, los valores locales por defecto, el cableado entre servicios, las revisiones exactas de los submódulos y la guía operativa para ejecutar el sistema completo.

## Propósito De Portafolio

El stack está diseñado para mostrar cómo los servicios individuales trabajan juntos como un solo producto:

- Laravel contiene autenticación, autorización, datos transaccionales, flujos de dominio, auditoría, automatización programada, respuestas API localizadas, eventos operativos y correos.
- Vue contiene la experiencia de navegador: consola autenticada bilingüe, navegación por roles, flujos CRUD, notificaciones realtime y vistas de analítica.
- Node contiene la entrega realtime: autenticación Socket.IO con tokens firmados, rooms autorizadas, consumo de Redis Streams, presencia, deduplicación y dead-letter.
- FastAPI contiene analítica: modelo de lectura PostgreSQL, initial sync desde Laravel, proyección desde Redis Streams, métricas observadas, pronósticos, alertas de riesgo, recomendaciones y códigos estructurados que la consola bilingüe puede traducir.

Cada servicio puede estudiarse por separado, pero la ruta de revisión esperada es el stack completo porque varias capacidades solo tienen sentido cuando los servicios están conectados.

## Qué Simula El Producto

MaintOps simula la operación diaria de una empresa de mantenimiento vehicular.

El producto no es solo un demo CRUD. Modela cómo un asesor recibe un vehículo de cliente, cómo el trabajo de mantenimiento se organiza en una orden, cómo los trabajos se aprueban y agendan, cómo los técnicos ejecutan trabajo asignado y cómo los administradores supervisan la operación mediante dashboard, notificaciones realtime, correos y analítica.

Conceptos principales del negocio:

| Concepto | Significado En El Demo |
| --- | --- |
| Owner / propietario | Cliente dueño de uno o más vehículos. Es un registro del dominio, no un usuario interactivo de la consola. |
| Vehicle / vehículo | Vehículo que recibe mantenimiento. Pertenece a un owner y guarda datos como placa y odómetro. |
| Workshop / taller | Ubicación física u operativa donde se realiza el mantenimiento. |
| Technician / técnico | Usuario que ejecuta trabajo de mantenimiento asignado. |
| Vehicle system / sistema del vehículo | Área de especialidad como motor, frenos, eléctrico, refrigeración, llantas o suspensión. Talleres y técnicos pueden limitarse por sistemas soportados. |
| Maintenance task / tarea | Trabajo que puede realizarse, como revisar frenos, cambiar aceite o diagnosticar un problema eléctrico. |
| Maintenance plan / plan | Conjunto de tareas recurrentes recomendadas por tiempo o kilometraje. |
| Maintenance order / orden | Solicitud principal de trabajo para un vehículo. Agrupa el mantenimiento que debe revisarse, aprobarse, agendarse y ejecutarse. |
| Order item / ítem de orden | Trabajo individual dentro de una orden. Puede aprobarse, rechazarse, agendarse, iniciarse, completarse o cancelarse. |
| Approval / aprobación | Decisión de cara al cliente sobre qué ítems de la orden se realizarán. |
| Scheduling / agenda | Asignación del trabajo aprobado a un taller, técnico y horario planeado. |

El flujo funcional esperado es:

1. Un administrador prepara datos operativos: usuarios, roles, talleres, técnicos, sistemas del vehículo, tareas y planes de mantenimiento.
2. Un asesor registra o revisa un owner y un vehículo.
3. Un asesor crea una orden de mantenimiento para ese vehículo.
4. El sistema puede agregar ítems recomendados desde planes vencidos y problemas activos específicos del vehículo.
5. El owner aprueba o rechaza cada ítem propuesto. Los aceptados avanzan; los rechazados quedan como historial de decisión.
6. Los ítems aprobados se agendan a un taller y técnico según capacidad, horario laboral, disponibilidad y duración esperada.
7. Un técnico inicia sesión y ve solo trabajo asignado.
8. El técnico inicia y completa los ítems asignados.
9. La orden puede completarse y entregarse.
10. Notificaciones realtime, correo al owner, dashboard, auditoría y analítica se actualizan desde los mismos cambios operativos.

No necesitas crear todo este flujo desde cero en la primera revisión. Los seeders incluyen datos demo en los principales estados del ciclo de vida para que puedas revisar órdenes, usuarios, talleres, tareas, notificaciones y analítica de inmediato.

## Servicios Incluidos

| Servicio | Proyecto | Responsabilidad |
| --- | --- | --- |
| API Laravel | `maintops-api-laravel` | Backend transaccional, auth, policies, state machines, scheduling, auditoría, correos y contratos de integración. |
| Consola Vue | `maintops-web-vue` | Consola web para operaciones, registros de mantenimiento, actividad realtime y analítica. |
| Gateway Realtime | `maintops-realtime-node` | Gateway Socket.IO que consume eventos operativos de Laravel desde Redis Streams. |
| API Analytics | `maintops-analytics-fastapi` | Servicio de analítica read-only con su propio modelo de lectura PostgreSQL. |
| MySQL | Imagen Docker | Base de datos transaccional de Laravel. |
| PostgreSQL | Imagen Docker | Base de datos del modelo de lectura de Analytics. |
| Redis | Imagen Docker | Transporte compartido con Redis Streams y soporte realtime. |
| Mailpit | Imagen Docker | Bandeja local para recuperación de contraseña y correos operativos al owner. |

## Requisitos

- Docker Engine o Docker Desktop con Docker Compose v2.
- Git.
- Acceso de red a GitHub para clonar este repositorio y sus submódulos públicos.

## Clonar

```bash
git clone --recurse-submodules https://github.com/cofran91/maintops-stack.git
cd maintops-stack
```

Si el repositorio fue clonado sin submódulos:

```bash
git submodule update --init --recursive
```

Los submódulos usan URLs HTTPS públicas para que el stack pueda clonarse sin configurar SSH. Los desarrolladores que prefieran SSH pueden configurarlo localmente con una regla `insteadOf` de Git.

## Iniciar El Demo Completo

No necesitas crear un archivo `.env` para desarrollo local. `compose.yaml` ya incluye valores por defecto prácticos. Copia `.env.example` a `.env` solo cuando necesites cambiar puertos, credenciales locales, orígenes CORS, nombres de colas o configuración compartida de tokens de servicio.

```bash
docker compose config
docker compose up -d --build
docker compose ps
```

La primera ejecución construye las imágenes, instala dependencias dentro de los contenedores, espera MySQL, PostgreSQL y Redis, ejecuta migraciones y seeders de Laravel, ejecuta migraciones de Analytics, importa el modelo de lectura inicial desde Laravel, inicia workers y scheduler de Laravel, inicia Realtime, inicia el worker de Analytics y sirve la consola Vue.

## URLs De Servicios

| Superficie | URL | Uso |
| --- | --- | --- |
| Frontend Vue | http://localhost:5173 | Consola principal de MaintOps. |
| API Laravel | http://localhost:8000/api/v1 | Raíz de la API transaccional. |
| Health check Laravel | http://localhost:8000/up | Verificación de disponibilidad del backend. |
| Readiness Realtime | http://localhost:3000/ready | Verificación del gateway Socket.IO y Redis. |
| API Analytics | http://localhost:8001 | Analítica operacional read-only. |
| Health check Analytics | http://localhost:8001/health | Verificación de vida de FastAPI. |
| Readiness Analytics | http://localhost:8001/ready | Verificación de FastAPI y PostgreSQL. |
| Mailpit | http://localhost:8025 | Bandeja local para correos demo. |

MySQL se publica solo en localhost:

```text
Host: 127.0.0.1
Port: 3307
Database: maintops
User: maintops
Password: maintops-local-password
```

PostgreSQL de Analytics se publica solo en localhost:

```text
Host: 127.0.0.1
Port: 5433
Database: maintops_analytics
User: maintops_analytics
Password: maintops-analytics-password
```

Redis queda disponible solo dentro de la red de Compose como `redis:6379`.

## Cuenta Demo

Los seeders base crean usuarios demo para los roles principales. Todas las cuentas demo usan:

```text
password: password
```

Cuentas recomendadas:

| Rol | Email | Qué Revisar |
| --- | --- | --- |
| Super admin | `admin@maint.test` | Revisión local completa, herramientas internas, documentación API, auditoría y administración sin restricciones. |
| Admin | `admin.demo@maint.test` | Dashboard, catálogos, usuarios, talleres, órdenes, auditoría y Analytics. |
| Jefe de taller | `manager.north@maint.test` | Operación limitada al taller asignado y visibilidad restringida. |
| Asesor | `advisor.north@maint.test` | Flujos de owners, vehículos y órdenes de mantenimiento. |
| Técnico | `technician.engine@maint.test` | Trabajo asignado y transiciones ejecutables de ítems. |

## Ruta Sugerida De Revisión

Para una guía visual paso a paso de la demo pública, abre:

```text
https://juandavidfranco.com/demo
```

Usa la misma lógica de revisión en local con estas URLs del stack:

- Consola Vue: `http://localhost:5173`
- Bandeja Mailpit: `http://localhost:8025`
- Documentación API Laravel: `http://localhost:8000/docs`
- Documentación API Analytics: `http://localhost:8001/docs`

Para revisión local, empieza con `admin@maint.test` porque es la cuenta `super_admin` y permite inspeccionar el ambiente completo, herramientas internas, auditoría y documentación API. Luego cambia a `admin.demo@maint.test`, `manager.north@maint.test`, `advisor.north@maint.test` y `technician.engine@maint.test` para validar comportamiento limitado por rol.

Usa solo datos de muestra. Los ambientes demo no deberían recibir datos reales de clientes, vehículos, direcciones, contraseñas, teléfonos, documentos o correos personales. Mailpit queda expuesto intencionalmente en el stack local para que los revisores puedan inspeccionar correos demo generados.

## Flujo De Ejecución

La consola Vue autentica contra Laravel. Laravel sigue siendo la fuente de verdad de identidad y autorización.

Para actualizaciones realtime, el navegador solicita a Laravel un token de servicio de corta duración con `audience: "realtime"` y lo usa durante el handshake de Socket.IO. El gateway Node valida el token firmado, consume eventos operativos de Laravel desde Redis Streams y entrega solo eventos autorizados a usuarios conectados.

Para analítica, el navegador solicita a Laravel un token de servicio de corta duración con `audience: "analytics"` y lo usa al llamar a FastAPI. Analytics valida el mismo contrato de token compartido y lee desde su propio modelo PostgreSQL en vez de leer directamente la base MySQL de Laravel.

Laravel publica eventos operativos en el Redis Stream compartido configurado por `MAINTOPS_EVENTS_STREAM`. Realtime y Analytics consumen el mismo stream con consumer groups separados, así que la entrega live de UI y la proyección analítica pueden evolucionar de forma independiente.

Si sobrescribes `SERVICE_TOKEN_SECRET`, usa el mismo valor para Laravel, Realtime y Analytics; de lo contrario las conexiones realtime y las peticiones a Analytics serán rechazadas.

## Colas Y Correos

El stack ejecuta tres workers de cola de Laravel:

- `queue` procesa jobs generales de la aplicación.
- `queue-events` publica eventos operativos en Redis Streams.
- `queue-mail` envía correos encolados a través de Mailpit.

Mailpit captura correos de aplicación en vez de enviarlos a internet. Se usa para recuperación de contraseña y correos operativos al owner.

La recuperación de contraseña es local por defecto:

1. Abre `http://localhost:5173`.
2. Usa el enlace "Forgot your password?" en la pantalla de login.
3. Revisa el correo de recuperación en Mailpit en `http://localhost:8025`.
4. Abre el enlace de recuperación. Laravel genera los enlaces usando `FRONTEND_PASSWORD_RESET_URL`, cuyo valor por defecto es `http://localhost:5173/reset-password`.

## Verificación

```bash
curl http://localhost:8000/up
curl http://localhost:3000/ready
curl http://localhost:8001/health
curl http://localhost:8001/ready
```

Logs útiles de servicios permanentes:

```bash
docker compose logs -f laravel queue queue-events queue-mail scheduler realtime analytics-api analytics-worker frontend
```

Si el arranque se detiene antes de que los servicios permanentes estén saludables, revisa los servicios de inicialización:

```bash
docker compose logs laravel-init analytics-migrations analytics-initial-sync
```

## Detener O Reiniciar

Detener el ambiente sin borrar datos:

```bash
docker compose down
```

Borrar datos de MySQL, PostgreSQL y Redis e iniciar de nuevo:

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

Cada repositorio de aplicación documenta sus propias decisiones tecnológicas, estructura de carpetas, comandos, pruebas y límites de integración. Los cambios de aplicación deben commitearse primero en sus propios repositorios. Este stack guarda las revisiones exactas de los submódulos y la configuración local necesaria para ejecutarlos juntos.

## Límite De Despliegue

La configuración específica de reverse proxy para despliegue queda intencionalmente fuera de este repositorio. Este stack está enfocado en el ambiente local replicable usado para revisar el demo de portafolio.
