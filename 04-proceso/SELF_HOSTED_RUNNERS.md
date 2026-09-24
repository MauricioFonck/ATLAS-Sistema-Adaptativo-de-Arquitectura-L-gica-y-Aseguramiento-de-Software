# Self-Hosted Runners — Estándar para ejecutar CI en infraestructura propia

## Principio principal

Un runner propio (*self-hosted runner*) es **infraestructura de producción que ejecuta código ajeno**: cada job descarga el repositorio y corre lo que diga el workflow, con acceso a la red, al disco, al Docker y a los secretos que se le pasen. Se adopta por una razón concreta y medible (límite de minutos o costo del CI alojado, hardware específico, acceso a red privada), nunca por comodidad, y se diseña para que el proyecto pueda **volver al CI alojado sin tocar código**.

La documentación oficial de GitHub fija los requisitos, la comunicación y el enrutamiento de jobs [1]. Este estándar añade las reglas que surgieron al operarlo en un equipo personal.

## Cuándo usarlo

Aplica la escalera de Ponytail (`../02-diseno-y-codigo/PONYTAIL_SIMPLICITY.md`): primero confirmar que el CI alojado no basta.

| Situación | Opción preferida | Motivo |
|---|---|---|
| Repositorio público | **CI alojado** (minutos gratis) | Un pull request de un fork podría ejecutar código en la máquina del runner; GitHub desaconseja runners propios en repos públicos [1] |
| Repo privado dentro del límite de minutos | CI alojado | Cero operación |
| Repo privado sin minutos, equipo personal disponible | Runner propio **Linux** con interruptor `vars.RUNNER` | Minutos ilimitados; vuelta atrás borrando una variable |
| Despliegue a producción | Runner **en el servidor de producción**, etiqueta propia (p. ej. `app-produccion`) | El equipo del desarrollador no debe ser el camino a producción |
| Muchos jobs en paralelo o equipo compartido | Runners efímeros con autoescalado (ARC o Scale Set Client) | GitHub recomienda efímeros para autoescalar: un job por runner y entorno limpio [1] |

## Reglas

### 1. Linux, no Windows

Los workflows escritos para `ubuntu-latest` usan `bash`, `sudo apt-get`, `services:` y acciones basadas en contenedores. **`services:` y las acciones Docker solo funcionan en runners Linux** [1]. En un equipo Windows, el runner va dentro de **WSL2** (Ubuntu), nunca como runner de Windows.

### 2. Interruptor en el workflow, no `runs-on` fijo

```yaml
runs-on: ${{ vars.RUNNER || 'ubuntu-latest' }}
```

Con la variable de repositorio `RUNNER=<etiqueta-propia>` los jobs van al runner propio; al borrarla vuelven al CI alojado. Nunca se sustituye `ubuntu-latest` por la etiqueta propia a mano en cada job.

### 3. Docker propio del runner, aislado del de desarrollo

El runner no comparte el motor Docker con los contenedores de desarrollo. Un workflow que ejecute `docker compose -p <proyecto> down -v` borraría los contenedores y **volúmenes** locales con el mismo nombre de proyecto. En WSL2 se instala `docker.io` dentro de la distribución del runner en vez de integrar Docker Desktop.

### 4. Puertos de `services:` que no choquen

Todas las distribuciones de WSL2 comparten la red con Docker Desktop: un `services:` que publique `5432:5432` falla con *address already in use* si hay un Postgres de desarrollo encendido. Los servicios de CI usan puertos altos dedicados (`55432:5432`, `56380:6379`…), que funcionan igual en el CI alojado. Se comprueba con un contenedor de prueba antes de cambiar los workflows.

### 5. Un directorio y un registro por repositorio

Un runner de repositorio sirve a un solo repo: cada repo tiene su carpeta (`~/actions-runner/<repo>`), su nombre (`<equipo>-<repo>`) y la misma etiqueta propia. El token de registro se pide con `gh api -X POST repos/<owner>/<repo>/actions/runners/registration-token` y caduca en una hora: no se guarda.

### 6. Nada crítico depende del equipo personal

Un runner en un equipo personal está apagado, dormido o sin red una parte del día. Un job programado que no encuentra runner espera en cola y **falla a las 24 horas** [1]. Por eso:

- Los **respaldos** y cualquier tarea programada crítica tienen su horario primario en el propio servidor (cron); el workflow queda como segunda vía.
- El **despliegue a producción** usa un runner en el servidor, con su propia etiqueta, nunca el del equipo personal.
- Los secretos que usan esos jobs (claves SSH, tokens) pasan por la máquina del runner: se acepta conscientemente o se mueve el job.

### 7. Recursos acotados

