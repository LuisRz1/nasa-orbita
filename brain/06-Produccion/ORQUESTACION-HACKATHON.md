---
id: orquestacion-hackathon
type: produccion
status: plan-sin-ejecutar
updated: 2026-10-19
tags: [orbita, hackathon, agentes, orquestacion]
---
# Orquestación de la hackathon

Plan para construir la demo greybox del [GDD](../02-Diseno/DEMO/GDD-DEMO.md) con dos personas y varios agentes **durante la ventana autorizada**. Hoy no se ejecuta nada de este plan ([ADR 003](../03-Arquitectura/ADR/ADR-003.md)).

Las horas son relativas al inicio autorizado (H+0) porque el horario de la sede de Cajamarca sigue pendiente ([PREP-04](BACKLOG.md)). Si la ventana real es más corta, se aplican antes los recortes de la última sección.

## Condiciones para empezar (H+0)

1. Registrar en [ESTADO-ACTUAL](../00-Inicio/ESTADO-ACTUAL.md) la evidencia que exige ADR 003: fuente, sede, zona horaria, inicio y cierre.
2. Revisar el enunciado completo (PREP-03) y aplicar los cambios a la [matriz](../04-Ciencia/REQUISITOS-NASA.md) y a [parámetros](../02-Diseno/DEMO/PARAMETROS-DEMO.md).
3. El usuario autoriza de forma expresa el arranque de agentes en esa sesión. La autorización documental de octubre no basta.

## Roles

| Rol | Quién | Responsabilidad |
| --- | --- | --- |
| Integrador humano | Propietario | Aprueba contratos, revisa y fusiona, decide recortes y prueba cada puerta. |
| Diseño y presentación | Segunda persona | Prueba de juego, ajuste de nivel y textos, guion del pitch, vídeo y paquete de entrega. Dirige el arte cuando se decida. |
| Orquestador | Sesión principal de Claude | Contratos compartidos, integración, reparto, revisión de coherencia con las esquemáticas. |
| Sim-Core | Agente | Bucle de paso fijo, estado, comandos y eventos, planificador y validación, recursos, informe y persistencia. |
| Vuelo | Agente | Crucero y aproximación (simulación y escenas). |
| Superficie | Agente | Proyección, movimiento, colisiones, sombra, mapa, interacción, incidencia, soporte vital, seguimiento de LUMA y escena. |
| UI | Agente | React: menú, briefing, planificador, HUDs, diálogo, diario, informe, ajustes y foco. |
| Contenido e i18n | Agente | JSON de módulos, tanques, POI, mapa y reacciones; textos ES/EN; fuentes. |
| QA | Agente | Pruebas e2e, CI, medición de rendimiento y recorrido del jurado. |

La propiedad exacta de carpetas está en [estructura de código](../03-Arquitectura/ESTRUCTURA-CODIGO.md). Los archivos de contrato (tipos, eventos, comandos y esquemas) solo los edita el Orquestador. Un agente que necesite un cambio de contrato lo pide; no lo hace él mismo.

Como máximo cinco agentes trabajan a la vez, cada uno en su propio worktree y rama `jam/<rol>`. Se fusiona a `main` mediante el Orquestador.

## Olas y puertas

| Ola | Horas | Trabajo | Puerta de salida |
| --- | --- | --- | --- |
| 0 · Base | H+0 → H+2 | Orquestador: verificar versiones vigentes, crear proyecto, fijar lockfile, contratos de tipos/eventos/comandos, parámetros como contenido, CI mínimo y preview en Vercel. | **G0:** build, typecheck y tests en verde; la URL de preview carga una escena vacía. |
| 1 · Esqueleto jugable | H+2 → H+8 | En paralelo: Sim-Core (bucle, planificador, V/W), Vuelo (crucero y aproximación), Superficie (proyección, movimiento, colisión, mapa ASCII → celdas, POI), UI (planificador y HUDs), Contenido (datos e i18n). | **G1:** recorrido de punta a punta en greybox: planificar → crucero → aproximar → superficie → informe, aunque sea con textos provisionales. |
| 2 · Sistemas completos | H+8 → H+18 | Incidencia del relé, soporte vital y emergencia, regla de sombra, asistente de vuelo, reacciones de LUMA, informe causal, guardado y puntos de control. QA escribe los e2e. | **G2:** cada fila de demo de [aceptación](../07-Calidad/ACEPTACION-Y-PRUEBAS.md) tiene prueba y pasa. |
| 3 · Pulido | H+18 → H+24 | Accesibilidad, ES/EN completos, medición de rendimiento, bug bash con la segunda persona y ajuste de parámetros con evidencia. | **G3 (congelación):** sin errores bloqueantes; solo correcciones desde aquí. |
| 4 · Entrega | H+24 → H+28 | Despliegue de producción, recorrido desde la URL pública, vídeo y pitch, paquete de [entrega](../07-Calidad/DEMO-Y-ENTREGA.md). | **G4:** enlace público verificado en un navegador limpio y entrega enviada antes del cierre. |
| Margen | H+28 → cierre | Imprevistos. No se añaden funciones. | — |

Una puerta no pasa con mocks ni capturas. Exige un recorrido real y pruebas en verde.

## Correspondencia con los Issues

