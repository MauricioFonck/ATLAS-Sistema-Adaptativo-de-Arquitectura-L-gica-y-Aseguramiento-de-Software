# API Gateway — Estándar profesional de borde de APIs

## Principio principal

Un API Gateway es **infraestructura de borde**: recibe tráfico, lo autentica, lo limita, lo observa y lo enruta. No es un lugar para reglas de negocio. Todo lo que el gateway hace debe poder explicarse como una política transversal, declarativa y versionada; todo lo que dependa del dominio pertenece al servicio que lo posee.

El gateway reduce el acoplamiento entre clientes y servicios, centraliza controles repetitivos y ofrece un punto único para observar y proteger la superficie pública. A cambio introduce un salto de red, un componente crítico de disponibilidad y un riesgo de convertirse en un monolito de configuración compartido. Microsoft describe estas ventajas y riesgos en su guía de gateways para microservicios [1].

## Cuándo introducirlo

Aplica la regla de oro de ATLAS y la escalera de Ponytail (`../02-diseno-y-codigo/PONYTAIL_SIMPLICITY.md`): el gateway se introduce cuando resuelve un problema presente, no uno imaginado.

| Situación | Opción preferida | Motivo |
|---|---|---|
| Un solo backend y un solo cliente | Reverse proxy o ingress con TLS, límites de tamaño y rate limit básico | Un gateway completo no aporta y añade un componente crítico |
| Varios servicios detrás de un dominio público | API Gateway de borde | Enrutamiento, autenticación y límites en un único punto |
| Varios tipos de cliente con necesidades distintas (web, móvil, terceros) | Backend for Frontend (BFF) por tipo de cliente, detrás o en lugar del gateway | Evita que un gateway genérico acumule lógica específica de cada cliente [2] |
| APIs expuestas a socios o monetizadas | API Management (claves de aplicación, planes, cuotas, portal de desarrolladores) | Gobierno del ciclo de vida y del consumo externo |
| Tráfico servicio a servicio dentro del clúster (este-oeste) | Service mesh o mTLS entre servicios, no el gateway de borde | El gateway gobierna norte-sur; hacer pasar tráfico interno por él crea un cuello de botella |
| Kubernetes | Implementación compatible con Kubernetes Gateway API | Estándar portable entre proveedores de gateway [3] |

Registra la decisión en un ADR (`../05-plantillas/ADR_TEMPLATE.md`) indicando qué problema concreto resuelve, qué alternativa más simple se descartó y cuál es la señal que obligaría a revisarla.

## Responsabilidades

| Pertenece al gateway | Nunca pertenece al gateway |
|---|---|
| Terminación TLS, HSTS y redirección a HTTPS | Reglas de negocio, cálculos o validaciones de dominio |
| Autenticación: validación de tokens, claves de aplicación o mTLS | Autorización a nivel de objeto (¿este usuario puede ver *este* pedido?) |
| Autorización gruesa por ruta y scope | Orquestación de transacciones entre servicios |
| Validación estructural contra el contrato OpenAPI | Transformaciones de cuerpo que cambian el significado del mensaje |
| Rate limiting, cuotas, límites de tamaño y de concurrencia | Acceso directo a bases de datos |
| Enrutamiento, versionado de rutas, canary y blue-green | Agregación específica de una pantalla (eso es un BFF con dueño) |
| Propagación de trazas, identificador de correlación y logs de acceso | Estado de sesión de usuario |
| CORS explícito y normalización de cabeceras | Secretos embebidos en la configuración |
| Respuestas de error uniformes para fallos propios del borde | Reescritura silenciosa de errores del servicio que oculte su causa |

Una política que aparece en el gateway para un solo servicio es una señal de que pertenece a ese servicio.

## Topologías

| Topología | Uso | Riesgo principal |
|---|---|---|
| Gateway de borde único | Pocos servicios y un equipo | Crece hasta ser un monolito de configuración sin dueño |
| Gateway por dominio o por equipo | Varios equipos con despliegue autónomo | Políticas de seguridad divergentes; exige una base común versionada |
| BFF por cliente | Web, móvil y terceros con contratos distintos | Duplicación de lógica si el BFF invade el dominio |
| Gateway + service mesh | Muchos servicios con mTLS interno obligatorio | Dos planos de configuración que deben diseñarse juntos |

El ownership debe ser explícito: cada ruta tiene un equipo propietario, y la configuración común (TLS, autenticación, límites globales, logging) tiene un propietario de plataforma.

## Contrato primero

