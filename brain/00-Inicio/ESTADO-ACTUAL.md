---
id: estado-actual
type: estado
status: preparacion-documental
updated: 2026-10-19
tags: [orbita, continuidad]
---
# Estado actual

## Punto de continuación

**Fase activa: preparación documental. No hay juego implementado.**

El propietario pidió publicar el plan y el cerebro para compartirlos. Esta entrega deja una base de documentación, decisiones y tareas; no acredita movimiento, misiones, guardado, IA o rendimiento.

La revisión 1.2 integra la historia de Aurora en los cuatro capítulos y 24 misiones: buscar a la hermana del protagonista y a los supervivientes y traerlos a la Tierra. LUMA llega en la cápsula de emergencia, se deteriora y se sacrifica para salvar al jugador. El título de trabajo es Órbita: La última luz. Son especificaciones, no contenido jugable.

Leer [historia principal — spoilers](../02-Diseno/HISTORIA-PRINCIPAL.md), [campaña y misiones](../02-Diseno/MUNDO-Y-CAMPANA.md) y [handoff de esta revisión](../08-Bitacora/HANDOFF-2026-10-14.md). La ruta Tierra → Luna → Marte → Bennu → Tierra reabastece en Marte; sustituye la escala terrestre intermedia anterior.

**Revisión 1.3 (19 de octubre): esquemáticas de la demo.** El usuario pidió empezar a programar. Ante la restricción de la FAQ, reafirmó que se conserva la elegibilidad. Se escribieron las esquemáticas completas de la demo greybox (gris y blanco), coordinadas por un orquestador y cuatro agentes:

- [GDD](../02-Diseno/DEMO/GDD-DEMO.md), con la máquina de estados;
- [parámetros](../02-Diseno/DEMO/PARAMETROS-DEMO.md), provisionales y calculados;
- [mecánicas](../02-Diseno/DEMO/MECANICAS.md), [físicas](../02-Diseno/DEMO/FISICAS.md) y [diseño de nivel](../02-Diseno/DEMO/DISENO-DE-NIVEL.md);
- [greybox](../05-Arte/GREYBOX.md), [estructura de código](../03-Arquitectura/ESTRUCTURA-CODIGO.md) y [orquestación de la hackathon](../06-Produccion/ORQUESTACION-HACKATHON.md).

Siguen siendo especificaciones: no hay código, dependencias ni assets. Ver el [handoff del 19 de octubre](../08-Bitacora/HANDOFF-2026-10-19.md).

## Entregado

- Plan completo y README de entrada.
- Vault compatible con Obsidian y GitHub, sin plugins obligatorios.
- Especificaciones de producto, diseño, arquitectura, ciencia, arte y calidad.
- Cuatro fichas de capítulo con acciones, objetivos, desbloqueos y condiciones de cierre.
- Historia principal con motivo del viaje, cadena de pruebas AUR-E01–AUR-E07, reencuentro, evacuación y epílogo.
- Las 24 principales especifican su aporte al rescate. Contratos, guardado y QA contemplan revelaciones y manifiesto de supervivientes.
- Viaje espacial y arco de LUMA separado con spoilers.
- Decisiones del propietario y ADR.
- Instrucciones para agentes, plantillas y secuencia de tareas.
- [Repositorio público](https://github.com/LuisRz1/nasa-orbita), con la rama main publicada.
- [18 Issues](https://github.com/LuisRz1/nasa-orbita/issues) con dependencias, aceptación y etiquetas por fase.

## Evidencia y límites

- Carpeta de partida vacía y sin Git, comprobado el 14 de octubre.
- Cuenta de publicación confirmada: LuisRz1.
- FAQ oficial revisada el 14 de octubre: fechas globales y restricción de trabajo previo siguen publicadas.
- No hay dependencias del juego, assets, datasets procesados, pruebas de ejecución ni demo.
- Las versiones técnicas son una base investigada; no existe todavía un lockfile.
- Publicación inicial confirmada en el commit ec0edf1; consultar Git para las actualizaciones posteriores.
- Revisión documental inicial: 45 archivos, 40 documentos Markdown, 32 notas y 63 enlaces válidos.
- Revisión 1.1: 52 archivos previstos para versionar, 47 documentos Markdown, 39 notas y 94 enlaces locales comprobados. Se verificaron seis misiones por capítulo, 18 regiones únicas y los totales 24/32/18/120.
- Revisión 1.2: 54 archivos para versionar, 49 documentos Markdown y 41 notas con IDs únicos. 140 enlaces locales, 24 principales, 18 regiones, siete evidencias y cuotas 24/32/18/120 comprobados. Revisión de patrones de secretos y formato sin incidencias.
- Diez Issues actualizados y leídos de nuevo para comprobar su contenido: #7–#11, #13 y #15–#18. Conservan estado abierto, dependencias y etiquetas de fase; hay 18 Issues en total.
- Calendario narrativo, tiempos de cápsula y autonomía del refugio pendientes de validar junto con las ventanas de vuelo. La historia propuesta no acredita una trayectoria real viable.
- El estado de publicación y los commits se comprueban en el historial y en la bitácora de esta entrega.
- **Revisión 1.3, comprobaciones del 19 de octubre:**
  - FAQ revisada de nuevo: la restricción de trabajo previo sigue publicada.
  - Fuentes verificadas: gravedad lunar de 1,62 m/s² (NSSDC), tránsito de Apollo 11 en unas 75 h 50 min y ecuación del cohete (NASA Glenn).
  - Documentación: 59 documentos Markdown, 49 notas con IDs únicos y 485 enlaces locales, sin rotos.
  - Parámetros: las configuraciones de referencia se recalcularon con scripts fuera del repositorio y coinciden en mecánicas y físicas.
  - Mapa ASCII de 56 × 44 comprobado con un camino más corto: todas las rutas dentro de ±10 %; sin lámpara no hay atajo iluminado.
  - Sin bloques de código TS/JS, sin package.json ni src/, sin patrones de secretos; `git diff --check` limpio.
- **Sin comprobar:** el balance es provisional y no se ha probado con jugadores. Riesgo abierto: con desvíos típicos, el soporte vital queda justo (ver parámetros). Tampoco hay rendimiento medido ni pruebas ejecutadas.

## Próximo trabajo permitido

1. Revisar con el equipo plan, historia, fichas de misión y las esquemáticas de demo; asignar responsables reales mediante PREP-01. Se puede jugar «en papel» el planificador con la tabla de parámetros para discutir el balance. El handoff enumera los detalles editoriales y de balance pendientes para sus fases autorizadas.
2. Completar la investigación donde la evidencia aún sea histórica o parcial.
3. El 28 de octubre, revisar el enunciado completo y actualizar la matriz de requisitos y, si hace falta, el GDD y los parámetros de la demo.
4. El 13 de noviembre, registrar guías y horario exacto de la sede.
5. Abrir implementación únicamente al inicio autorizado, siguiendo la [orquestación de la hackathon](../06-Produccion/ORQUESTACION-HACKATHON.md).

Consulta [backlog](../06-Produccion/BACKLOG.md) para IDs, dependencias y enlaces a Issues.

## Preguntas externas pendientes

- Sede y horas exactas de inicio y entrega.
- Requisitos y datasets definitivos del enunciado completo.
- Nombres y disponibilidad de los integrantes.
- Acceso gratuito real al modelo de conversación en la fecha de entrega.

Estas incógnitas tienen tareas de revisión; no justifican adelantar desarrollo.