| Issue | Agentes |
| --- | --- |
| JAM-01 base técnica | Orquestador (ola 0) |
| JAM-02 movimiento isométrico e interacción | Superficie |
| JAM-03 planificador, recursos e informe | Sim-Core + UI |
| JAM-04 región lunar y seis lugares | Superficie + Contenido |
| JAM-05 LUMA escrito | Contenido + Superficie |
| JAM-06 guardado, localización y accesibilidad | Sim-Core + Contenido + UI |
| JAM-07 QA y publicación | QA + Orquestador |
| JAM-08 pilotaje y crucero | Vuelo |
| OPT-01 chat de IA | Solo tras G3, con acceso gratuito verificado y pagos cero |

Ver [backlog](BACKLOG.md).

## Protocolo de fusión

1. El agente sincroniza su rama con `main` y ejecuta typecheck, lint, Vitest y build en su worktree.
2. Entrega un handoff breve: qué cambió, qué pruebas pasan, qué falta y qué IDs de prueba cubre.
3. El Orquestador revisa contra la esquemática, sin aceptar reglas inventadas, y fusiona en este orden: Sim-Core → Contenido → Vuelo y Superficie → UI → QA.
4. Tras cada fusión, `main` debe quedar en verde. Si se rompe, se revierte la fusión antes de continuar.

## Plantilla de prompt para cada agente

Se copia y se completa en la sesión de la hackathon:

> Eres el agente **{ROL}** de Órbita. La ventana autorizada está abierta (ver ESTADO-ACTUAL). Trabaja solo en tu worktree y rama `jam/{rol}`.
> **Lee:** AGENTS.md, GDD-DEMO, PARAMETROS-DEMO, {ESQUEMÁTICAS DEL ROL} y ESTRUCTURA-CODIGO (propiedad de carpetas y contratos).
> **Posees:** {CARPETAS}. **No toques:** contratos compartidos ni carpetas ajenas; si necesitas un cambio de contrato, pídelo.
> **Implementa:** {LISTA}. Las reglas y cifras salen de las esquemáticas y de PARAMETROS; si algo no está definido, pregunta en lugar de inventarlo.
> **Pruebas obligatorias:** {IDs T-MEC/T-FIS/T-NIV/T-E2E}.
> **Hecho cuando:** pruebas en verde, build en verde, recorrido manual de tu parte descrito y handoff entregado.
> Sin claves ni servicios de pago. La IA no modifica el progreso.

| Rol | Esquemáticas | Implementa (resumen) |
| --- | --- | --- |
| Sim-Core | [Mecánicas](../02-Diseno/DEMO/MECANICAS.md), [Físicas §bucle](../02-Diseno/DEMO/FISICAS.md), [guardado](../03-Arquitectura/GUARDADO.md) | Bucle, estado, bus, planificador, V1–V6/W1–W5, recursos, informe y persistencia. |
| Vuelo | [Físicas](../02-Diseno/DEMO/FISICAS.md), [viaje espacial](../02-Diseno/VIAJE-ESPACIAL.md) | Crucero con corrección; aproximación, asistente, condiciones, fallos y reintento; escenas greybox. |
| Superficie | [Físicas](../02-Diseno/DEMO/FISICAS.md), [diseño de nivel](../02-Diseno/DEMO/DISENO-DE-NIVEL.md), [mecánicas](../02-Diseno/DEMO/MECANICAS.md) | Proyección, movimiento, colisiones, sombra, POI, incidencia, soporte vital, LUMA y escena. |
| UI | [Mecánicas](../02-Diseno/DEMO/MECANICAS.md), [greybox](../05-Arte/GREYBOX.md) | Pantallas y HUDs, foco de teclado y lectura de estado vía store. |
| Contenido e i18n | [Mecánicas](../02-Diseno/DEMO/MECANICAS.md), [diseño de nivel](../02-Diseno/DEMO/DISENO-DE-NIVEL.md), [fuentes](../04-Ciencia/FUENTES.md) | Datos validados por esquema, textos ES/EN y fuentes con su nivel O/S/F. |
| QA | [Aceptación](../07-Calidad/ACEPTACION-Y-PRUEBAS.md), [estructura de código](../03-Arquitectura/ESTRUCTURA-CODIGO.md) | e2e T-E2E-*, CI, protocolo de rendimiento y guion de recorrido del jurado. |

## Recortes de alcance, en orden

Según el [roadmap](ROADMAP.md): se conservan planificación, operación, una decisión técnica, regreso e informe.

1. OPT-01 (chat de IA) queda fuera.
2. Escena de LUMA en el crucero: solo una línea de texto.
3. Alineación manual del relé: una acción de un botón con la misma duración.
4. Zoom y transparencia de oclusores: una sola regla simple.
5. Inglés: interfaz y textos esenciales primero; textos secundarios después de la demo.
6. Exportar e importar: queda el guardado automático local; la exportación se aplaza y se declara como limitación.

No se recortan: la validación causal, las dos soluciones de energía, la elección de ruta, el informe ni las fuentes.

## Riesgos de la orquestación

| Riesgo | Mitigación |
| --- | --- |
| Agentes que inventan reglas o cifras | Solo las esquemáticas y PARAMETROS mandan. Revisión del Orquestador antes de fusionar. |
| Conflictos de fusión | Carpetas disjuntas, contratos centralizados y fusiones pequeñas y frecuentes. |
| Integración tardía | La puerta G1 exige el recorrido completo a H+8. |
| Demo que funciona solo en local | Preview desde G0 y verificación en la URL pública en G4. |
| Cansancio del equipo | Turnos de descanso planificados: los agentes avanzan en tareas acotadas mientras una persona descansa, nunca fusiones sin revisión. |
