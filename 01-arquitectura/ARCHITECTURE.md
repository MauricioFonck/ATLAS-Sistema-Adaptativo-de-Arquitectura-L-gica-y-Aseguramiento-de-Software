# Arquitectura universal de software y monorepos

## Principio principal

La arquitectura debe proteger las reglas de negocio, facilitar el cambio y mantener bajo el costo operativo. No existe una arquitectura superior para todos los sistemas. La decisión correcta depende del dominio, escala, equipo, riesgo, latencia, disponibilidad, regulación y presupuesto.

La forma predeterminada recomendada para un producto nuevo es un **monorepo con monolito modular o aplicaciones claramente separadas**, límites de dominio explícitos, contratos versionados, documentación viva y automatización desde la raíz. El monorepo es una estrategia de organización y colaboración; no obliga a ejecutar todo como un único proceso ni impide desplegar aplicaciones de forma independiente.

## Estructura de referencia

La estructura base debe seguir directamente este patrón de monorepo:

```text
monorepo/
  .github/                  # CI/CD, seguridad, backups y colaboración
  .githooks/                # validaciones locales
  .claude/                 # agentes, reglas y workflows de IA, si aplica
  .superpowers/             # extensiones o automatización de IA, si aplica
  backend-api/              # servidor, API o aplicación backend
    src/
      configuracion/        # configuración validada
      infraestructura/      # DB, ORM, cache, auth, eventos y observabilidad
      modulos/              # capacidades de negocio
    prisma/ o database/     # esquemas y migraciones
    scripts/                # tareas de backend
    tests/                  # unit, integración, contratos y benchmarks
  frontend-app/             # aplicación web, móvil o cliente principal
    src/app/
      pages/                # pantallas y features
      core/                 # servicios transversales, auth y offline
      shared/               # componentes reutilizables
    tests/
    android/ o native/      # plataforma nativa, si aplica
  contexto/                 # memoria técnica viva
    backend/
    frontend/
    arquitectura/
  docs/                     # specs, diagramas, investigación y operación
    specs/
    diagrams/
    operations/
  scripts/                  # mantenimiento, migraciones y DevOps
  infra/                    # IaC, contenedores o cloud, si aplica
  docker-compose.yml        # desarrollo local, si aplica
  docker-compose.prod.yml   # producción, si aplica
  CLAUDE.md o AI_RULES.md   # reglas de asistentes
  PROJECT_CONTEXT.md        # mapa tecnológico real
  README.md                 # entrada principal
```

Este patrón, inspirado directamente en el ejemplo del usuario, es la convención estructural base. Los nombres `backend-api`, `frontend-app` y `contexto` pueden sustituirse solo cuando el proyecto tenga una razón clara; las responsabilidades deben mantenerse. Las tecnologías dentro de cada zona son variables y se detectan desde los archivos reales del proyecto.

## Estilos y criterios de selección

| Contexto | Opción preferida | Cuándo cambiar |
|---|---|---|
| CRUD pequeño o prototipo | Aplicación simple por capas | Cuando aumente la complejidad de negocio o equipos |
| Dominio mediano o complejo | Monolito modular + Clean/Hexagonal selectiva | Cuando un módulo necesite escalar o desplegarse de forma autónoma |
| Trabajo pesado o diferido | Aplicación web/API + cola + workers | Cuando el procesamiento bloquee solicitudes o necesite escalado separado |
| Dominios y equipos autónomos | Servicios separados por bounded context | Cuando existan límites estables, ownership y capacidad operativa |
| Muchos consumidores reactivos | Event-driven | Cuando el desacoplamiento y la asincronía aporten valor verificable |
| Datos analíticos masivos | Pipeline y almacén analítico especializado | Cuando las necesidades de consulta difieran del OLTP |

Microsoft describe varios estilos arquitectónicos y sus beneficios, desafíos y escenarios de uso [1]. Fowler destaca que los servicios permiten despliegue independiente, pero introducen comunicación remota, contratos, consistencia distribuida y complejidad operativa [2]. Por eso, no se deben elegir microservicios por defecto.

