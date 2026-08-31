# Adaptación a cualquier tecnología

## Regla

La IA debe aprender primero el ecosistema del proyecto y después aplicar los principios. No debe reemplazar la convención existente por una preferencia personal ni convertir un ejemplo de Node, Angular, Python, Java, .NET, Go, Rust, PHP, móvil nativo o cloud en una obligación universal.

## Matriz de equivalencias

| Responsabilidad | Posibles implementaciones | Qué debe preservarse |
|---|---|---|
| Cliente/presentación | React, Angular, Vue, Svelte, Flutter, SwiftUI, Jetpack Compose, HTML | Accesibilidad, estado explícito, componentes cohesivos y pruebas de comportamiento |
| API/servidor | Fastify, Express, Nest, FastAPI, Django, Spring, ASP.NET, Gin, Axum | Handlers delgados, validación, autorización, contratos y errores consistentes |
| Dominio | Clases, módulos funcionales, aggregates, services, funciones puras | Invariantes, lenguaje del negocio y ausencia de infraestructura |
| Persistencia | PostgreSQL, MySQL, SQLite, MongoDB, DynamoDB, Redis, archivos | Consistencia, índices, migraciones, límites de transacción y recuperación |
| Mensajería | Kafka, RabbitMQ, SQS, Pub/Sub, NATS, colas propias | Idempotencia, orden, reintentos, dead-letter y observabilidad |
| Identidad | OAuth/OIDC, JWT, sesiones, passkeys, proveedor gestionado | Autenticación separada de autorización, mínimo privilegio y rotación |
| Testing | Jest, Vitest, Pytest, JUnit, NUnit, Go test, XCTest | Pruebas proporcionales al riesgo y centradas en comportamiento |
| CI/CD | GitHub Actions, GitLab CI, Jenkins, Buildkite, Azure DevOps | Checks reproducibles, artefactos, seguridad y rollback |
| Infraestructura | Docker, Kubernetes, serverless, VMs, PaaS, IaC | Configuración declarativa, secretos fuera del código y operación observable |

## Procedimiento de detección

Lee primero el manifiesto y lockfile, configuración del compilador, scripts, estructura de tests, Dockerfiles, workflows y documentación. Identifica el comando oficial para instalar, desarrollar, probar, verificar y construir. Si hay varias aplicaciones, determina sus dependencias y contratos antes de tocar código.

Después documenta en `PROJECT_CONTEXT.md` el stack real, las sustituciones permitidas, las herramientas oficiales y las restricciones. Usa la terminología del ecosistema del proyecto, pero conserva las responsabilidades arquitectónicas: dominio, aplicación, adaptadores, infraestructura, presentación, contratos, pruebas y operación.

## Adaptaciones por tipo de sistema

En un sistema web, prioriza contratos HTTP, accesibilidad, seguridad de sesión y rendimiento percibido. En móvil, añade ciclo de vida, permisos, almacenamiento local, conectividad intermitente y compatibilidad de plataforma. En APIs, prioriza versionado, límites, rate limiting, idempotencia y observabilidad. En datos, prioriza esquema, calidad, linaje, reproducibilidad y costos. En sistemas distribuidos, prioriza fallos parciales, consistencia, tiempo y contratos. En librerías, prioriza compatibilidad semántica, API mínima y documentación de migración.

## Lo que nunca debe cambiar

Aunque cambien las herramientas, la IA siempre debe preservar separación de responsabilidades, validación de entradas, autorización por recurso, gestión segura de secretos, errores explícitos, pruebas relevantes, logs sin datos sensibles, dependencias controladas, documentación de decisiones y capacidad de revertir cambios.
