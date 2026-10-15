---
id: viaje-espacial
type: diseno
status: especificado-sin-implementar
updated: 2026-10-14
tags: [orbita, vuelo, navegacion]
---
# Viaje espacial y pilotaje 2.5D

El propietario eligió **pilotaje directo en maniobras y aproximaciones, con trayectos largos acelerados**.

## Ciclo de un trayecto

Planificar ventana y reservas → desacoplar → ejecutar maniobra → crucero acelerado → corrección o incidencia prevista → aproximación → acoplar o descender.

El mapa muestra origen, destino, duración física estimada, combustible, comunicaciones y reservas. Cuando haya alternativas, compara una ruta económica y otra más rápida que exige más recursos. Las cifras pertenecen al contenido de misión.

## Pilotaje local

Vista 2.5D de la nave, con posición y velocidad relativas al destino. El motor modifica la velocidad; soltarlo conserva movimiento en el modelo simplificado. Frenar exige una maniobra o asistencia con consumo.

Controles de nave propuestos, remapeables: W impulsa; S retropropulsa; A/D orientan; Q/E trasladan lateralmente; X solicita igualar velocidad; Mayús activa precisión. Hay acciones equivalentes en UI y soporte de mando.

En superficie: WASD mueve, E interactúa, R escanea, M abre mapa y Tab abre diario. El modo actualiza las indicaciones; los campos de texto capturan el teclado.

La aproximación se completa dentro del corredor y con alineación y velocidad permitidas, visibles en pantalla. Las tolerancias son parámetros de misión.

## Crucero y tiempo

Se distinguen tiempo físico y tiempo de juego. Días o meses se resuelven con saltos y decisiones, sin esperar ante la pantalla.

Se contabilizan suministros y desgaste del tramo planificado. Soporte de vida y reservas corresponden al itinerario ficticio. Menús y conversaciones pausados no avanzan riesgos.

Las incidencias aparecen en hitos definidos; no dependen de conectarse a una hora real. Un tramo puede abrir un descanso con LUMA.

No se acelera en acoplamientos, cerca de superficie o durante peligros locales. Se guarda antes de maniobras importantes y al cambiar de tramo.

## Modelos y límites

Dinámica local simplificada para maniobras; tramos parametrizados para crucero. No se simula el Sistema Solar completo en una sola escena ni una física de n cuerpos.

Las trayectorias representadas no se ofrecen como cálculo operativo real. Referencias: [JPL Horizons](https://ssd-api.jpl.nasa.gov/doc/horizons.html) y [NASA, gravedad y mecánica](https://science.nasa.gov/learn/basics-of-space-flight/chapter3-2/).

## Encuentros y recuperación

Satélites, señales y observaciones tienen un propósito de misión. No se rellenan trayectos con campos densos de rocas aleatorias.

Se impiden planes sin reservas mínimas. Un accidente permite volver al punto de control o abortar a zona segura; el informe explica la causa.

La demo incluirá una aproximación corta con frenado y asistencia, más un tramo acelerado. La campaña amplía acoplamientos, inserciones, ascensos y contacto anclado.