La especificación OpenAPI 3.1 [4] es la fuente de verdad de cada API expuesta. De ella se derivan la validación del gateway, la documentación, las pruebas de contrato y, cuando aplique, los SDK de clientes. El gateway valida método, ruta, parámetros, cabeceras, tipo de contenido y esquema del cuerpo antes de reenviar; el servicio vuelve a validar sus invariantes, porque la validación de borde no sustituye la del dominio.

**Versionado.** Versiona solo en cambios incompatibles, con la versión mayor en la ruta (`/v1/`) o en una cabecera documentada; elige una convención por sistema y no las mezcles. Los cambios compatibles (campos nuevos opcionales, endpoints nuevos) no crean versión. Anuncia retiradas con las cabeceras `Deprecation` [5] y `Sunset` [6], con fecha, enlace a la guía de migración y métricas de uso de la versión antigua antes de apagarla.

**Errores.** Todos los errores, tanto los generados por el gateway (401, 403, 404 de ruta, 413, 429, 502, 503, 504) como los de los servicios, siguen el formato `application/problem+json` de RFC 9457 [7]: `type`, `title`, `status`, `detail`, `instance` y un campo de correlación. Nunca se exponen trazas de pila, nombres de hosts internos, versiones de software ni mensajes de la base de datos.

## Seguridad

Aplica defensa en profundidad y confianza cero: el gateway es el primer control, nunca el único. El OWASP API Security Top 10 2023 [8] es la lista mínima de amenazas a revisar.

| Riesgo OWASP API 2023 | Control en el gateway | Control obligatorio en el servicio |
|---|---|---|
| API1 Broken Object Level Authorization | Ninguno suficiente | Verificar propiedad del recurso en cada acceso |
| API2 Broken Authentication | Validación estricta de tokens, mTLS, límites a endpoints de login | Sesiones y credenciales gestionadas por un proveedor de identidad |
| API3 Broken Object Property Level Authorization | Validación de esquema de entrada | Filtrar propiedades de salida por permisos; prohibir asignación masiva |
| API4 Unrestricted Resource Consumption | Rate limit, cuotas, tamaño de payload, timeouts, paginación máxima | Límites de consultas y trabajo por petición |
| API5 Broken Function Level Authorization | Scopes y roles por ruta y método | Recomprobar permisos de la función |
| API6 Unrestricted Access to Sensitive Business Flows | Límites por identidad, detección de automatización | Reglas de negocio anti-abuso |
| API7 Server Side Request Forgery | Bloquear rutas no declaradas | Allowlist de destinos salientes y egress controlado |
| API8 Security Misconfiguration | Configuración como código, CORS explícito, cabeceras de seguridad | Defaults seguros en cada servicio |
| API9 Improper Inventory Management | Catálogo de rutas y versiones, rechazo de rutas no registradas | Retirada real de versiones antiguas |
| API10 Unsafe Consumption of APIs | — | Validar y limitar respuestas de terceros como entrada no confiable |

**Autenticación.** Para usuarios, OAuth 2.0 / OpenID Connect siguiendo la guía de seguridad actual de OAuth [9]. Al validar un JWT, el gateway comprueba firma con una allowlist de algoritmos (nunca `none`, nunca el algoritmo que declara el propio token sin restricción), `iss`, `aud`, `exp`, `nbf` y tolerancia de reloj acotada, según las buenas prácticas de JWT [10]. Las claves públicas se obtienen del JWKS del emisor con caché y soporte de rotación. Las claves de API identifican aplicaciones, no usuarios, y no sustituyen a la autenticación de usuario. Para socios y tráfico entre el gateway y los servicios, prefiere mTLS.

**Propagación de identidad.** El gateway elimina cualquier cabecera de identidad que venga del cliente (`X-User-Id`, `X-Tenant-Id`, `X-Roles` o equivalentes) antes de añadir las suyas. Los servicios no aceptan esas cabeceras salvo que lleguen por un canal autenticado con el gateway (mTLS o token firmado reenviado); los backends no son alcanzables directamente desde internet.

**Otras reglas.** CORS con allowlist explícita de orígenes; nunca `*` junto con credenciales. Límites de tamaño de cuerpo, de cabeceras y de URL. Normalización de rutas antes de enrutar para evitar bypass por codificación. Métodos no declarados devuelven 405. Los secretos (certificados, credenciales de upstream, claves de firma) viven en un gestor de secretos, nunca en el repositorio de configuración.

## Control de tráfico y resiliencia

**Rate limiting y cuotas.** Limita por identidad (aplicación, usuario y, en SaaS, tenant), no solo por IP. Responde `429` con `Retry-After` y, cuando el stack lo soporte, las cabeceras `RateLimit` del borrador IETF [11]. En sistemas multi-tenant las cuotas son tenant-aware y protegen contra noisy neighbors, en coherencia con `SAAS_MULTITENANCY_CACHE.md`. Define explícitamente qué ocurre si el almacén compartido del limitador cae: fail-open documentado para límites de protección de capacidad, fail-closed para límites contractuales o de seguridad.

