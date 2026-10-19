---
id: backlog
type: produccion
status: inicial
updated: 2026-10-19
tags: [orbita, tareas]
---
# Backlog y dependencias

Los [GitHub Issues](https://github.com/LuisRz1/nasa-orbita/issues) son la fuente del estado operativo, asignación y conversación. Esta nota conserva IDs, orden y especificaciones; no duplica una lista de progreso.

| ID | Resultado | Fase | Depende de |
| --- | --- | --- | --- |
| [PREP-01 · #1](https://github.com/LuisRz1/nasa-orbita/issues/1) | Revisar plan y confirmar responsables. | Preparación | Publicación documental. |
| [PREP-02 · #2](https://github.com/LuisRz1/nasa-orbita/issues/2) | Completar y contrastar investigación de referentes. | Preparación | Dossier inicial. |
| [PREP-03 · #3](https://github.com/LuisRz1/nasa-orbita/issues/3) | Revisar enunciado completo y matriz. | Preparación, desde 28/10 | Publicación oficial. |
| [PREP-04 · #4](https://github.com/LuisRz1/nasa-orbita/issues/4) | Registrar guías, sede y ventana autorizada. | Preparación, 13/11 | PREP-03, guías y horario de la sede. |
| [JAM-01 · #5](https://github.com/LuisRz1/nasa-orbita/issues/5) | Base técnica y escena mínima. | Hackathon | PREP-03, PREP-04 y apertura autorizada. |
| [JAM-02 · #6](https://github.com/LuisRz1/nasa-orbita/issues/6) | Movimiento isométrico e interacción. | Hackathon | JAM-01. |
| [JAM-03 · #7](https://github.com/LuisRz1/nasa-orbita/issues/7) | Planificador, recursos e informe. | Hackathon | JAM-01. |
| [JAM-04 · #8](https://github.com/LuisRz1/nasa-orbita/issues/8) | Región lunar, seis lugares y registro parcial de Aurora. | Hackathon | JAM-02, JAM-03. |
| [JAM-05 · #9](https://github.com/LuisRz1/nasa-orbita/issues/9) | LUMA escrito y contexto de misión. | Hackathon | JAM-02, JAM-03. |
| [JAM-06 · #10](https://github.com/LuisRz1/nasa-orbita/issues/10) | Guardado, localización y accesibilidad. | Hackathon | JAM-03, JAM-04, JAM-05. |
| [JAM-07 · #11](https://github.com/LuisRz1/nasa-orbita/issues/11) | QA, publicación y recorrido de jurado. | Hackathon | JAM-06 y JAM-08. |
| [JAM-08 · #14](https://github.com/LuisRz1/nasa-orbita/issues/14) | Pilotaje 2.5D y crucero acelerado de demo. | Hackathon | JAM-01 y JAM-03. |
| [OPT-01 · #12](https://github.com/LuisRz1/nasa-orbita/issues/12) | Conversación libre gratuita opcional. | Hackathon o posterior | JAM-07 y acceso gratuito verificado. |
| [EXP-01 · #13](https://github.com/LuisRz1/nasa-orbita/issues/13) | Producción de campaña por capítulos. | Posterior | Demo aceptada y revisión del equipo. |
| [EXP-C1 · #15](https://github.com/LuisRz1/nasa-orbita/issues/15) | Primera luz: cápsula, LUMA y pista lunar; seis misiones y tres regiones. | Posterior | JAM-07. |
| [EXP-C2 · #16](https://github.com/LuisRz1/nasa-orbita/issues/16) | Bajo otro cielo: reconstruir registro; seis misiones y cinco regiones. | Posterior | JAM-07. |
| [EXP-C3 · #17](https://github.com/LuisRz1/nasa-orbita/issues/17) | El mundo rojo: desvío, supervivientes y plan de rescate; seis misiones y seis regiones. | Posterior | EXP-C1 y EXP-C2. |
| [EXP-C4 · #18](https://github.com/LuisRz1/nasa-orbita/issues/18) | Lo que queda: evacuación, sacrificio de LUMA y regreso; seis misiones, cuatro regiones y epílogo. | Posterior | EXP-C3. |

## Esquemática por tarea de hackathon

| ID | Esquemáticas que la definen |
| --- | --- |
| JAM-01 | [Estructura de código](../03-Arquitectura/ESTRUCTURA-CODIGO.md), [orquestación, ola 0](ORQUESTACION-HACKATHON.md) |
| JAM-02 | [Físicas](../02-Diseno/DEMO/FISICAS.md), [diseño de nivel](../02-Diseno/DEMO/DISENO-DE-NIVEL.md) |
| JAM-03 | [Mecánicas](../02-Diseno/DEMO/MECANICAS.md), [parámetros](../02-Diseno/DEMO/PARAMETROS-DEMO.md) |
| JAM-04 | [Diseño de nivel](../02-Diseno/DEMO/DISENO-DE-NIVEL.md), [greybox](../05-Arte/GREYBOX.md) |
| JAM-05 | [Mecánicas: LUMA](../02-Diseno/DEMO/MECANICAS.md) |
| JAM-06 | [Estructura de código](../03-Arquitectura/ESTRUCTURA-CODIGO.md), [guardado](../03-Arquitectura/GUARDADO.md) |
| JAM-07 | [Estructura de código: matriz de pruebas](../03-Arquitectura/ESTRUCTURA-CODIGO.md), [aceptación](../07-Calidad/ACEPTACION-Y-PRUEBAS.md) |
| JAM-08 | [Físicas](../02-Diseno/DEMO/FISICAS.md), [mecánicas: crucero](../02-Diseno/DEMO/MECANICAS.md) |

Índice: [GDD de la demo](../02-Diseno/DEMO/GDD-DEMO.md). Las esquemáticas no cierran ningún Issue de software.

## Enlaces operativos

Los 18 Issues están publicados con dependencias y aceptación. Los responsables no se han asignado todavía. Ninguna tarea de implementación puede empezar antes de la apertura autorizada, aunque su Issue ya exista.

EXP-01 es la tarea general de campaña; sus cuatro tickets de capítulo describen el trabajo de producción. Las fichas de capítulo son especificaciones y no cierran esos tickets.

La revisión 1.2 enlaza el trabajo con la [historia de Aurora — spoilers](../02-Diseno/HISTORIA-PRINCIPAL.md): pruebas AUR-E01–AUR-E07, rescate de la hermana, manifiesto de evacuación y continuidad de LUMA. La demo entrega una parte del registro y conserva los límites de su fragmento lunar. Las tareas de software siguen abiertas hasta implementarse y verificarse.

Etiquetas: fase:preparacion, fase:hackathon, fase:expansion, bloqueado:ventana y opcional. La etiqueta de bloqueo temporal se retira solo después de registrar la apertura y cumplir las dependencias.

## Cierre

Un Issue cerrado requiere sus criterios de aceptación y evidencia. Las tareas de documentación no cierran tareas de software. La responsabilidad propuesta no equivale a asignación confirmada.
