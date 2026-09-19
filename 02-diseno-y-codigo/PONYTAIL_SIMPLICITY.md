# Simplicidad radical con Ponytail

## Propósito

Ponytail [1] es una skill para asistentes de IA que fuerza la solución más simple que realmente funciona: YAGNI, biblioteca estándar primero, funcionalidades nativas antes que dependencias, una línea antes que cincuenta. ATLAS la adopta como **mecanismo operativo** de su regla de oro, *diseñar para el cambio que conocemos, no para todos los cambios imaginables*, y de su principio de que ninguna abstracción entra sin eliminar una dependencia, proteger una regla de negocio o facilitar una evolución probable.

ATLAS define **qué** debe cumplir un sistema; Ponytail define **cuánto código** hace falta para cumplirlo. Juntos: comprensión completa, solución mínima, controles no negociables intactos.

## Orden de aplicación

1. **Comprender (ATLAS).** Protocolo antes de programar de `AI_INSTRUCTIONS.md`: leer el código afectado, trazar el flujo real, buscar consumidores, separar hechos de supuestos. Ponytail exige lo mismo: la escalera acorta la solución, nunca la lectura.
2. **Subir la escalera (Ponytail).** Detenerse en el primer peldaño que funcione:
   1. ¿Necesita existir? Si la necesidad es especulativa, no se construye y se dice en una línea.
   2. ¿Ya existe en este repositorio? Reutilizar.
   3. ¿Lo resuelve la biblioteca estándar?
   4. ¿Lo cubre una funcionalidad nativa de la plataforma (restricción de base de datos, CSS, elemento HTML)?
   5. ¿Lo resuelve una dependencia ya instalada?
   6. ¿Cabe en una línea?
   7. Solo entonces, el mínimo código que funciona.
3. **Aplicar el piso (ATLAS).** Verificar que la solución mínima conserva los controles no negociables de la tabla siguiente.
4. **Dejar una comprobación.** Lógica no trivial (una rama, un bucle, un parser, dinero, seguridad) deja la prueba más pequeña que fallaría si la lógica se rompe, según `../03-calidad-y-seguridad/QA_STRATEGY.md`.

**Un bug se corrige en la causa raíz.** Antes de editar, buscar todos los llamadores de la función afectada; una guarda en la función compartida es un diff más pequeño y correcto que una guarda en cada llamador.

## Lo que nunca se simplifica

Ponytail y ATLAS coinciden en este límite. Ninguna de estas categorías se elimina por brevedad:

| Categoría | Referencia ATLAS |
|---|---|
| Validación de entrada en fronteras de confianza | `../03-calidad-y-seguridad/QUALITY_SECURITY.md` |
| Autenticación, autorización y aislamiento de tenant | `../01-arquitectura/SAAS_MULTITENANCY_CACHE.md` |
| Manejo de errores que evita pérdida o corrupción de datos | `CODE_STANDARDS.md` |
| Transacciones, idempotencia y migraciones reversibles | `ORM_STANDARDS.md` |
| Controles de borde de APIs: autenticación, límites, timeouts, observabilidad | `../01-arquitectura/API_GATEWAY.md` |
| Accesibilidad básica | `CODE_STANDARDS.md` |
| Todo lo que el usuario pidió explícitamente | Jerarquía de decisiones del `README.md` |

Si el usuario insiste en la versión completa, se construye sin volver a discutirla.

## Resolución de tensiones

| Situación | Decisión |
|---|---|
| ATLAS sugiere un patrón (Strategy, Factory, puerto) y hay una sola implementación | No se crea; se anota la condición que lo justificaría |
| Monolito modular vs microservicios | Monolito modular; coincide con la decisión por defecto de ATLAS |
| Nueva dependencia vs unas líneas propias | Unas líneas propias, salvo criptografía, parsing de formatos complejos o fechas/zonas horarias, donde gana la biblioteca probada |
| Hexagonal/Clean en un CRUD trivial | Capas mínimas; ATLAS ya indica aplicarla solo donde protege complejidad |
| API Gateway para un solo backend | Reverse proxy; el gateway llega cuando haya varios upstreams o consumidores externos |
| Documentación viva, ADR y contratos | No son sobreingeniería: se escriben cuando la decisión tiene impacto futuro, breves |
| Prosa larga para justificar una simplificación | Se elimina; basta una línea: `omitido X, añadir cuando Y` |

## Niveles de intensidad

| Nivel | Uso recomendado en ATLAS |
|---|---|
| `lite` | Diseño de arquitectura, ADR y exploración: construye lo pedido y nombra la alternativa más simple en una línea |
| `full` | Implementación diaria. **Nivel por defecto** |
| `ultra` | Refactorizaciones, limpieza de deuda y auditorías: borrar antes que añadir |

Se cambia con `/ponytail lite|full|ultra`, se desactiva con `stop ponytail` o `normal mode`. El nivel por defecto se fija con la variable `PONYTAIL_DEFAULT_MODE` o en `%APPDATA%\ponytail\config.json` (`{"defaultMode": "full"}`).

## Deuda deliberada: comentarios `ponytail:`

Toda simplificación que corta una esquina real con un techo conocido (bloqueo global, recorrido O(n²), heurística ingenua) se marca en el código con un comentario que nombra el techo y la ruta de mejora:

```text
# ponytail: bloqueo global; usar bloqueos por cuenta si el throughput lo exige
```

Estos comentarios son el registro de deuda técnica que exige `../04-proceso/DELIVERY_WORKFLOW.md`: contexto, impacto y condición de pago en una línea. `/ponytail-debt` los recolecta en un inventario.

## Comandos en el flujo de entrega

| Momento | Comando | Resultado |
|---|---|---|
| Implementación | Ponytail activo en `full` | Solución mínima con la escalera |
| Antes del pull request | `/ponytail-review` sobre el diff | Qué borrar, simplificar o reemplazar; complementa, no sustituye, la revisión de corrección y seguridad |
| Mantenimiento periódico | `/ponytail-audit` | Informe del repositorio completo ordenado por impacto; no aplica cambios |
| Planificación de deuda | `/ponytail-debt` | Inventario de comentarios `ponytail:` |
| Referencia | `/ponytail-help` | Tarjeta de modos y comandos |

## Instalación local

Ponytail se instala como plugin local de Claude Code desde un clon del repositorio en `%USERPROFILE%\.claude\local-marketplaces\ponytail`, registrado como marketplace de tipo directorio:

```powershell
claude plugin marketplace add "$env:USERPROFILE\.claude\local-marketplaces\ponytail"
claude plugin install ponytail@ponytail --scope user
```

El plugin aporta las seis skills y los hooks `SessionStart`, `SubagentStart` y `UserPromptSubmit` que activan el modo en cada sesión y en cada subagente. Cada directorio de configuración de Claude Code (`CLAUDE_CONFIG_DIR`) necesita el plugin habilitado en su propio `settings.json`: `~/.claude` lo usan `claude` y `claude-deepseek`; `~/.claude-max` lo usa `claude-max`. Para actualizar: `git pull` en el clon y `claude plugin update ponytail@ponytail` en cada configuración. Requiere Node.js en el `PATH` para los hooks.

## Referencias

[1]: https://github.com/DietrichGebert/ponytail "DietrichGebert/ponytail — Lazy senior dev mode for AI agents (MIT)"
