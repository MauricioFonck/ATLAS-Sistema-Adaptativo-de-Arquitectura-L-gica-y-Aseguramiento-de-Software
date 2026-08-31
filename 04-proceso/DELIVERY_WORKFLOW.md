# Flujo de trabajo de entrega

## 1. Descubrimiento

Define el problema, usuario afectado, resultado esperado, alcance, no-alcance, restricciones y criterios de aceptación. Identifica riesgos técnicos y de negocio. Si el requisito es ambiguo, convierte la ambigüedad en preguntas o supuestos explícitos.

## 2. Diseño

Inspecciona el sistema existente y dibuja el flujo actual y el propuesto. Decide módulo, caso de uso, contrato, persistencia, errores, seguridad, observabilidad y estrategia de migración. Registra en un ADR toda decisión que afecte dependencias, datos, despliegue, rendimiento o evolución.

## 3. Implementación

Construye verticalmente una rebanada funcional pequeña: entrada, caso de uso, dominio, adaptador y prueba. Mantén commits pequeños y descriptivos. Evita mezclar refactorizaciones masivas con cambios funcionales. No cambies contratos públicos sin buscar consumidores y acordar compatibilidad.

## 4. Verificación

Ejecuta, según el proyecto, formateador, lint, type-check, compilación, pruebas unitarias, integración, end-to-end, análisis de dependencias, escáner de secretos y pruebas de rendimiento. Revisa manualmente autorización, logs, migraciones, rollback y casos límite.

## 5. Revisión y entrega

Revisa el diff completo. Actualiza documentación, ejemplos, variables de entorno, migraciones y runbooks. Describe pruebas ejecutadas, resultado, riesgos residuales y pasos de despliegue. La entrega no está completa si funciona localmente pero no puede operarse o revertirse con seguridad.

## Definition of Done

| Criterio | Verificación |
|---|---|
| Requisito | Todos los criterios de aceptación pasan. |
| Diseño | El cambio respeta los límites del módulo y las dependencias. |
| Código | Formato, lint y tipos pasan; no hay secretos ni duplicación crítica. |
| Pruebas | Los riesgos relevantes tienen pruebas automatizadas. |
| Seguridad | Entradas, permisos, datos y errores fueron revisados. |
| Operación | Logs, métricas, alarmas, migración y rollback están considerados. |
| Documentación | Se actualizaron contrato, README, ADR o runbook cuando corresponde. |
| Revisión | El diff fue revisado por otra persona o por una revisión sistemática de IA. |

## Gestión de deuda técnica

Registra deuda con contexto, impacto, riesgo y condición de pago. No escondas decisiones provisionales. Una solución temporal debe tener un propietario y una señal que indique cuándo deja de ser adecuada.

## Versionado y cambios incompatibles

Prefiere cambios compatibles hacia atrás. Para esquemas, usa expansión y contracción. Para APIs, mantén versiones solo cuando sea necesario y comunica deprecaciones. Para eventos, conserva consumidores tolerantes a campos nuevos y evita renombrar o reutilizar significados.

## Plantillas relacionadas

Usa `../05-plantillas/FEATURE_TEMPLATE.md` para preparar funcionalidades y `../05-plantillas/ADR_TEMPLATE.md` para decisiones arquitectónicas.
