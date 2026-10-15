---
id: contratos
type: especificacion
status: diseno-no-codigo
updated: 2026-10-14
tags: [orbita, interfaces]
---
# Contratos de contenido y servicios

Este documento describe responsabilidades. No es un esquema ejecutable ni fija todos los campos de la futura implementación.

| Contrato | Datos que debe representar |
| --- | --- |
| MissionDefinition | ID, objetivo, requisitos, recursos, estados, evaluación y recompensa. |
| RegionDefinition | ID, destino, mapa, lugares, navegación, entorno y recursos visuales. |
| CollectibleDefinition | ID, tipo, evidencia, origen, condiciones y consecuencia. |
| DialogueNode | ID, texto por idioma, opciones, requisitos y eventos permitidos. |
| ScienceSource | ID, institución, URL, fecha, versión, unidades y transformación. |
| SaveGame | Versión de esquema/contenido, progreso, recursos, hallazgos y memoria de LUMA. |

Los IDs no se traducen. El texto visible usa español e inglés. Las referencias desconocidas se detectan al validar contenido.

## Comandos y eventos

Los controles producen comandos como mover, interactuar, planificar o regresar. Solo la simulación acepta un comando y genera eventos de progreso.

Un evento de hallazgo o entrega debe poder procesarse de nuevo sin conceder dos recompensas. La UI y LUMA observan eventos; no los inventan.

## POST /api/companion/chat

Entrada: mensaje, idioma y contexto limitado de misión. Salida: texto y referencias a fuentes permitidas.

El servidor valida tamaño y forma, reconstruye el contexto científico desde su catálogo y mantiene las credenciales fuera del cliente. No ejecuta código ni acciones sugeridas por el texto del jugador.

Ante indisponibilidad, devuelve un estado que activa los diálogos escritos. El progreso permanece local. La API no recibe autoridad para modificar la partida.
