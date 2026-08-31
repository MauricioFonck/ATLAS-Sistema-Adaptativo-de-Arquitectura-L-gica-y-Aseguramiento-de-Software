# Estructura base del monorepo

## Filosofía

La estructura mostrada en este documento es un **ejemplo de referencia para trabajar de forma organizada**, no una estructura rígida ni obligatoria. Representa una forma recomendada de separar aplicaciones, documentación, contexto de IA, automatización, pruebas e infraestructura dentro de un monorepo.

La IA debe adaptarse a cualquier estructura que ya exista y también puede proponer una estructura nueva cuando el proyecto se inicia desde cero. En un repositorio existente, primero debe descubrir y respetar su organización, convenciones, dependencias y restricciones; no debe mover carpetas solo para hacerlas coincidir con este ejemplo. En un proyecto nuevo, puede usar esta referencia como punto de partida y ajustarla al dominio, equipo, tecnologías, tamaño y forma de despliegue.

La estructura base de trabajo utiliza un monorepo con aplicaciones principales separadas, documentación técnica versionada, contexto vivo para la IA, automatización operativa y configuración de CI/CD. Esta organización es una **convención de trabajo**, no una lista de tecnologías obligatorias.

El ejemplo de referencia usa `backend-api`, `frontend-app`, `contexto`, `docs`, `scripts`, `.github`, `.githooks`, `.claude`, `.superpowers`, archivos Docker y documentación raíz. Si otro proyecto usa Python, Java, .NET, Go, Rust, PHP, React, Vue, Flutter, bases NoSQL o infraestructura serverless, se conservan las responsabilidades y se reemplaza la implementación tecnológica.

## Estructura recomendada

```text
nombre-proyecto/
├── .github/                    # CI/CD, políticas y plantillas de colaboración
│   ├── workflows/              # PR checks, CI, despliegue, backups y seguridad
│   ├── ISSUE_TEMPLATE/         # Plantillas de incidencias
│   └── PULL_REQUEST_TEMPLATE.md
│
├── .githooks/                  # Validaciones locales antes de commit/push
│
├── .claude/                    # Agentes, reglas y workflows de IA, si aplica
│   ├── agents/
│   └── workflows/
│
├── .superpowers/               # Automatización o extensiones de asistencia, si aplica
│
├── backend-api/                # API, servidor o aplicación de backend
│   ├── src/
│   │   ├── configuracion/      # Configuración y variables validadas
│   │   ├── infraestructura/    # DB, ORM, cache, auth, mensajería, observabilidad
│   │   └── modulos/            # Capacidades de negocio por módulo
│   ├── database/ o prisma/     # Esquemas, migraciones y seeds
│   ├── scripts/                # Operaciones específicas del backend
│   ├── tests/                  # unit, integration, contract, e2e o benchmark
│   └── README.md
│
├── frontend-app/               # Aplicación web, móvil o cliente principal
│   ├── src/
│   │   ├── app/ o features/    # Pantallas y funcionalidades
│   │   ├── core/                # Servicios transversales, auth, estado, offline
│   │   └── shared/              # Componentes y utilidades genéricas
│   ├── tests/                   # Componentes, integración y E2E
│   ├── native/ o android/      # Plataforma nativa, si aplica
│   └── README.md
│
├── contexto/                   # Memoria técnica viva del sistema
│   ├── backend/                # Módulos, contratos, DB y eventos
│   ├── frontend/               # Pantallas, rutas, componentes y flujos
│   └── arquitectura/           # ADRs, decisiones y modelos transversales
│
├── docs/                       # Diseño, especificaciones, investigación y runbooks
│   ├── specs/                  # Features grandes
│   ├── diagrams/               # Diagramas de arquitectura y flujos
│   └── operations/             # Despliegue, backup, incidentes y recuperación
│
├── scripts/                    # Mantenimiento, migración, backfill y DevOps
│
├── infra/                      # IaC, Docker, Kubernetes o configuración cloud, si aplica
│
├── docker-compose.yml          # Desarrollo local, si aplica
├── docker-compose.prod.yml     # Override o composición de producción, si aplica
├── CLAUDE.md o AI_RULES.md     # Reglas obligatorias para asistentes
├── PROJECT_CONTEXT.md          # Tecnologías, comandos y decisiones del proyecto
└── README.md                   # Entrada principal para desarrolladores
```

## Reglas de cada zona

| Zona | Regla de trabajo |
|---|---|
| `backend-api/` | La entrada técnica coordina; los módulos contienen negocio; infraestructura implementa detalles. |
| `frontend-app/` | La UI presenta y coordina; servicios o casos de uso manejan comportamiento; componentes compartidos no conocen features concretas. |
| `contexto/` | Se actualiza junto con código cuando cambian contratos, flujos, modelos o decisiones. |
| `docs/` | Las features grandes se especifican antes de programar y los runbooks describen cómo operar. |
| `scripts/` | Los scripts críticos son idempotentes, auditables, con dry-run y rollback documentado. |
| `.github/` | La calidad mínima debe ejecutarse automáticamente y bloquear entregas inseguras. |
| `.githooks/` | Previene errores locales, secretos y formato inconsistente; no sustituye CI. |
| `.claude/` y `.superpowers/` | Contienen comportamiento de asistencia; la IA debe leer las reglas aplicables antes de actuar. |
| Docker/IaC | Reproduce servicios y configuración sin incluir secretos. |
| Raíz | Contiene orientación, contexto, comandos y políticas, no lógica de negocio dispersa. |

## Política de adaptación estructural

| Situación | Comportamiento de la IA |
|---|---|
| Repositorio existente y organizado | Conservar la estructura; mapear responsabilidades y seguir convenciones locales. |
| Repositorio existente pero desordenado | No reorganizar todo automáticamente; proponer mejoras incrementales con riesgos, beneficios y migración. |
| Proyecto nuevo | Usar esta estructura como referencia y adaptarla al dominio y al stack elegido. |
| Estructura impuesta por framework o plataforma | Respetar la convención tecnológica y aplicar los principios dentro de sus límites. |
| Monorepo con varios clientes o servicios | Mantener aplicaciones separadas, contratos explícitos, ownership y documentación compartida. |
| Repositorio que no es monorepo | No forzar un monorepo; documentar la excepción y conservar los mismos principios de calidad. |

La IA debe distinguir siempre entre **estructura recomendada**, **estructura encontrada** y **estructura propuesta**. Antes de crear o mover carpetas debe explicar el motivo. La organización es correcta cuando mejora descubribilidad, ownership, límites, pruebas, automatización, despliegue o mantenimiento; no simplemente cuando se parece al árbol de este documento.

## Adaptación de nombres

El nombre `backend-api` representa cualquier servidor, API, servicio principal o aplicación de backend. `frontend-app` representa cualquier cliente web, móvil, escritorio o interfaz. Si hay múltiples backends o clientes, añade subdirectorios manteniendo el mismo criterio. Si no existe una capa concreta, no la inventes: documenta su ausencia.

## Regla de evolución

No reorganices el monorepo solo por estética. Una carpeta cambia cuando mejora ownership, límites, descubribilidad, aislamiento, build, despliegue o capacidad de prueba. Toda reorganización debe preservar historial cuando sea posible, actualizar referencias y ejecutarse con validación completa.
