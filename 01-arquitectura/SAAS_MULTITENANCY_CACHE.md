# SaaS multi-tenant, bases de datos y caché

## Objetivo

Este estándar permite operar un SaaS con muchos tenants, distintas bases de datos y diferentes niveles de aislamiento. La aplicación debe poder comenzar con almacenamiento compartido, repartir carga entre shards, promover un tenant a una base dedicada y moverlo de una base a otra sin perder datos ni interrumpir el servicio más allá del objetivo acordado.

La unidad de aislamiento debe definirse explícitamente. En la mayoría de SaaS es el **tenant**, no el usuario individual. Si el negocio exige que algunos usuarios tengan residencia dedicada, trátalo como una política de residencia por tenant, organización, usuario o conjunto de datos; no mezcles esas unidades sin documentar las reglas de acceso.

## Modelo de control recomendado

Mantén un **catálogo de control** separado de las bases de datos de negocio. El catálogo debe registrar tenant, plan, estado, modo de aislamiento, ubicación actual, capacidad, versión de esquema, región, cifrado, límites, migración activa y salud. Nunca determines la base destino leyendo una preferencia enviada por el cliente.

```text
Request
  → identidad y autorización
  → resolución segura de tenant/usuario
  → catálogo de control
  → router de residencia
  → pool de conexión de la base destino
  → operación de negocio
  → auditoría, métricas y caché aislada
```

| Dato del catálogo | Propósito |
|---|---|
| `tenant_id` | Identidad lógica estable |
| `residency_mode` | Compartida, shard, dedicada o híbrida |
| `database_id` / `shard_id` | Ubicación actual de los datos |
| `schema_version` | Compatibilidad de estructura |
| `capacity_class` | Límites y capacidad contratada |
| `migration_state` | Estado de movimiento, si existe |
| `region` | Residencia geográfica y latencia |
| `version` | Control optimista de cambios de ubicación |
| `health_status` | Salud de la base y capacidad disponible |

## Estrategias de residencia

| Estrategia | Descripción | Ventajas | Riesgos y uso |
|---|---|---|---|
| Base compartida con columna de tenant | Muchos tenants en las mismas tablas, filtrados por `tenant_id` | Bajo costo y operación simple | Mayor riesgo de fuga; exige aislamiento probado |
| Esquema por tenant | Cada tenant tiene esquema lógico separado en una base | Mejor separación lógica | Muchas migraciones y objetos; depende del motor |
| Shard compartido | Cada base contiene varios tenants con límites de capacidad | Escala horizontal y costo equilibrado | Rebalanceo, routing y consistencia operativa |
| Base dedicada por tenant | Base aislada para un tenant | Aislamiento, rendimiento y compliance | Coste, backups, upgrades y observabilidad por base |
| Híbrida | Mezcla de compartida, shards y dedicadas | Ajuste a planes y necesidades | Mayor complejidad del catálogo y automatización |

La recomendación general es iniciar con **base compartida o shards pequeños**, siempre que exista aislamiento fuerte, y promover solo los casos que lo requieran por volumen, ruido de vecinos, seguridad, residencia, SLA o personalización. La base dedicada debe ser una política declarada, no una excepción manual escondida.

## Cuotas y capacidad

Define límites por tenant y por base: usuarios activos, almacenamiento, filas, conexiones, operaciones por minuto, tamaño de archivos, jobs, eventos y consumo de caché. Las cuotas deben existir en el catálogo y aplicarse en el caso de uso, no solo en la interfaz.

Mide capacidad por tenant y por base. Señales para reubicar pueden ser CPU, memoria, I/O, conexiones, latencia, crecimiento, tamaño de tablas, errores, saturación de caché o incumplimiento de SLO. Nunca uses únicamente cantidad de usuarios: dos tenants con igual número pueden tener cargas radicalmente diferentes.

## Migración entre bases

Una migración debe ser una máquina de estados auditable, no un script improvisado. El flujo recomendado es:

```text
precheck → snapshot/backup → schema compatible → copia inicial
→ replicación de cambios → validación → cutover corto
→ actualización atómica del catálogo → invalidación de caché
→ observación → confirmación → limpieza diferida
```