**Timeouts.** Cada ruta declara su timeout. El presupuesto se reparte de fuera hacia dentro: cliente > gateway > servicio > dependencias, para que la capa externa no abandone una petición que la interna sigue procesando. Un timeout de upstream devuelve `504` en formato problem.

**Reintentos.** Solo para errores transitorios y solo en métodos idempotentes (GET, HEAD, PUT, DELETE) o en POST con clave de idempotencia. Con backoff exponencial, jitter y presupuesto de reintentos. Reintenta en una sola capa: reintentos en cliente, gateway y servicio a la vez multiplican la carga durante un incidente.

**Idempotencia.** Las operaciones POST con efectos (pagos, pedidos, movimientos) aceptan una cabecera `Idempotency-Key` [12]; el servicio almacena el resultado por clave, porque solo él conoce el efecto de negocio.

**Circuit breaker y load shedding.** Abre el circuito ante una dependencia degradada y responde rápido con `503` y `Retry-After`. Ante saturación, descarta primero tráfico de menor prioridad en lugar de degradar a todos.

**Caché en el borde.** Solo para respuestas `GET` cacheables según `Cache-Control`. Las respuestas autenticadas o por tenant no se cachean salvo que la clave incluya la identidad o el tenant; en caso de duda, no se cachea en el gateway.

## Observabilidad

El gateway propaga W3C Trace Context (`traceparent`, `tracestate`) [13] y genera un identificador de correlación que se devuelve al cliente y aparece en los errores. Los logs de acceso son estructurados e incluyen ruta plantilla (no la URL con identificadores), método, estado, latencia total, latencia de upstream, tamaño, identificador de aplicación, tenant y versión de API. Nunca registran tokens, cabeceras `Authorization`, cookies, cuerpos con datos personales ni claves de API completas.

Mide tráfico, errores y latencia (p50, p95, p99) por ruta y por consumidor, más rechazos por autenticación, por rate limit y por validación. Define SLO por API y alerta sobre consumo de presupuesto de error, no sobre umbrales aislados. Un pico de `401`/`403` o de `429` es una señal de seguridad, no solo operativa.

## Configuración como código y entrega

La configuración del gateway es código: vive en git, se revisa en pull request, se valida en CI (lint del OpenAPI, validación de esquema de la configuración, pruebas) y se despliega por pipeline con promoción entre entornos. Los cambios manuales en consola están prohibidos en producción; si ocurren por emergencia, se reconcilian en el repositorio el mismo día.

Cada cambio de ruta es reversible: canary o blue-green en el propio gateway, métricas comparadas antes de promover y rollback en un paso. Una ruta nueva no se expone sin contrato, propietario, autenticación declarada (incluida la decisión explícita de hacerla pública), límites y pruebas.

## Alta disponibilidad

El gateway es stateless y se ejecuta en varias réplicas detrás de un balanceador, en más de una zona cuando el SLO lo exija. Distingue liveness (el proceso vive) de readiness (puede servir tráfico: configuración cargada, JWKS disponible). Las dependencias del gateway (proveedor de identidad, almacén del rate limiter, gestor de secretos) tienen su propio plan de fallo documentado. Su capacidad se prueba con carga antes de cada evento de tráfico previsible.

## Selección tecnológica

No hay una herramienta universal; elige por el problema y por lo que el equipo puede operar.

| Categoría | Ejemplos | Cuándo encaja |
|---|---|---|
| Reverse proxy | NGINX, Traefik, Caddy, HAProxy | Pocos servicios; TLS, enrutamiento y límites básicos |
| Gateway autogestionado | Kong, Apache APISIX, Tyk, KrakenD, Envoy Gateway | Control total, multi-cloud o on-premise, plugins |
| Gateway gestionado en la nube | AWS API Gateway, Azure API Management, Google Apigee / API Gateway | Operación delegada, integración con la identidad y facturación del proveedor |
| Estándar Kubernetes | Implementaciones de Gateway API | Portabilidad entre gateways dentro de Kubernetes |

Criterios: soporte de OpenAPI y validación, modelo de autenticación, rate limiting distribuido, observabilidad nativa (OpenTelemetry), configuración declarativa, latencia añadida medida, costo por petición, lock-in y capacidad real del equipo para operarlo. Detecta primero si el proyecto ya tiene un proxy, ingress o gateway; extiéndelo antes de añadir otro.

## Pruebas

