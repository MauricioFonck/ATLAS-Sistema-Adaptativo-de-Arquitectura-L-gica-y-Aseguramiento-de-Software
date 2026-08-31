# Estrategia QA senior y automatización

## Propósito

QA no es la última etapa ni una actividad limitada a buscar bugs. Es una disciplina de prevención, evaluación de riesgo y protección del valor del producto. La calidad es responsabilidad de todo el equipo: negocio define comportamiento, desarrollo construye, QA diseña evidencia y todos colaboran para reducir defectos.

## Shift-left y análisis de riesgo

QA participa desde requisitos, diseño y arquitectura. Antes de implementar, revisa ambigüedades, estados imposibles, reglas contradictorias, permisos, datos sensibles, fallos de dependencias, escalabilidad y recuperabilidad. Clasifica cada riesgo por probabilidad, impacto, detectabilidad y costo de fallo. La cobertura debe concentrarse primero en dinero, identidad, permisos, multi-tenancy, migraciones, auditoría, integridad de datos y flujos críticos del usuario.

| Riesgo | Pregunta QA | Evidencia esperada |
|---|---|---|
| Funcional | ¿El sistema hace lo que el negocio espera? | Criterios de aceptación y pruebas de comportamiento |
| Integridad | ¿Puede crear datos duplicados o inconsistentes? | Pruebas de transacción, concurrencia e idempotencia |
| Seguridad | ¿Puede un actor acceder a más de lo permitido? | Pruebas negativas de autenticación y autorización |
| Multi-tenant | ¿Puede filtrarse información entre tenants? | Tests de aislamiento, consultas, caché y jobs |
| Resiliencia | ¿Qué ocurre ante timeout, retry o caída? | Pruebas de degradación y recuperación |
| Operación | ¿Se puede diagnosticar y revertir? | Logs, métricas, alertas, backup y rollback |

## Pruebas que deben funcionar

Una prueba positiva confirma que una entrada válida produce el resultado correcto. Incluye el flujo feliz, variantes legítimas, permisos válidos, datos mínimos y máximos aceptados, estados anteriores válidos, reintentos seguros y respuestas con contrato correcto. Cada criterio de aceptación debe tener al menos una evidencia positiva.

Ejemplos universales: un usuario autenticado puede ejecutar una acción autorizada; una venta válida actualiza los agregados correctos; una consulta devuelve solo los datos del tenant; una migración conserva los registros; una operación offline se sincroniza al recuperar conexión; una base dedicada recibe tráfico después del cutover; una caché válida acelera una lectura sin alterar el resultado.

## Pruebas que deben fallar

Una prueba negativa confirma que el sistema rechaza entradas o acciones inválidas de manera segura, predecible y observable. No basta con que falle: debe devolver el contrato de error correcto, no modificar datos, no revelar secretos y registrar información operativa suficiente.

| Categoría | Casos negativos mínimos |
|---|---|
| Entrada | Campos ausentes, tipos incorrectos, formato inválido, valores límite, payload excesivo, caracteres peligrosos |
| Identidad | Token ausente, expirado, manipulado, usuario deshabilitado, sesión revocada |
| Autorización | Rol insuficiente, recurso ajeno, tenant ajeno, endpoint administrativo sin permiso |
| Datos | Duplicado, referencia inexistente, estado inválido, conflicto de versión, migración incompatible |
| Multi-tenant | `tenant_id` alterado, cache key de otro tenant, shard incorrecto, exportación cruzada |
| Concurrencia | Dos escrituras simultáneas, retry duplicado, mensajes fuera de orden |
| Dependencias | Timeout, respuesta 5xx, respuesta malformada, base inaccesible, caché caída |
| Offline | Cola corrupta, token expirado, conflicto, operación ya procesada, dispositivo sin espacio |
| Seguridad | Inyección, acceso directo a objeto, rate limit excedido, subida de archivo no permitida |

La prueba debe comprobar explícitamente que no hubo efectos parciales. Para operaciones financieras o de inventario, valida saldos, movimientos, reservas, auditoría y conteos antes y después.

## Pirámide y niveles de automatización

Usa muchas pruebas rápidas de dominio y unidad, una cantidad equilibrada de integración y pocas pruebas UI end-to-end para recorridos críticos. La pirámide es una guía de distribución, no una ley mecánica.