La base origen y destino deben tener una versión de esquema compatible. Usa copia inicial por lotes, control de velocidad, reanudación, checksum o conteos verificables, y captura de cambios cuando el corte no pueda detener escrituras. Durante el cutover, bloquea o serializa las operaciones afectadas según el modelo de consistencia. Cambia la residencia mediante una actualización atómica y versionada del catálogo.

Cada operación debe ser idempotente y registrar tenant, origen, destino, versión, timestamps, filas procesadas, errores, reintentos y operador o proceso. Conserva el origen hasta superar la ventana de verificación y backup. Debe existir rollback claro: volver a enrutar, restaurar caché y reabrir escrituras solo si no hubo divergencia posterior.

Nunca migres datos financieros, auditoría o relaciones críticas sin validación de integridad. Para usuarios individuales, define cómo se mantienen referencias, permisos, sesiones, archivos, índices, eventos y reportes que cruzan la frontera de residencia.

## Aislamiento y seguridad

Toda consulta debe obtener el tenant desde contexto confiable y aplicar aislamiento en servidor. Los repositorios no deben aceptar libremente un `tenant_id` arbitrario proveniente de la UI. Añade pruebas negativas que intenten leer, modificar, cachear, exportar o migrar datos de otro tenant. Las herramientas administrativas requieren autorización elevada, doble revisión y auditoría.

Separa credenciales y pools cuando el nivel de aislamiento lo requiera. Cifra backups, conexiones y datos sensibles. Define retención, eliminación, exportación, residencia regional y recuperación por tenant. La observabilidad nunca debe filtrar datos de un tenant a otro.

## Caché

La caché es una optimización o una proyección temporal, no la fuente de verdad por defecto. Cada entrada debe tener propietario, formato, TTL, versión, límites, política de invalidación y comportamiento ante fallo.

| Patrón | Cuándo usarlo | Riesgo principal |
|---|---|---|
| Cache-aside | Lecturas frecuentes y recomputables | Datos obsoletos o stampede |
| Write-through | Se necesita actualizar caché junto con escritura | Acoplamiento y latencia |
| Write-behind | Escritura diferida tolerable | Pérdida o reordenamiento |
| Refresh-ahead | Datos previsibles y costosos | Complejidad de planificación |
| Caché local | Datos pequeños y muy calientes | Divergencia entre instancias |
| Caché distribuida | Varias instancias comparten lecturas | Red, disponibilidad y aislamiento |

Las claves deben incluir al menos versión, entorno, tipo de dato y ámbito de tenant o usuario cuando corresponda. Ejemplo conceptual: `v2:prod:tenant:{tenantId}:user:{userId}:resource:{id}`. Nunca permitas que una clave global contenga datos tenant-specific.

Define TTL con jitter, límites de tamaño, serialización segura y protección contra cache stampede. Usa locks breves o single-flight solo cuando sea necesario. Invalida por evento después de una escritura confirmada y acepta que la invalidación distribuida puede fallar: el TTL y la reconstrucción deben ser seguros. Para datos de permisos, caja, inventario, cuotas y residencia, prefiere consistencia fuerte o bypass de caché cuando el riesgo de dato obsoleto sea alto.

La caché debe tolerar miss, timeout, datos corruptos, evicción y caída completa. La aplicación debe degradar de forma controlada, sin convertir una caída de Redis en pérdida de datos ni en una cascada de consultas. Mide hit ratio, latencia, evicciones, tamaño, errores, hot keys y carga de reconstrucción, segmentadas por aplicación y tenant cuando sea seguro.

## Checklist operativo

Antes de habilitar una nueva estrategia, documenta router, catálogo, cuotas, migraciones, backups, restore, esquema, credenciales, SLO, alertas, métricas, pruebas de aislamiento y plan de rollback. Prueba carga desigual, tenant ruidoso, base llena, caída de una base, pérdida de caché, migración interrumpida, reintentos duplicados y cambio de residencia durante una solicitud.

## Decisión resumida

La política base es **multi-tenant híbrida y evolutiva**: compartir cuando sea seguro y económico; repartir en shards cuando la capacidad lo requiera; dedicar una base cuando lo justifiquen aislamiento, compliance, rendimiento o contrato; y hacer todos los movimientos mediante catálogo, migración auditable, validación, cutover controlado y caché tenant-aware.
