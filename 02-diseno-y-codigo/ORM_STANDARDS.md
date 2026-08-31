# Estándares universales para ORMs

## Principio

El ORM es un **adaptador de persistencia**, no el dominio de la aplicación. La IA debe adaptarse al ORM existente —o al que el proyecto elija— sin convertir automáticamente sus modelos, entidades, decorators, migrations o query builders en reglas de negocio.

Este estándar aplica a ORMs y herramientas equivalentes de cualquier ecosistema: Prisma, TypeORM, Sequelize, MikroORM, Drizzle, Hibernate/JPA, Entity Framework, SQLAlchemy, Django ORM, Active Record, Eloquent, GORM, Diesel, Exposed y otros. Si el proyecto usa SQL directo, repositorios manuales o una base NoSQL, se conservan los mismos objetivos y se adapta la implementación.

## Detección obligatoria

Antes de escribir consultas, identifica lenguaje, ORM, versión, driver, base de datos, estrategia de migraciones, pool de conexiones, transacciones, relaciones, soft delete, hooks, caché y comandos del proyecto. Lee el esquema, modelos, migraciones, repositorios existentes y pruebas. No cambies de ORM ni mezcles dos patrones sin una decisión arquitectónica explícita.

## Separación de modelos

Distingue cuatro representaciones cuando sea necesario: DTO de entrada/salida, modelo de persistencia del ORM, entidad o agregado de dominio y modelo de integración. Para CRUD simple pueden coincidir parcialmente, pero esa coincidencia debe ser consciente. El dominio no debe depender de decorators, tipos generados, sesiones, lazy loading, nombres de tablas o excepciones concretas del ORM.

El repositorio o adaptador traduce entre persistencia y dominio. Expone operaciones orientadas al caso de uso, no una API genérica que filtre detalles del ORM a toda la aplicación. Mantén queries complejas cerca del módulo que conoce su significado.

## Consultas y rendimiento

Proyecta solo columnas necesarias. Evita N+1, eager loading indiscriminado, joins innecesarios, consultas dentro de bucles y paginación por offset en conjuntos grandes cuando el cursor sea más adecuado. Revisa índices, cardinalidad, planes de ejecución, filtros por tenant, ordenamiento y límites. No optimices solo por intuición: mide consultas lentas y carga real.

Toda consulta multi-tenant debe incluir el ámbito de tenant mediante contexto confiable, restricciones de base, políticas de fila o una combinación apropiada. Prueba tanto acceso permitido como intento de fuga. Nunca aceptes el tenant directamente desde un parámetro del cliente sin validación de autorización.

## Transacciones y concurrencia

Define el límite transaccional en el caso de uso. No mantengas transacciones abiertas durante llamadas HTTP, procesamiento lento o servicios externos. Conoce el nivel de aislamiento real del motor. Para concurrencia usa restricciones únicas, control optimista, bloqueo pesimista o serialización solo cuando el riesgo lo justifique.

Una transacción local no convierte en atómica una llamada externa, una publicación de mensaje o una escritura en otra base. Para combinar base de datos y eventos considera outbox, consumidores idempotentes y reintentos controlados. Revisa deadlocks, timeouts y reintentos de transacción.

## Migraciones

Toda modificación de esquema debe ser versionada, revisable, automatizable y compatible con el despliegue. Prefiere expandir, migrar y contraer: primero añade estructuras compatibles, después despliega consumidores, migra datos por lotes y finalmente elimina lo obsoleto cuando ninguna versión activa lo necesite.

No edites migraciones ya aplicadas. Revisa locks, duración, tamaño de tabla, índices concurrentes, defaults, nullable, backfills, triggers, datos existentes, réplica, rollback y diferencias entre entornos. En bases compartidas, shards o bases dedicadas, define cómo se aplica la migración a cada ubicación y cómo se verifica que ninguna quedó atrasada.

## Relaciones y ciclo de vida

Haz explícitos ownership, cardinalidad, cascadas, borrado lógico, retención y referencias huérfanas. No confíes en cascadas implícitas para datos críticos. Los hooks del ORM no deben esconder efectos relevantes como movimientos financieros, auditoría, eventos o llamadas externas; esas reglas deben vivir en casos de uso o dominio.

## Pool y operación

Configura límites de conexiones, timeouts, healthchecks, reintentos, límites de concurrencia y cierre ordenado. En un monorepo con varias aplicaciones o shards, calcula el total potencial de conexiones por instancia y evita que el escalado de una aplicación agote la base. Observa latencia, errores, conexiones activas, esperas, locks, consultas lentas y tamaño de pool.

## Caché y ORM

No caches entidades mutables sin una política clara. Define clave, ámbito, versión, TTL, invalidación y comportamiento ante datos obsoletos. Si se cachean resultados de consultas, la clave debe incluir todos los filtros relevantes, incluido tenant, usuario, permisos y versión del esquema. Después de migraciones o cambios de escritura, invalida o versiona las entradas afectadas.

## Pruebas obligatorias

Prueba mapeos entre dominio y persistencia, restricciones únicas, nulls, relaciones, migraciones desde datos reales representativos, transacciones, rollback, concurrencia, N+1, paginación, aislamiento tenant, caché y errores del driver. Las unitarias no sustituyen integración cuando el comportamiento depende del motor de base de datos o del ORM.

## Regla de decisión

El ORM debe reducir trabajo repetitivo sin ocultar el costo de las consultas ni debilitar el modelo de dominio. Cuando una abstracción del ORM dificulte rendimiento, consistencia o claridad, usa una consulta especializada o SQL directo dentro de un adaptador documentado y probado. La herramienta se adapta al diseño; el diseño no se subordina ciegamente a la herramienta.