El runner compite por memoria con el resto del equipo. En WSL2 se fija un tope en `%USERPROFILE%\.wslconfig` (memoria, swap, procesadores) y `autoMemoryReclaim=gradual` para que la caché vuelva a Windows. El tope aplica a todo WSL: Docker Desktop y la distribución del runner lo comparten.

### 8. Scripts de CI y de operación: fallar ruidosamente

Los scripts que corren en el runner o en el servidor usan `set -euo pipefail` y por eso **el último comando de un pipeline debe consumir toda su entrada**. Un `awk '… { exit }'`, `grep -q` o `head` que termina antes provoca SIGPIPE (código 141) en el escritor; `pipefail` lo convierte en error y `set -e` corta el script **sin mensaje**. Un fallo así depende del orden de los datos y de la versión de las herramientas: se diagnostica con una trampa `ERR` en el entorno real, no reproduciéndolo en otra máquina:

```bash
bash -E -c 'trap "echo \"falló (\$?) en línea \$LINENO: \$BASH_COMMAND\" >&2" ERR; . script.sh'
```

### 9. Mismo contrato que la aplicación

Todo proceso de CI u operación que recorra "todos los X" (bases, shards, tenants, buckets) obtiene la lista **de la misma fuente y con la misma traducción que la aplicación**, no de una lista escrita a mano ni de una convención de nombres propia. Si la aplicación registra recursos dinámicamente, el script los lee del registro y **falla** ante un hueco (un recurso registrado sin configuración) en lugar de saltarlo.

## Operación mínima

| Tarea | Cómo |
|---|---|
| Encender | Un lanzador (`iniciar-runners.cmd`) que abre la distribución y ejecuta `run.sh` de cada repo; ventana visible mientras trabaja |
| Comprobar | `gh api repos/<owner>/<repo>/actions/runners` → `online`; en GitHub, *Settings → Actions → Runners* → *Idle* |
| Volver al CI alojado | `gh variable delete RUNNER -R <owner>/<repo>` |
| Retirar un runner | `./config.sh remove --token <token de gh api …/remove-token>` |
| Actualizar | Automático: el runner se actualiza al recibir un job o en una semana [1] |

Tras `wsl --shutdown` (necesario para aplicar `.wslconfig`), los contenedores de desarrollo sin política de reinicio quedan detenidos: se vuelven a encender con `docker start`.

## Anti-patrones

| Anti-patrón | Consecuencia | Alternativa |
|---|---|---|
| Runner propio en un repositorio público | Código de un fork ejecutándose en la máquina | CI alojado |
| Runner de Windows para workflows escritos para Ubuntu | `services:`, `bash` y acciones Docker fallan | Runner Linux en WSL2 |
| `runs-on` cambiado a mano job por job | Volver atrás exige otro commit en cada workflow | `vars.RUNNER` |
| Runner compartiendo Docker Desktop con desarrollo | Un `down -v` del CI borra los volúmenes locales | Motor Docker propio del runner |
| Respaldo o despliegue que dependen del equipo personal encendido | Días sin respaldo o deploys colgados 24 h | Cron o runner en el servidor |
| `awk … exit` / `grep -q` al final de un pipeline con `pipefail` | El script muere en silencio con 141 | Leer toda la entrada |
| Lista fija de recursos en un script de operación | Recursos nuevos sin respaldo ni limpieza | Leer el registro que usa la aplicación |

## Ejemplo de referencia (equipo personal, 2026-09-24)

Windows 11 (12 GB RAM, 4 núcleos) → WSL2 `Ubuntu-24.04` (usuario `runner` con `sudo` sin contraseña, `docker.io` + `docker-compose-v2` + `docker-buildx` propios, `postgresql-client`, `gh`) → un runner por repositorio privado en `~/actions-runner/<repo>`, etiqueta `local-linux`, variable `RUNNER=local-linux` en cada repo, puertos de `services:` en 55432/55438–55440/56380, `.wslconfig` con `memory=5GB`, `swap=4GB`, `processors=3`. El despliegue a producción conserva su etiqueta propia y el respaldo diario corre por cron en el servidor.

## Relación con Ponytail

El runner propio es una respuesta a una restricción real (minutos), no una mejora por defecto. La solución mínima es un runner persistente por repo y un interruptor de una línea; el autoescalado, los runners efímeros o una granja de runners se añaden solo cuando haya concurrencia o equipo que lo justifiquen. Lo que no se simplifica: el aislamiento de Docker, la exclusión de repos públicos y que lo crítico no dependa del equipo personal.

## Referencias

[1]: https://docs.github.com/en/actions/reference/runners/self-hosted-runners "GitHub Docs — Self-hosted runners reference"
[2]: https://learn.microsoft.com/en-us/windows/wsl/wsl-config "Microsoft Learn — Advanced settings configuration in WSL (.wslconfig)"
[3]: https://docs.github.com/en/actions/learn-github-actions/variables "GitHub Docs — Store information in variables"
