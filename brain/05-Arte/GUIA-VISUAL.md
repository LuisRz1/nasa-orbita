---
id: guia-visual
type: arte
status: direccion-acordada
updated: 2026-10-14
tags: [orbita, arte]
---
# Guía visual

## Dirección

Pixel art 2.5D isométrico con astronauta de cuerpo completo y silueta legible. Combinar superficies sobrias, luz ambiental y equipos identificables.

La resolución interna propuesta es 960 × 540, escalada respetando los píxeles cuando sea posible. La interfaz HTML conserva texto nítido y tamaño ajustable en pantallas de PC.

## Lenguaje

- La silueta distingue astronauta, animal LUMA, rover, instrumento y obstáculo.
- Cada destino tiene una paleta y un contraste ambiental propio.
- Color, forma e icono comunican estados; no depender solo de rojo y verde.
- El brillo señala información útil sin convertir todos los objetos en recompensas.
- La iluminación diferencia zonas y peligros sin ocultar por completo la navegación.
- Los elementos que tapan al personaje se atenúan de manera consistente.

## Pipeline durante implementación

Mapa en Tiled → exportación JSON → validación de capas y referencias. Sprites editables en LibreSprite → atlas y metadatos de animación → carga por región.

Conservar archivos editables y registrar licencias. Los tamaños de tile y sprite se fijarán con la primera prueba de legibilidad, antes de producir el resto del contenido.

Esta nota es una especificación. No se han producido sprites, mapas o una muestra visual del juego.

## LUMA y paleta estelar

Conservar forma compacta y cúbica, patas cortas y ojos pequeños. Sustituir paleta sobre el mismo diseño; no cambiar de especie por capítulo.

| Etapa | Color principal de referencia |
| --- | --- |
| C1 | Blanco cálido/dorado · #FFEAA7 |
| C2 | Naranja/ámbar · #F1A246 |
| C3 | Rojo cobrizo · #C45A43 |
| C4 | Blanco azulado · #D9EEFF |

Son valores artísticos ajustables tras revisar contraste; no codifican temperatura. Postura, conducta y texto comunican los cambios también con efectos reducidos.
