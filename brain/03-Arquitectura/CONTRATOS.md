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
| MissionDefinition | ID, objetivo, aporte al rescate, requisitos, piezas de evidencia, recursos, estados, evaluación y recompensa. |
| ChapterDefinition | ID, misiones principales, dependencias, cierre y siguiente capítulo. |
| TransitLegDefinition | Origen, destino, plan de recursos, duración física, hitos y llegada. |
| FlightState | Marco local, objetivo, posición, orientación, velocidad, combustible y asistencia. |
| CompanionNarrativeState | Etapa, deterioro, paleta, condición de vida, recuerdos y escenas procesadas. |
| AuroraEvidenceState | IDs de piezas localizadas/recuperadas, evidencias validadas, fuentes, fechas y revelaciones permitidas. |
| RescueManifest | IDs de supervivientes confirmados, localización, estado de traslado y recepción; plazas y soporte de vida requeridos. |
| RegionDefinition | ID, destino, mapa, lugares, navegación, entorno y recursos visuales. |
| CollectibleDefinition | ID, tipo, evidencia, origen, condiciones y consecuencia. |
| DialogueNode | ID, texto por idioma, opciones, requisitos y eventos permitidos. |
| ScienceSource | ID, institución, URL, fecha, versión, unidades y transformación. |
| SaveGame | Versiones, capítulos y misiones, recursos, hallazgos, estado narrativo, evidencias de Aurora, manifiesto, nave y tramo activo. |

Los IDs no se traducen. El texto visible usa español e inglés. Las referencias desconocidas se detectan al validar contenido.

## Comandos y eventos

Los controles producen comandos como mover, interactuar, planificar o regresar. Solo la simulación acepta un comando y genera eventos de progreso.

Un evento de hallazgo o entrega debe poder procesarse de nuevo sin conceder dos recompensas. La UI y LUMA observan eventos; no los inventan.

## Reglas de historia y rescate

La [historia principal — spoilers](../02-Diseno/HISTORIA-PRINCIPAL.md) define AUR-E01–AUR-E07. Conservar por separado piezas recuperadas, evidencias validadas y destinos revelados. Validar exige las piezas y misiones indicadas; un texto de chat o registro aún corrupto no lo sustituye.

El manifiesto describe a los supervivientes; los miembros originales de la nave de rescate y LUMA se contabilizan también al comprobar capacidad. Los IDs de personas evitan duplicar traslados al repetir eventos. El total y las necesidades se muestran antes de aprobar la evacuación. Ningún manifiesto vacío puede acreditar un rescate completo.

El director de campaña permite iniciar C4-M05 solo con todos los supervivientes confirmados a bordo y retorno viable. El cierre de C4-M06 exige recepción terrestre de ese mismo conjunto. El director narrativo registra los hitos de LUMA una vez y selecciona recuerdos existentes para la despedida escrita.

## POST /api/companion/chat

Entrada: mensaje, idioma y contexto limitado de misión. Salida: texto y referencias a fuentes permitidas.

El servidor valida tamaño y forma, reconstruye el contexto científico desde su catálogo y mantiene las credenciales fuera del cliente. No ejecuta código ni acciones sugeridas por el texto del jugador.

Ante indisponibilidad, devuelve un estado que activa los diálogos escritos. El progreso permanece local. La API no recibe autoridad para modificar la partida.

La condición de vida se comprueba antes de ofrecer conversación. Una avería de IA no modifica esa condición. Una etapa que no permite conversaciones nuevas conserva solo archivos identificados; no activa diálogo de personaje por fallback.

El contexto contiene únicamente la etapa y los hechos revelados. Los acontecimientos del cierre se ejecutan con contenido escrito.

El servidor no recibe el guion completo como contexto de conversación. Construye una vista de conocimiento desde las evidencias validadas y la etapa permitida; no adelanta Bennu, supervivientes o sacrificio. Los textos de piezas todavía parciales tienen su propia versión revelable.
