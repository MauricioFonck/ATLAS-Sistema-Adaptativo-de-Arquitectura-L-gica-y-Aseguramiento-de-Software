# Calidad, seguridad y operación

## Modelo de calidad

Evalúa cada entrega por adecuación funcional, confiabilidad, eficiencia, mantenibilidad, seguridad, compatibilidad y facilidad de operación. ISO/IEC 25010 define un modelo para establecer requisitos y medidas de calidad de producto [1]. Convierte las cualidades relevantes en objetivos observables.

| Área | Expectativa mínima |
|---|---|
| Correctitud | Casos felices, límites y errores están cubiertos. |
| Mantenibilidad | Código modular, nombres claros y deuda registrada. |
| Seguridad | Autenticación, autorización, validación y secretos gestionados. |
| Rendimiento | Latencia, throughput y consumo tienen objetivos medibles. |
| Confiabilidad | Timeouts, reintentos, recuperación y degradación están definidos. |
| Observabilidad | Logs, métricas y trazas permiten diagnosticar una operación. |
| Desplegabilidad | Cambios automatizados, reproducibles y con rollback. |

## Estrategia de pruebas

Prueba el comportamiento, no la implementación. Las pruebas unitarias deben ser rápidas y concentrarse en reglas de dominio. Las pruebas de integración verifican adaptadores, base de datos, colas y contratos reales. Las pruebas end-to-end cubren pocos recorridos críticos. Añade pruebas de contrato cuando equipos o servicios evolucionen de forma independiente.

La cobertura numérica no es un objetivo suficiente. Prioriza invariantes, caminos de alto riesgo, permisos, dinero, datos irreversibles, concurrencia y compatibilidad. Cada bug corregido debe incorporar una prueba de regresión proporcional.

## Seguridad por defecto

Sigue defensa en profundidad y mínimo privilegio. Valida y normaliza entradas en los límites; usa consultas parametrizadas; codifica salidas según el contexto; protege CSRF cuando aplique; aplica autenticación fuerte y autorización por recurso; limita peticiones; cifra datos sensibles en tránsito y reposo; rota secretos; actualiza dependencias; y registra eventos de seguridad sin exponer información sensible. OWASP mantiene prácticas de codificación segura y remite su guía archivada al Developer Guide [2].

Nunca guardes contraseñas en texto plano. No pongas secretos en el repositorio, imágenes, logs, URLs ni mensajes de error. No confíes en controles solo del frontend. Trata archivos subidos, webhooks, contenido generado por usuarios y respuestas de servicios externos como no confiables.

## Observabilidad

Cada operación importante debe tener un identificador de correlación. Los logs estructurados deben incluir evento, severidad, servicio, entorno, actor técnico o anónimo, duración, resultado y error sanitizado. Define métricas de tráfico, errores, latencia y saturación, además de métricas de negocio. Usa trazas distribuidas cuando existan llamadas remotas. Los dashboards y alertas deben corresponder a acciones operativas, no a ruido.

## Resiliencia

Toda llamada externa necesita timeout explícito. Los reintentos deben ser limitados, con backoff y solo para fallos transitorios. Usa circuit breaker cuando una dependencia degradada pueda producir una cascada. Diseña consumidores idempotentes, colas con dead-letter y mecanismos de replay controlado. Especifica qué puede degradarse y qué debe fallar de forma segura.

## Rendimiento

Mide antes de optimizar. Busca primero complejidad innecesaria, consultas N+1, falta de índices, payloads excesivos, serialización repetida y llamadas secuenciales. Define presupuestos de latencia y memoria. Usa caché solo con política de invalidación, TTL y límites. Toda optimización debe tener una medición antes/después y una prueba que preserve correctitud.

## Revisión de código

La revisión protege la salud acumulada del código, no solo busca defectos puntuales. Google establece como propósito principal que la salud general de la base de código mejore con el tiempo [3]. Mantén cambios pequeños, una intención por pull request, descripción clara, pruebas visibles y revisión enfocada en diseño, corrección, seguridad y mantenibilidad.

## Checklist específico del monorepo

Antes de aprobar cambios multi-tenant, prueba al menos un caso de acceso autorizado, un intento de acceso cruzado entre tenants, selección correcta entre control DB y shard, cache aislada, logs sin datos sensibles y rollback de migración. Los tests de integración deben usar PostgreSQL real cuando el comportamiento dependa de SQL, índices, transacciones o aislamiento.

Antes de aprobar cambios offline, prueba pérdida de conectividad, reconexión, reenvío duplicado, operaciones fuera de orden, token expirado, conflicto de edición, rechazo del servidor y recuperación de una cola local corrupta. Verifica que una operación repetida no duplique ventas, movimientos de caja, inventario o gastos.

Los workflows de `.github/` y los hooks de `.githooks/` son controles de calidad del producto. Si un cambio necesita omitirlos, documenta la razón, ejecuta el equivalente manualmente y crea una corrección para no depender de la omisión.

## Referencias

[1]: https://www.iso.org/obp/ui/#iso:std:iso-iec:25010:en "ISO/IEC 25010:2023 — Software product quality model"
[2]: https://owasp.org/www-project-secure-coding-practices-quick-reference-guide/ "OWASP — Secure Coding Practices Quick Reference Guide"
[3]: https://google.github.io/eng-practices/review/reviewer/standard.html "Google — The Standard of Code Review"
