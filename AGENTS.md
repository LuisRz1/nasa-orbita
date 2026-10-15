# Instrucciones para agentes y colaboradores

## Estado y autorización

Este repositorio contiene investigación y planificación de **Órbita: Cartógrafos del futuro**. El propietario eligió conservar la elegibilidad de NASA Space Apps 2026.

**No implementar el juego, producir assets del reto, descargar/procesar datasets para la solución ni desplegar una versión jugable antes del inicio autorizado de la hackathon.** Tampoco usar ramas auxiliares para adelantar ese trabajo. Esta restricción deriva de la decisión expresa del propietario y de la FAQ enlazada en [requisitos NASA](brain/04-Ciencia/REQUISITOS-NASA.md).

La fecha global es 14–15 de noviembre de 2026. El horario y sede exactos aún deben verificarse. La fecha del sistema por sí sola no acredita que la ventana esté abierta. La documentación de preparación está autorizada; no presentar esta interpretación como aprobación del organizador.

## Lectura de entrada

1. [Estado actual](brain/00-Inicio/ESTADO-ACTUAL.md).
2. [Decisiones del propietario](brain/00-Inicio/DECISIONES.md).
3. [Roadmap](brain/06-Produccion/ROADMAP.md) y [backlog](brain/06-Produccion/BACKLOG.md).
4. Especificación y ADR del módulo que se vaya a tocar.
5. Issue asignado, sus dependencias y aceptación.

## Cómo trabajar

- Buscar primero en el repositorio. No volver a preguntar decisiones ya registradas.
- Conservar los cambios de otros colaboradores. Un estado de Git inesperado requiere inspección, no limpieza automática.
- Documentación en español; identificadores técnicos en inglés.
- Cada tarea tiene un responsable operativo. No asumir que una persona o agente está asignado por aparecer en un reparto propuesto.
- No iniciar delegación o agentes paralelos sin autorización expresa del usuario o instrucciones aplicables que la pidan.
- Cambios de motor, persistencia, alcance, costes o contratos requieren actualizar el ADR y la especificación afectados.
- Las versiones y disponibilidad de servicios se verifican antes de instalar. No instalar herramientas globales si basta una dependencia local o portable.
- No contratar servicios, comprar créditos, activar recargas ni añadir cobros. La IA online es opcional y el presupuesto autorizado de pagos es cero.
- No publicar claves, tokens, partidas personales, conversaciones completas ni configuraciones privadas en este repositorio público.
- Nunca dar a la IA autoridad para modificar progreso, recursos o recompensas.
- Los datos científicos deben enlazar una fuente y distinguir observación, simplificación y ficción.

## Verificación y cierre

En esta fase, comprobar enlaces locales, referencias Obsidian, coherencia del estado y ausencia de secretos. No inventar resultados de ejecución: todavía no hay un juego.

Cuando se autorice implementación, ejecutar las comprobaciones relevantes descritas en [calidad](brain/07-Calidad/ACEPTACION-Y-PRUEBAS.md). Un mock o una captura no acredita una función completa.

Antes de finalizar una tarea:

1. Actualizar el Issue operativo y la documentación afectada.
2. Actualizar ESTADO-ACTUAL con evidencia y siguiente paso.
3. Añadir una entrada a la bitácora.
4. Dejar un [handoff](brain/09-Plantillas/HANDOFF.md) si el trabajo continúa.
5. Indicar qué se verificó y qué sigue sin comprobar.

## Fuentes de verdad

| Información | Fuente |
| --- | --- |
| Preferencias y alcance acordados | DECISIONES y PLAN |
| Trabajo operativo y asignación | GitHub Issues |
| Estado comprobado y punto de continuación | ESTADO-ACTUAL |
| Justificación técnica | ADR |
| Historial real de cambios | Git y bitácora |

No mantener estados contradictorios en varias listas. El backlog documental enlaza los Issues y conserva el contexto.
