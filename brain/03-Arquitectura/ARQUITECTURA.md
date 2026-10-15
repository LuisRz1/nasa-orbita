---
id: arquitectura
type: arquitectura
status: acordado
updated: 2026-10-14
tags: [orbita, tecnica]
---
# Arquitectura modular

~~~mermaid
flowchart LR
  Input[Controles] --> Core[Simulación TypeScript]
  UI[Interfaz React] <--> Core
  Content[Contenido validado] --> Core
  Core --> View[Escenas Phaser]
  Core <--> Save[IndexedDB / Dexie]
  UI --> API[API opcional de LUMA]
  API --> Gateway[AI Gateway]
~~~

## Límites

- **Simulación:** recibe comandos y tiempo; produce estado y eventos. Sin dependencias de React, Phaser o red.
- **Presentación:** representa el estado y reproduce animaciones. No concede progreso por terminar una animación.
- **Interfaz:** muestra información y envía comandos. No mantiene una segunda copia independiente de las reglas.
- **Contenido:** definiciones validadas con IDs y referencias comprobables.
- **Persistencia:** serializa estado estable, valida versiones y restaura.
- **API:** genera conversación opcional; nunca es necesaria para completar una misión.

La simulación usa un paso fijo, desacoplado del frame gráfico. Renderizado isométrico y coordenadas lógicas de movimiento se mantienen separados. Colisiones y altura se calculan en el espacio lógico.

Vuelo local, crucero y superficie tienen estados y marcos separados. El crucero avanza por tramos, sin integrar días de física local. Ver [ADR 004](ADR/ADR-004.md).

El director de campaña verifica objetivos, pruebas de Aurora, manifiesto y cierre; el director narrativo aplica revelaciones y los hitos escritos de LUMA. La generación procedural y la IA no los sustituyen. Las vistas de conocimiento usan únicamente evidencia ya validada para evitar adelantar destinos o acontecimientos.

Zustand expone el estado de interfaz y lecturas derivadas; no se propaga un render React por cada objeto o frame de Phaser.

## Recursos y publicación

Atlases, audio y regiones se cargan según necesidad. El build cliente es estático; la función de LUMA se despliega por separado dentro del proyecto Vercel.

No habrá cuentas, base de datos remota, sincronización ni microservicios en la primera versión. La API usa credenciales del servidor, nunca secretos empaquetados en el navegador.

Decisiones: [motor](ADR/ADR-001.md), [juego local e IA](ADR/ADR-002.md), [ventana de implementación](ADR/ADR-003.md).