## Clean Architecture y Hexagonal

La regla esencial es que las dependencias apunten hacia las políticas de negocio:

```text
Entradas: UI, HTTP, CLI, eventos, jobs
                 ↓
Adaptadores de entrada / casos de uso
                 ↓
Dominio: entidades, value objects, reglas, eventos
                 ↑
Puertos: interfaces necesarias por las políticas
                 ↑
Adaptadores de salida: DB, ORM, APIs, colas, archivos, cloud
```

El dominio no conoce frameworks, transporte, ORM, base de datos ni proveedor. La aplicación coordina casos de uso y límites transaccionales. Los adaptadores convierten formatos externos. La composición de dependencias se concentra en el arranque de cada aplicación.

No es necesario imponer todas las capas a cada pantalla o endpoint. Aplica Clean Architecture donde protege complejidad, testabilidad o evolución; para operaciones triviales, mantén una solución simple.

## Organización por capacidades

Cuando el sistema tenga reglas de negocio, organiza módulos por capacidad o bounded context, no únicamente por tipo técnico. Cada módulo debe controlar sus invariantes, servicios, puertos y pruebas. Las librerías compartidas deben contener abstracciones realmente transversales y no convertirse en un cajón de modelos de negocio mezclados.

La frontera de aplicaciones debe definirse mediante contratos. No compartas entidades internas, modelos ORM o estado mutable entre frontend, backend y workers. Comparte contratos públicos, esquemas validados o librerías deliberadamente versionadas.

## DDD táctico

Usa entidades para objetos con identidad, value objects para valores con invariantes, agregados para límites transaccionales, servicios de dominio para operaciones que no pertenecen a una entidad, repositorios como puertos y eventos de dominio para hechos relevantes. Un agregado debe ser pequeño y proteger sus invariantes.

Los bounded contexts pueden convivir dentro del mismo monolito. No conviertas automáticamente cada contexto en un microservicio. La distribución se decide después de validar límites, ownership, SLO, contratos, datos, observabilidad, costos y plan de fallos.

## Patrones de integración

| Problema | Solución habitual | Condición |
|---|---|---|
| Proveedor externo | Adapter o Anti-Corruption Layer | Aislar su modelo y fallos |
| Variantes de algoritmo | Strategy | Debe existir una variación real |
| Creación con invariantes | Factory | No crear factorías ceremoniales |
| Evento antes de publicar | Outbox | Consumidor idempotente y reintentos |
| Llamada remota inestable | Timeout, retry acotado y circuit breaker | Reintentos solo para errores transitorios |
| Lecturas muy distintas | CQRS | Complejidad justificada por escala o modelo |
| Compatibilidad entre equipos | Contract testing | No sustituye integración real |

## Multi-tenancy, datos y offline

Si existe multi-tenancy, el contexto del tenant debe atravesar autenticación, autorización, persistencia, caché, mensajes, jobs, auditoría y scripts. El enrutamiento hacia particiones, bases o shards debe estar encapsulado, validado y probado contra accesos cruzados.

Si existe operación offline, cada mutación debe definir identificador estable, idempotencia, versión o cursor, reintentos, estados, conflicto y recuperación. La interfaz debe distinguir pendiente, sincronizada, rechazada y conflicto. Estas reglas son universales y se adaptan a cualquier tecnología de almacenamiento o sincronización.

## Decisiones y documentación

Los cambios importantes deben registrar contexto, alternativas, consecuencias, evidencia y reversión en un ADR. Los cambios coordinados entre aplicaciones deben actualizar especificaciones, contratos, consumidores, pruebas y documentación viva. El archivo `PROJECT_CONTEXT.md` debe mapear la estructura y tecnologías reales sin convertirlas en reglas universales.

## Referencias

[1]: https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/ "Microsoft Learn — Architecture styles"
[2]: https://martinfowler.com/articles/microservices.html "Martin Fowler — Microservices"
