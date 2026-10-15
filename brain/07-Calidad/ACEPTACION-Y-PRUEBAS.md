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
| Pilotaje | Frenar respecto al objetivo, acoplar y reintentar con asistencia visible. |
| Crucero | Acelerar, pausar y reanudar sin duplicar consumo de recursos. |
| Historia local | Recuperar un registro parcial de Aurora, reparar el enlace y entregarlo cierra la demo; no declara el rescate global ni completa el capítulo. |

Vitest comprobará reglas y persistencia relevantes. Playwright comprobará recorridos de usuario y fallos de red. No escribir pruebas que solo repitan la implementación.

## Rendimiento

Objetivos: 60 FPS en el equipo de referencia, opción de 30 FPS; carga inicial comprimida menor a 15 MB; memoria de pestaña menor a 500 MB.

Registrar navegador, resolución, GPU utilizada, escena, duración y resultado. Medir recorrido normal y escena exigente; no presentar una captura de un contador como prueba sostenida.

## Campaña

Jugadores nuevos deben probar orientación, legibilidad, progresión, repetición, dificultad y duración. Ajustar cuotas de contenido con evidencia.

Comprobar las 24 principales y sus dependencias, las cuatro condiciones de cierre y la libertad de orden en Luna/Marte. No exigir opcionales para pasar.

Verificar cuatro paletas de LUMA con señales accesibles, traducción escrita sin red, ausencia de spoilers en respuestas y persistencia de sus acontecimientos. La exploración posterior no depende de su presencia o chat.

### Narrativa y rescate — spoilers

- Las 24 principales tienen acción, condición de cierre y aporte a una pista, ruta o capacidad de rescate.
- La cadena AUR-E01–AUR-E07 coincide con la [historia principal](../02-Diseno/HISTORIA-PRINCIPAL.md). Una pieza recogida sin autenticar no abre la revelación siguiente.
- Probar las seis permutaciones de C2-M03/M04/M05 y ambos órdenes de C3-M04/M05. El informe final revela los mismos hechos; antes no aparecen conclusiones futuras en diario, diálogos o IA.
- La cápsula prueba un envío de supervivientes en el pasado; el mensaje marciano conserva fecha; la presencia viva se confirma al llegar. El juego distingue esos niveles de certeza.
- C3-M06 y C4-M01 rechazan una carga sin plazas, soporte de vida o reservas suficientes para los supervivientes y quienes ya viajan. La logística sale de Marte y regresa a Tierra después de la extracción.
- C4-M03 verifica a la hermana y al manifiesto completo; M04 no cierra con alguien fuera de la nave. Una lista vacía no acredita éxito.
- C4-M05 salva al protagonista por el sacrificio de LUMA, con los rescatados ya seguros. La despedida escrita usa recuerdos existentes y ocurre una vez.
- C4-M06 exige recibir a todas las personas del manifiesto en destino terrestre. Entregar muestras por sí solo no completa la campaña.
- Esperar en menús o explorar secundarias no mata a LUMA o a Aurora. Los riesgos ordinarios permiten recuperación y no sustituyen el desenlace.
- Recargar en los límites de evacuación, despedida y recepción conserva el estado. El epílogo reconoce rescate y pérdida, y mantiene los pendientes mediante nuevas expediciones.

**Resultado actual:** ninguna prueba del juego ejecutada; todavía no existe implementación.
