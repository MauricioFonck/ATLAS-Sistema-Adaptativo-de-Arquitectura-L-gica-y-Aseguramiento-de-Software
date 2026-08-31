# Contexto del proyecto — Plantilla universal de monorepo

> Copia este archivo a la raíz del repositorio como `PROJECT_CONTEXT.md`. Completa solo lo específico de cada proyecto. Esta ficha permite que la IA aplique la misma forma de trabajar aunque cambien las tecnologías.

## Identidad del sistema

- Nombre: `[COMPLETAR]`
- Propósito de negocio: `[COMPLETAR]`
- Tipo de producto: web | móvil | escritorio | API | plataforma | datos | otro
- Entornos: local | test | staging | producción
- Equipo y propietarios: `[COMPLETAR]`

## Mapa real del monorepo

| Ruta real | Papel arquitectónico | Tecnologías actuales | Propietario |
|---|---|---|---|
| `[ruta]` | Aplicación frontend o cliente | `[stack]` | `[equipo]` |
| `[ruta]` | API o backend | `[stack]` | `[equipo]` |
| `[ruta]` | Worker, jobs o procesos asíncronos | `[stack]` | `[equipo]` |
| `[ruta]` | Librería compartida | `[stack]` | `[equipo]` |
| `[ruta]` | Documentación y contexto vivo | `[formato]` | `[equipo]` |
| `[ruta]` | Scripts y operaciones | `[lenguaje]` | `[equipo]` |
| `[ruta]` | CI/CD, hooks o infraestructura | `[herramientas]` | `[equipo]` |

## Inventario tecnológico

| Capacidad | Tecnología o proveedor | Sustitutos permitidos | Decisiones y límites |
|---|---|---|---|
| Lenguajes y runtimes | `[completar]` | `[completar]` | |
| Frontend/web | `[completar]` | `[completar]` | |
| Móvil/escritorio | `[completar]` | `[completar]` | |
| Backend/API | `[completar]` | `[completar]` | |
| Persistencia | `[completar]` | `[completar]` | |
| ORM o capa de acceso | `[completar]` | `[completar]` | Versión, driver, migraciones, pool y límites |
| Caché | `[completar]` | `[completar]` | |
| Mensajería/eventos | `[completar]` | `[completar]` | |
| Identidad y autorización | `[completar]` | `[completar]` | |
| Observabilidad | `[completar]` | `[completar]` | |
| Contenedores/cloud | `[completar]` | `[completar]` | |
| CI/CD | `[completar]` | `[completar]` | |

## Límites y arquitectura

- Arquitectura actual: `[monolito modular / servicios / serverless / híbrida / otra]`.
- Organización de módulos: `[por dominio / por feature / por capa / otra]`.
- Fuente de verdad de contratos: `[completar]`.
- Fuente de verdad de esquemas y migraciones: `[completar]`.
- ORM o estrategia de acceso a datos: `[completar]`.
- Contexto de tenant, si existe: `[completar]`.
- Consistencia y transacciones: `[completar]`.
- Modo offline o sincronización, si existe: `[completar]`.
- Decisiones arquitectónicas: `[ruta de ADRs]`.

## Comandos oficiales

| Acción | Comando |
|---|---|
| Instalar | `[completar]` |
| Ejecutar entorno local | `[completar]` |
| Test unitario | `[completar]` |
| Test integración | `[completar]` |
| Test end-to-end | `[completar]` |
| Lint y formato | `[completar]` |
| Type-check o análisis estático | `[completar]` |
| Build | `[completar]` |
| Migración | `[completar]` |
| Seguridad | `[completar]` |
| Despliegue y rollback | `[completar]` |

## Reglas de cambio

Un cambio que afecte más de una aplicación debe actualizar el contrato, consumidores, pruebas y documentación. Un cambio de base de datos debe incluir migración compatible, plan de recuperación y validación de datos. Un cambio de autenticación, permisos, datos personales, dinero, auditoría, sincronización o infraestructura requiere revisión de seguridad y operación.

## Documentación viva

- Índice general: `[ruta]`
- Contexto de backend: `[ruta]`
- Contexto de frontend: `[ruta]`
- Especificaciones: `[ruta]`
- ADRs: `[ruta]`
- Diagramas: `[ruta]`
- Runbooks: `[ruta]`
- Reglas de IA: `[ruta]`

## Convenciones del equipo

- Estilo de nombres: `[completar]`
- Formato de commits: `[completar]`
- Estrategia de ramas: `[completar]`
- Política de pull requests: `[completar]`
- Política de releases: `[completar]`
- Requisitos de revisión: `[completar]`

## Supuestos activos y deuda

Registra aquí supuestos que la IA no debe convertir en hechos y deuda técnica que debe tenerse en cuenta. Cada elemento debe indicar impacto, propietario y condición de revisión.
