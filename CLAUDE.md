# Adaptador de contexto para Claude Code

Antes de actuar, carga y cumple `AI_INSTRUCTIONS.md`. Es el contrato canónico de ATLAS y aplica igual con `claude`, `claude-max` o `claude-deepseek`.

- **Simplicidad:** la skill Ponytail está activa en nivel `full` (`02-diseno-y-codigo/PONYTAIL_SIMPLICITY.md`). Comprender primero, luego la solución mínima; los controles de seguridad, datos y APIs nunca se simplifican.
- **APIs:** todo borde HTTP sigue `01-arquitectura/API_GATEWAY.md`.
- **Contexto del proyecto:** si existe `PROJECT_CONTEXT.md`, sus decisiones prevalecen sobre las preferencias generales.
- **Entrega:** Definition of Done de `04-proceso/DELIVERY_WORKFLOW.md`, con `/ponytail-review` sobre el diff antes de cerrar.

Al copiar ATLAS a otro proyecto, copia también este archivo a la raíz.
