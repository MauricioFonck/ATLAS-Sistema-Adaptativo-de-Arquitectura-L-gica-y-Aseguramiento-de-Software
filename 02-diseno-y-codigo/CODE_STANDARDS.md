# Estándares de diseño y código

## Principio rector

El código debe comunicar intención. La claridad, la cohesión y la capacidad de prueba tienen prioridad sobre la brevedad, la abstracción y la optimización prematura.

## Reglas de diseño

| Principio | Aplicación práctica |
|---|---|
| Single Responsibility | Un módulo debe tener una razón dominante para cambiar. |
| Open/Closed | Extiende mediante composición o estrategias cuando exista variación real. |
| Liskov Substitution | Una implementación debe respetar el contrato completo de su abstracción. |
| Interface Segregation | Interfaces pequeñas, orientadas a consumidores, no interfaces universales. |
| Dependency Inversion | Las políticas definen puertos; la infraestructura los implementa. |
| DRY | No dupliques conocimiento; una pequeña repetición puede ser mejor que una abstracción falsa. |
| KISS | Elige el diseño más simple que cubra requisitos actuales. |
| YAGNI | No construyas capacidades hipotéticas sin evidencia. |
| Composition over inheritance | Usa composición para combinar comportamientos y reducir jerarquías frágiles. |
| Explicit over implicit | Haz visibles dependencias, efectos secundarios, estados y errores. |

## Funciones, clases y módulos

Una función debe hacer una cosa reconocible, tener entradas y salidas claras y evitar efectos secundarios ocultos. Las clases deben proteger invariantes o encapsular una responsabilidad concreta; no deben ser contenedores de métodos sin cohesión. Si un módulo necesita conocer demasiados detalles de otro, revisa el límite.

Usa nombres de dominio, no nombres genéricos como `Manager`, `Helper`, `Utils` o `Data`. Evita booleanos ambiguos; `isActive` comunica mejor que `flag`. Prefiere tipos que hagan inválidos los estados imposibles. Representa dinero, fechas, zonas horarias, identificadores y unidades mediante tipos o value objects adecuados.

## Errores y contratos

Distingue errores de validación, errores de negocio, errores de infraestructura y errores inesperados. No uses excepciones como flujo normal si el lenguaje ofrece resultados explícitos. En APIs, usa códigos y contratos consistentes, sin filtrar trazas. En trabajos asíncronos, registra correlación, reintentos y estado final. Todo reintento debe tener límite, backoff y condición de idempotencia.

## Estado y concurrencia

Minimiza estado mutable compartido. Define quién es dueño de cada estado y cuál es su ciclo de vida. Usa transacciones en límites explícitos. No asumas que dos operaciones remotas son atómicas. Diseña para duplicados, orden variable, timeouts y caídas parciales.

## Patrones de diseño

Los patrones GoF son vocabulario, no una lista de tareas. Los más útiles como decisiones pragmáticas son Strategy para algoritmos intercambiables, Adapter para integrar modelos externos, Factory para proteger creación con invariantes, Decorator para añadir comportamiento, Observer o eventos para desacoplar reacciones y Facade para ofrecer una interfaz pequeña a un subsistema complejo.

No introduzcas Singleton para compartir estado; usa inyección de dependencias y ciclo de vida explícito. No uses Repository genérico para esconder diferencias importantes entre consultas. No uses Service Locator porque oculta dependencias. No conviertas cada clase en una interfaz: abstrae en fronteras donde exista una razón de sustitución o prueba.

## API y datos

Diseña contratos desde las necesidades del consumidor. Valida esquema, límites, paginación, ordenamiento y autorización. Usa nombres consistentes y evita devolver modelos internos directamente. Las migraciones deben ser incrementales: expandir, migrar, cambiar consumidores y contraer. No borres columnas o cambies semántica en una sola operación si existen versiones activas.

## Documentación y comentarios

La documentación debe explicar decisiones, invariantes, límites, ejemplos de uso y procedimientos de operación. Un comentario que contradice al código es un defecto. Para decisiones de impacto, usa la plantilla ADR. Para módulos complejos, documenta el modelo y sus términos ubicuos.

## Checklist de revisión

Antes de aprobar un cambio, verifica que el diseño sea entendible, que las dependencias apunten en la dirección correcta, que no se filtren detalles de infraestructura al dominio, que las validaciones estén en los límites y en las invariantes, que el manejo de errores sea explícito y que no exista duplicación de reglas. Revisa también compatibilidad, rendimiento, seguridad, logs y facilidad de rollback.
