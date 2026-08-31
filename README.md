# ATLAS — Sistema Adaptativo de Arquitectura, Lógica y Aseguramiento de Software

# Estándares personales de ingeniería de software para trabajar con IA

**Versión:** 1.0.0  
**Propósito:** servir como contrato técnico reutilizable para que una IA comprenda, cuestione y aplique una forma de trabajo profesional en cualquier proyecto.

## Cómo usar esta carpeta

Copia esta carpeta en la raíz del proyecto o incorpora sus archivos como contexto de trabajo. El archivo `AI_INSTRUCTIONS.md` es la entrada principal para la IA. Antes de escribir código, la IA debe leerlo junto con el contexto específico del proyecto, identificar restricciones y declarar los supuestos que utilizará.

Este sistema no pretende imponer una tecnología única. Define decisiones por defecto, criterios para elegir alternativas y límites para evitar la sobreingeniería. La mejor arquitectura no es la más sofisticada: es la que satisface los requisitos de negocio y calidad con el menor acoplamiento y costo operativo razonable.

La estructura del monorepo documentada aquí es una **referencia de organización**, inspirada en el ejemplo del usuario. Debe adaptarse a cualquier estructura ya creada y a cualquier proyecto nuevo. En repositorios existentes, la IA debe descubrir, respetar y documentar la organización local; en proyectos nuevos, puede proponer esta estructura y ajustarla al dominio, stack, equipo y despliegue. No debe mover carpetas ni imponer un monorepo sin una razón técnica explícita.

## Decisión por defecto

Para la mayoría de productos nuevos, comenzar con un **monolito modular**, organizado por capacidades de negocio y con límites explícitos. Dentro de cada módulo se recomienda una combinación pragmática de **arquitectura hexagonal/clean**, principios de **DDD táctico** cuando exista complejidad de negocio y separación clara entre dominio, aplicación, adaptadores e infraestructura.

Evolucionar hacia microservicios únicamente cuando exista una razón verificable: equipos independientes, escalado desigual, límites de dominio estables, despliegue autónomo, aislamiento de fallos o requisitos regulatorios. Los microservicios añaden redes, observabilidad, consistencia distribuida, versionado y operación; no deben ser el punto de partida automático. Esta cautela coincide con la distinción entre componentes enlazados en memoria y servicios desplegables independientemente descrita por Fowler [1], y con los criterios comparativos de estilos arquitectónicos de Microsoft [2].

## Mapa de contenidos

| Archivo | Contenido | Cuándo consultarlo |
|---|---|---|
| `AI_INSTRUCTIONS.md` | Contrato de comportamiento para la IA | Siempre, antes de modificar el proyecto |
| `00-MONOREPO_STRUCTURE.md` | Estructura base del monorepo y responsabilidades por carpeta | Al iniciar cualquier proyecto |
| `PROJECT_CONTEXT_TEMPLATE.md` | Ficha de contexto del monorepo y reglas críticas | Copiar a la raíz como `PROJECT_CONTEXT.md` |
| `01-arquitectura/ARCHITECTURE.md` | Arquitecturas, capas, módulos y decisiones | Al iniciar o cambiar estructura |
| `01-arquitectura/SAAS_MULTITENANCY_CACHE.md` | Tenancy, shards, bases compartidas/dedicadas, migraciones y caché | En cualquier SaaS o sistema multi-base |
| `02-diseno-y-codigo/CODE_STANDARDS.md` | SOLID, legibilidad, patrones y errores comunes | Al diseñar o escribir código |
| `02-diseno-y-codigo/ORM_STANDARDS.md` | Reglas universales para ORMs, consultas, migraciones y transacciones | Al trabajar con persistencia |
| `03-calidad-y-seguridad/QUALITY_SECURITY.md` | Pruebas, seguridad, rendimiento y observabilidad | Antes de entregar o desplegar |
| `03-calidad-y-seguridad/QA_STRATEGY.md` | QA senior, pruebas positivas/negativas, automatización y métricas | Desde requisitos hasta producción |
| `04-proceso/DELIVERY_WORKFLOW.md` | Flujo profesional de trabajo y Definition of Done | En cada tarea o feature |
| `05-plantillas/ADR_TEMPLATE.md` | Plantilla para decisiones arquitectónicas | Cuando una decisión tenga impacto futuro |
| `05-plantillas/FEATURE_TEMPLATE.md` | Plantilla para especificar funcionalidades | Antes de implementar una feature |
| `05-plantillas/TEST_CASE_TEMPLATE.md` | Casos de prueba positivos, negativos y de regresión | Al diseñar o reportar pruebas |

## Jerarquía de decisiones

La IA debe priorizar, en este orden: requisitos explícitos del usuario, restricciones legales y de seguridad, contratos públicos existentes, requisitos no funcionales medibles, coherencia con la arquitectura actual, simplicidad y preferencias de estilo. Si dos reglas entran en conflicto, debe señalarlo y pedir confirmación cuando el impacto sea alto.

## Principios no negociables

En sistemas SaaS, el aislamiento de tenant, la residencia de datos, las cuotas, la capacidad, las migraciones entre bases y la caché tenant-aware son preocupaciones arquitectónicas de primer nivel. La calidad se valida con pruebas positivas y negativas, priorizadas por riesgo, y no solo con recorridos felices. Deben diseñarse y probarse como parte del sistema, no agregarse como detalles de infraestructura al final.

El código debe ser comprensible, testeable, seguro, observable y reversible. Las decisiones deben poder explicarse. La lógica de negocio no debe depender de frameworks, bases de datos, proveedores cloud ni mecanismos de transporte. Ningún patrón se introduce por moda: cada abstracción debe eliminar una dependencia, proteger una regla de negocio o facilitar una evolución probable.

## Referencias

[1]: https://martinfowler.com/articles/microservices.html "Martin Fowler — Microservices"
[2]: https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/ "Microsoft Learn — Architecture styles"
[3]: https://owasp.org/www-project-secure-coding-practices-quick-reference-guide/ "OWASP — Secure Coding Practices Quick Reference Guide"
[4]: https://google.github.io/eng-practices/review/ "Google Engineering Practices — Code Review"
[5]: https://www.iso.org/obp/ui/#iso:std:iso-iec:25010:en "ISO/IEC 25010:2023 — Software product quality model"

> **Regla de oro:** diseñar para el cambio que conocemos, no para todos los cambios imaginables.

## Registro de cambios

| Versión | Fecha | Cambio |
|---|---|---|
| 1.0.0 | 2026-08-31 | Primera versión reutilizable. |

## Personalización recomendada

Sustituye las preferencias generales por decisiones del proyecto en `PROJECT_CONTEXT.md`, usando `PROJECT_CONTEXT_TEMPLATE.md` como base. Completa los comandos reales, contratos, workflows y políticas de despliegue. Conserva estos estándares como la base estable y modifica únicamente lo que realmente sea una decisión consciente. La IA debe distinguir entre una regla global y una excepción documentada del proyecto.

## Licencia de uso

Puedes copiar, adaptar y mantener esta carpeta libremente para tus proyectos personales o profesionales. Revisa las licencias de las fuentes enlazadas si redistribuyes contenido derivado de ellas.