| Nivel | Objetivo | Características |
|---|---|---|
| Unitario | Reglas y transformaciones | Rápido, determinista, sin red ni base real |
| Componente | Módulo con dependencias simuladas | Verifica colaboración y contratos internos |
| Integración | DB, colas, caché, filesystem o proveedor | Usa infraestructura real o efímeramente reproducida |
| Contrato | Compatibilidad entre productor y consumidor | Detecta rupturas antes del despliegue |
| API | Flujos HTTP o equivalente | Verifica esquema, auth, errores y persistencia |
| E2E/UI | Recorridos críticos | Pocos, estables y centrados en comportamiento |
| No funcional | Rendimiento, seguridad, accesibilidad y resiliencia | Se ejecuta según riesgo y entorno |

## API y frontend

Las pruebas de API deben validar status, esquema, headers, autenticación, autorización, errores, paginación, idempotencia, tiempos razonables y efectos en datos. Pueden implementarse con Postman/Newman, REST Assured, pytest, Supertest, Karate u otra herramienta idiomática del stack.

Las pruebas de frontend deben verificar comportamiento visible y accesible, no detalles frágiles de implementación. Usa selectores estables, Page Object Model o App Actions cuando aporten mantenimiento, y evita esperar tiempos fijos. Cypress, Playwright, Selenium, WebdriverIO, Detox, Appium o herramientas nativas son opciones según plataforma; la estrategia no depende del producto elegido.

## BDD y Gherkin

Usa BDD cuando ayude a alinear negocio, QA y desarrollo. Un escenario debe describir comportamiento, no implementación:

```gherkin
Característica: aislamiento de información por tenant

  Escenario: un usuario no puede consultar un recurso de otro tenant
    Dado que el usuario pertenece al tenant A
    Y existe un recurso perteneciente al tenant B
    Cuando intenta consultar el recurso del tenant B
    Entonces la operación es rechazada
    Y el sistema no revela si el recurso existe
    Y no se modifica ningún dato
```

## Automatización y CI/CD

El código de pruebas recibe la misma revisión, versionado, diseño limpio y mantenimiento que el código de producción. Usa fixtures aisladas, datos controlados, limpieza segura, paralelización solo cuando sea determinista y reportes con trazabilidad. No ocultes pruebas inestables: aíslalas, registra la causa y establece fecha de reparación.

En cada pull request ejecuta checks rápidos: formato, lint, tipos, unitarias, seguridad básica y contratos afectados. En integración o staging ejecuta DB, API, migraciones, E2E críticas, rendimiento acotado y pruebas de despliegue. Antes de producción verifica smoke tests, healthchecks, migraciones, observabilidad y rollback. El pipeline debe dejar artefactos, logs y reportes reproducibles.

## Defectos

Un reporte profesional contiene título preciso, entorno, versión o commit, severidad, prioridad, precondiciones, pasos reproducibles, resultado esperado, resultado actual, evidencia, logs, request/response, datos de prueba e impacto. Separa severidad técnica de prioridad de negocio. Investiga causa probable sin culpar a personas y añade una prueba de regresión al corregir.

| Severidad | Significado |
|---|---|
| Crítica | Pérdida de datos, vulnerabilidad grave, caída general o bloqueo de operación esencial |
| Alta | Función crítica inutilizable o impacto amplio con workaround insuficiente |
| Media | Función afectada con alternativa razonable o impacto acotado |
| Baja | Defecto visual, texto, ergonomía o caso de bajo impacto |

## Métricas útiles

Mide señales de valor, no volumen de actividad. Observa escape de defectos a producción, defectos reabiertos, tiempo de feedback del pipeline, duración y estabilidad de pruebas, porcentaje de escenarios críticos automatizados, cobertura de riesgos, tiempo de resolución y frecuencia de regresiones. La cobertura de líneas no sustituye la cobertura de comportamiento ni de riesgo.

## No funcionales

Según el sistema, incluye carga, estrés, soak, recuperación, seguridad, accesibilidad, compatibilidad, usabilidad, consumo, disponibilidad y recuperación ante desastre. En SaaS multi-tenant prueba tenants pequeños y grandes, noisy neighbors, crecimiento desigual, migración entre bases, caída de un shard, pérdida de caché, hot keys, límites de conexiones y aislamiento de métricas.

## Definition of QA Done

La feature tiene criterios positivos y negativos, riesgos identificados, pruebas automatizadas proporcionales, evidencia reproducible, regresión actualizada, seguridad revisada, documentación sincronizada y resultado del pipeline conocido. No se considera validada si solo funciona manualmente en el entorno del desarrollador.