Además de la estrategia de `../03-calidad-y-seguridad/QA_STRATEGY.md`, toda API expuesta por el gateway tiene pruebas automatizadas de borde:

| Tipo | Casos mínimos |
|---|---|
| Positivas | Token válido con scope correcto llega al servicio con la identidad propagada; traza y correlación presentes |
| Autenticación negativa | Sin token, token expirado, firma inválida, `alg: none`, `aud` o `iss` incorrectos → `401` problem+json |
| Autorización negativa | Scope insuficiente → `403`; acceso a recurso de otro usuario o tenant → rechazado por el servicio |
| Suplantación de cabeceras | `X-User-Id` o `X-Tenant-Id` enviados por el cliente no llegan al servicio |
| Límites | Payload excesivo → `413`; rate limit superado → `429` con `Retry-After`; método no declarado → `405` |
| CORS | Origen no permitido no recibe cabeceras CORS |
| Resiliencia | Upstream lento → `504`; upstream caído → `502`/`503` sin filtrar detalles internos |
| Contrato | Respuestas del servicio conformes con el OpenAPI publicado |
| Carga | Latencia añadida por el gateway y comportamiento en saturación medidos |

## Checklist por ruta

| Pregunta | Respuesta requerida |
|---|---|
| ¿Contrato OpenAPI publicado y validado? | Ruta del archivo y versión |
| ¿Propietario? | Equipo o persona |
| ¿Autenticación? | Esquema, o decisión explícita y documentada de ruta pública |
| ¿Scopes o roles por método? | Lista |
| ¿Rate limit y cuota? | Por identidad, tenant y valor |
| ¿Timeout y reintentos? | Valores y métodos reintentables |
| ¿Idempotencia? | Obligatoria en POST con efectos |
| ¿Tamaño máximo de payload? | Valor |
| ¿Caché? | Sí con clave y TTL, o no |
| ¿Observabilidad? | Dashboard, SLO y alertas |
| ¿Pruebas de borde? | Positivas y negativas en CI |
| ¿Rollback? | Mecanismo y responsable |

## Anti-patrones

- Lógica de negocio o reglas por cliente dentro de plugins o scripts del gateway.
- Confiar en el gateway como única barrera de autorización.
- Servicios accesibles directamente sin pasar por el borde.
- Un gateway central compartido por todos los equipos sin dueño ni revisión.
- Reintentos en todas las capas y timeouts sin presupuesto.
- Configuración editada a mano en consola de producción.
- Errores con formatos distintos según la ruta o filtrando detalles internos.
- Adoptar un gateway completo para un único backend.

## Relación con Ponytail

La simplicidad decide **si** hace falta un gateway y **cuánto** gateway; nunca decide quitar un control de seguridad. Con un único backend, un reverse proxy bien configurado es la respuesta correcta. Cuando el gateway ya existe o está justificado, la autenticación, los límites, los timeouts, la validación y la observabilidad descritos aquí son el mínimo, no ornamento: pertenecen a la categoría "nunca simplificar" de Ponytail.

## Referencias

[1]: https://learn.microsoft.com/en-us/azure/architecture/microservices/design/gateway "Microsoft Learn — Use API gateways in microservices"
[2]: https://learn.microsoft.com/en-us/azure/architecture/patterns/backends-for-frontends "Microsoft Learn — Backends for Frontends pattern"
[3]: https://gateway-api.sigs.k8s.io/ "Kubernetes Gateway API"
[4]: https://spec.openapis.org/oas/v3.1.0 "OpenAPI Specification 3.1"
[5]: https://www.rfc-editor.org/rfc/rfc9745 "RFC 9745 — The Deprecation HTTP Response Header Field"
[6]: https://www.rfc-editor.org/rfc/rfc8594 "RFC 8594 — The Sunset HTTP Header Field"
[7]: https://www.rfc-editor.org/rfc/rfc9457 "RFC 9457 — Problem Details for HTTP APIs"
[8]: https://owasp.org/API-Security/editions/2023/en/0x11-t10/ "OWASP API Security Top 10 — 2023"
[9]: https://www.rfc-editor.org/rfc/rfc9700 "RFC 9700 — Best Current Practice for OAuth 2.0 Security"
[10]: https://www.rfc-editor.org/rfc/rfc8725 "RFC 8725 — JSON Web Token Best Current Practices"
[11]: https://datatracker.ietf.org/doc/draft-ietf-httpapi-ratelimit-headers/ "IETF draft — RateLimit header fields for HTTP"
[12]: https://datatracker.ietf.org/doc/draft-ietf-httpapi-idempotency-key-header/ "IETF draft — The Idempotency-Key HTTP Header Field"
[13]: https://www.w3.org/TR/trace-context/ "W3C — Trace Context"
