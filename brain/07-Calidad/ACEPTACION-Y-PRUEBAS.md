---
id: aceptacion-pruebas
type: calidad
status: criterios-sin-ejecucion
updated: 2026-10-14
tags: [orbita, pruebas]
---
# Aceptación y pruebas

## Preparación documental

- Enlaces locales y navegación del vault resuelven.
- IDs de notas únicos y metadatos coherentes.
- Estado distingue entregado, propuesto y pendiente.
- No hay secretos, partidas ni contenido privado.
- Los Issues reflejan la fase permitida y las dependencias.

## Demo

| Área | Escenario de aceptación |
| --- | --- |
| Planificación | Cambiar instrumentos modifica restricciones y resultado esperado. |
| Ciclo | Planificar, desplegar, explorar, resolver incidencia, regresar y ver informe. |
| Movimiento | Profundidad, colisiones e interacción funcionan en bordes y obstáculos. |
| Progresión | Repetir una entrega o recargar no duplica recompensas. |
| Guardado | Restaurar un hito conserva estado; importar datos dañados no borra una partida válida. |
| Compañero | Reacciona a un hallazgo y conserva decisiones; sin IA sigue la historia. |
| Red | Se puede completar la región cargada al perder conexión. |
| Idioma y controles | Español e inglés, foco de teclado y texto legible; escribir no mueve al personaje. |
| Ciencia | Datos consultables y simplificaciones reconocibles. |

Vitest comprobará reglas y persistencia relevantes. Playwright comprobará recorridos de usuario y fallos de red. No escribir pruebas que solo repitan la implementación.

## Rendimiento

Objetivos: 60 FPS en el equipo de referencia, opción de 30 FPS; carga inicial comprimida menor a 15 MB; memoria de pestaña menor a 500 MB.

Registrar navegador, resolución, GPU utilizada, escena, duración y resultado. Medir recorrido normal y escena exigente; no presentar una captura de un contador como prueba sostenida.

## Campaña

Jugadores nuevos deben probar orientación, legibilidad, progresión, repetición, dificultad y duración. Ajustar cuotas de contenido con evidencia.

**Resultado actual:** ninguna prueba del juego ejecutada; todavía no existe implementación.
