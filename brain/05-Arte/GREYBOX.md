---
id: greybox
type: arte
status: especificado-sin-implementar
updated: 2026-10-19
tags: [orbita, arte, greybox, demo]
---
# Greybox de la demo

## Propósito

Es la representación **temporal** en grises y blanco con la que se programa y prueba la demo antes de tener arte. El propietario pidió gris y blanco por ahora. **No sustituye la dirección acordada:** el arte final sigue siendo pixel art 2.5D isométrico con la paleta solar de LUMA, como fija la [guía visual](GUIA-VISUAL.md). Se decidirá después y entrará sin tocar la simulación (ver la sección de reemplazo).

Fuentes: [GDD de la demo](../02-Diseno/DEMO/GDD-DEMO.md), [parámetros](../02-Diseno/DEMO/PARAMETROS-DEMO.md), [diseño de nivel](../02-Diseno/DEMO/DISENO-DE-NIVEL.md), [mecánicas](../02-Diseno/DEMO/MECANICAS.md), [animaciones y audio](ANIMACIONES-Y-AUDIO.md) y [LUMA](../02-Diseno/LUMA.md).

**No se ha producido ningún sprite, atlas ni imagen.** Esta nota es una especificación. La producción empieza en la ventana autorizada ([AGENTS](../../AGENTS.md)).

Reglas de base: rombo isométrico `isoTileWidthPx × isoTileHeightPx` = 64 × 32 px, `heightStepPx` = 16 px y `internalResolution` = 960 × 540 px. En el plano, 1 m equivale a 16 px en horizontal (eje `x − y`) y 8 px en vertical (eje `x + y`), así que un círculo de diámetro d (en metros) se dibuja como una elipse de unos 22,6·d × 11,3·d px.

## Tokens de color

Hay ocho grises y blanco. Son neutros, para que ningún estado dependa del matiz.

| Token | Hex | Rol |
| --- | --- | --- |
| `gb-space` | `#0E0E0E` | Fondo de espacio y contorno oscuro de entidades |
| `gb-panel` | `#1C1C1C` | Panel de UI (opaco) y cara derecha de la roca |
| `gb-shadow` | `#2B2B2B` | Suelo en sombra (con patrón obligatorio) y cara izquierda de la roca |
| `gb-rock` | `#4A4A4A` | Cara superior de roca y acantilado; cara lateral de la elevación |
| `gb-lit` | `#8C8C8C` | Suelo iluminado y arista de luz de la roca |
| `gb-muted` | `#A3A3A3` | Texto secundario de UI e iconos inactivos |
| `gb-elev` | `#B8B8B8` | Cara superior de elevación y cresta; cuerpo de LUMA |
| `gb-text` | `#EDEDED` | Texto principal de UI |
| `gb-white` | `#FFFFFF` | Astronauta y acento de interacción (anillos, marcos y foco) |

### Contraste comprobado

Fórmula: luminancia relativa WCAG 2.x, `L = 0,2126 R + 0,7152 G + 0,0722 B` con canales linealizados, y ratio `(L1 + 0,05) / (L2 + 0,05)`. Los valores están calculados, no estimados.

| Primer plano / fondo | Ratio | Uso | Criterio |
| --- | --- | --- | --- |
| `gb-text` / `gb-panel` | 14,56 | Texto UI principal | AAA (≥ 7) |
| `gb-white` / `gb-panel` | 17,04 | Cifras destacadas, foco | AAA |
| `gb-muted` / `gb-panel` | 6,76 | Texto secundario | AA (≥ 4,5); no usar en texto esencial < 18,66 px negrita |
| `gb-text` / `gb-space` | 16,49 | Texto sobre fondo de espacio (menús) | AAA |
| `gb-muted` / `gb-space` | 7,65 | Secundario sobre espacio | AAA |
| `gb-space` / `gb-lit` | 5,74 | Contorno y glifos sobre suelo iluminado | AA texto; ≥ 3 no-texto |
| `gb-space` / `gb-elev` | 9,73 | Contorno sobre elevación y LUMA | AAA |
| `gb-panel` / `gb-elev` | 8,59 | Tecla dibujada (glifo sobre tecla clara) | AAA |
| `gb-text` / `gb-rock` | 7,57 | Etiqueta sobre roca (evitar; preferir panel) | AAA |

Contraste de objetos que no son texto (WCAG 1.4.11, ≥ 3:1) y soluciones donde no llega:

| Par | Ratio | ¿≥ 3? | Regla |
| --- | --- | --- | --- |
| `gb-white` / `gb-lit` (astronauta sobre luz) | 3,36 | Sí | Contorno de 1 px `gb-space` siempre. |
| `gb-white` / `gb-elev` (astronauta en el mirador) | 1,98 | No | Contorno de 1 px `gb-space` (9,73). |
| `gb-white` / `gb-shadow` | 14,16 | Sí | — |
| `gb-lit` / `gb-shadow` (luz y sombra) | 4,21 | Sí | Además, patrón rayado y línea de borde (no depender solo de luminancia). |
| `gb-rock` / `gb-lit` | 2,64 | No | Contorno de base `gb-space` (5,74) y caras laterales con altura. |
| `gb-rock` / `gb-shadow` (roca en sombra) | 1,60 | No | Arista superior de 1 px `gb-lit` (4,21 sobre `gb-shadow`). |
| `gb-elev` / `gb-lit` (elevación) | 1,70 | No | Escalón de `heightStepPx`, cara lateral `gb-rock` (4,47) y curva de nivel. |
| `gb-elev` / `gb-shadow` (cresta junto a sombra) | 7,14 | Sí | — |
| `gb-space` / `gb-shadow` | 1,36 | No | No usar contorno oscuro como única señal en sombra; en sombra, el contorno de entidades es `gb-white`. |

## Entidades y primitivas

Tamaños de la caja de dibujo en px, a escala 64 × 32 y zoom 1. El ancla es el centro de la base, sobre el suelo. Todas llevan un contorno de `outlinePx` = 1 px: `gb-space` en luz y `gb-white` en sombra. El contorno de estado es de 2 px.

| Entidad (ID de atlas) | Primitiva | Tamaño (px) | Notas |
| --- | --- | --- | --- |
| Astronauta `ent.astronaut` | Cápsula vertical blanca (rectángulo redondeado) y elipse de base | 20 × 44; base 18 × 9 (`playerRadiusM` 0,4) | **Indicador de 8 orientaciones:** cuña de visor `gb-panel` de 8 × 6 px en la parte alta, orientada hacia N, NE, E, SE, S, SO, O o NO. Debe leerse en una captura fija. |
| LUMA `ent.luma` | **Cubo isométrico** `gb-elev` (cara superior `gb-elev`, caras `gb-lit` y `gb-rock`), cuatro **patas cortas** de 2 × 3 px, **dos puntos «ojos»** de 2 × 2 px `gb-space` en la cara frontal y dos apéndices cortos de 1 × 4 px arriba | 16 × 20; base 14 × 7 (`lumaRadiusM` 0,3) | Animal compacto, **no un vehículo**: sin ruedas, antena de radio ni luces de equipo. La postura de los apéndices comunica estado (arriba: atención; caídos: descanso). Sin barra de salud. |
| Módulo `ent.lander` | Caja de 3 × 3 rombos con mástil | 192 × 160 (huella 192 × 96) | Caras `gb-text`, `gb-muted` y `gb-lit`. Escotilla marcada con un marco blanco de 2 px en el ancla de POI-01. |
| Terminal `ent.terminal` | Pilar bajo con pantalla rectangular | 24 × 40 | Pantalla `gb-panel` con barras de estado (ver estados). |
| Relé `ent.relay` | Mástil y plato (elipse) sobre una base | 32 × 88 | Es el objeto más alto del mapa. Plato inclinado = desalineado; vertical = alineado. |
| Caché `ent.cache` | Caja baja con baliza | 28 × 24 | Baliza de 4 × 4 px `gb-white`. Abierta = tapa desplazada 4 px. |
| Roca `tile.rock` | Bloque de un rombo con altura 1–2 niveles | 64 × 48 / 64 × 64 | Cara superior `gb-rock`, arista `gb-lit` y caras `gb-shadow`/`gb-panel`. |
| Borde de cráter `tile.rim` | Bloque de un rombo, un nivel | 64 × 48 | Cara superior `gb-elev` con **peine de trazos** `gb-rock` (marcas de talud cartográficas) en la cara que da al cráter. Bloqueante. |
| Elevación `tile.elevation` | Rombo elevado un nivel | 64 × 48 | Cara superior `gb-elev`, cara lateral `gb-rock` y curva de nivel de 1 px. |
| Suelo iluminado `tile.ground.lit` | Rombo plano | 64 × 32 | `gb-lit`. Rejilla de depuración opcional (F3). |
| Suelo en sombra `tile.ground.shadow` | Rombo plano con patrón | 64 × 32 | `gb-shadow` con patrón (ver luz y sombra). |
| POI `ui.poi` | Rombo de contorno sobre el ancla y número | 64 × 32 | Etiqueta HTML «01»…«06» con nombre corto. Estados por contorno (ver tabla). |
| Waypoint `ent.waypoint` | Poste con banderín triangular | 8 × 24 | Solo contorno. No bloquea. |
| Nave en aproximación `ent.ship` | Punta de flecha isométrica con tobera | 48 × 48 | Línea de rumbo de 24 px, **vector de velocidad** como línea discontinua proporcional y chorro del motor como triángulo de contorno. |
| Zona de descenso `ui.landingZone` | Círculo de radio `landingZoneRadiusM` a la escala de la aproximación | Según la escala de vuelo | Cuña de orientación de ±`landingHeadingTolDeg` y estados de descenso (ver tabla). |

## Estados sin depender del color

Cada estado combina **forma, contorno o patrón e icono**. El parpadeo es siempre redundante: con movimiento reducido se sustituye por una alternativa estática equivalente.

| Estado | Codificación principal | Icono | Con animación | Sin animación |
| --- | --- | --- | --- | --- |
| Seleccionable (bajo cursor o foco) | Esquinas «[ ]» de 2 px alrededor | — | — | Igual |
| Interactuable en rango (≤ `interactRadiusM`, R ≤ `scanRadiusM`) | Contorno **continuo** de 2 px `gb-white` | Tecla «E» o «R» en un rombo sobre el objeto, con duración («E · 6 min») | Contorno que late a 1 Hz | Contorno **doble** estático |
| Completado | Relleno sólido del rombo del POI | Marca ✓ en un círculo | — | Igual |
| Bloqueado (falta equipo o requisito) | Contorno **punteado** | Candado y texto del motivo («Requiere espectrómetro») | — | Igual |
| Sombra transitable (con `MOD-LAMP`) | Rayado diagonal **simple** y borde **discontinuo** | Icono de lámpara en el HUD al pisarla | — | Igual |
| Sombra no transitable (sin lámpara) | Rayado **cruzado** denso y borde **continuo** de 2 px con chevrones hacia fuera cada 32 px | Al tocar el borde: el mensaje de sombra de [mecánicas](../02-Diseno/DEMO/MECANICAS.md) (§6.3) | — | Igual |
| Aviso de soporte vital | Barra segmentada con **borde doble** y marca de regreso «R│» | Triángulo «!» y texto «Regresa: N min de camino» | Pulso de 1 Hz del borde | Borde doble estático y texto en negrita |
| Condiciones de descenso cumplidas o no | Tres casillas: velocidad ≤ `landingMaxSpeedMps`, orientación ±`landingHeadingTolDeg` y dentro de zona. Cumplida = cuadrado **relleno** con ✓; no cumplida = cuadrado **vacío** con ×, más el valor numérico | Anillo de la zona **discontinuo** si falta alguna y **continuo** si se cumplen todas. Temporizador `landingHoldS` en cuatro segmentos de 0,5 s | Los segmentos se llenan | Igual (el temporizador es numérico además de gráfico) |

## LUMA en greybox

- Las cuatro paletas solares (#FFEAA7, #F1A246, #C45A43, #D9EEFF) **no se aplican en greybox**. LUMA usa `gb-elev` en toda la demo. La paleta de C2 llegará con el arte final.
- **Reacción (`LUMA-R-*`):** burbuja del traductor con forma de rectángulo redondeado `gb-panel`, borde de 1 px `gb-white` y cola hacia LUMA. Lleva un icono de onda «≈», que significa «vocalización traducida», y el texto en HTML con subtítulo. La reacción se distingue por **forma** (burbuja con cola y onda) y **postura** (apéndices arriba, giro hacia el estímulo), nunca por color. Con animación, LUMA da un salto de 2 px; sin animación, cambia a un fotograma de postura estático.
- Las reacciones no tapan los avisos de interacción ni la barra de soporte vital. Su pausa y su límite de repetición los fijan [LUMA](../02-Diseno/LUMA.md) y [mecánicas](../02-Diseno/DEMO/MECANICAS.md).

## Luz y sombra en grises

- **Sombra:** fondo `gb-shadow` con patrón rayado a 45° (línea `gb-rock` de 1 px cada `shadowHatchSpacingPx` = 4 px). El patrón cambia de simple a cruzado según el equipo: con lámpara es simple; sin lámpara es cruzado, la sombra no se puede cruzar.
- **Borde de sombra:** línea en la arista entre celdas de luz y de sombra. Es continua de 2 px con chevrones si no se puede cruzar y discontinua de 1 px si se lleva lámpara.
- **Lámpara en uso:** círculo de render alrededor del astronauta (`lampVisualRadiusM` = 3 m) donde el suelo se dibuja `gb-lit` con un contorno punteado. El HUD muestra el consumo `lampW`. El círculo es solo visual: la regla de consumo depende de la celda que pisa el jugador.
- **Luz rasante:** las caras que miran al oeste son claras y las que miran al este, oscuras. Coincide con `sunFromDirection` = W.
- **Oclusión:** un oclusor delante del jugador o de LUMA baja a `occluderAlpha` (0,4) durante `occluderFadeS` (0,15 s), valores de [físicas](../02-Diseno/DEMO/FISICAS.md) aceptados en [parámetros](../02-Diseno/DEMO/PARAMETROS-DEMO.md), y la silueta del personaje se dibuja encima con un contorno `gb-white` de 1 px.

## HUD y planificador en greybox

Son wireframes ASCII. El texto vive en la interfaz HTML (nítida y escalable) y el canvas solo dibuja el mundo. Los códigos de validación son los del [GDD](../02-Diseno/DEMO/GDD-DEMO.md#validación-del-planificador): V1–V6 son errores y bloquean el lanzamiento; W1–W5 son avisos y no bloquean (W5 = sin kit). Los iconos van en texto: `[*]` seleccionado, `[x]` error, `[!]` aviso, `[ok]` válido, `[E]` tecla y `[#]` candado.

### Planificador

```text
+--------------------------------------------------------------------------+
| PLANIFICADOR · MIS-DEMO-01         | Presupuesto 57 / 100 cr             |
+--------------------------------------------------------------------------+
| MODULOS                 kg  cr     | CARGA      [######--]  40 / 60 kg   |
| [*] Lector de registros  5  10 ob  | COSTE      [######--]  57 / 100 cr  |
| [*] Proteccion de LUMA   7   0 ob  | PROPELENTE cargado 120 kg           |
| [*] Bateria portatil    14  12     |            exigido 102,6 kg         |
| [ ] Panel solar          9  18     |            margen 34,5 %            |
| [*] Espectrometro        8  15     | SOPORTE    81,4 / 120 min (67,8 %)  |
| [ ] Kit de reparacion   10  10     | ENERGIA    460 Wh                   |
| [*] Lampara de sombra    6   8     | RUTA       sombra 237 m (80 m)      |
| [ ] Reserva soporte     12  14     |                                     |
| TANQUE ( ) S 96 (o) M 120 ( ) L 150|                                     |
+--------------------------------------------------------------------------+
| VALIDACION (V = error, bloquea; W = aviso, no bloquea)                   |
| [ok] V1 carga 40 <= 60 kg          | [ok] W1 margen 34,5 % >= 30 %       |
| [ok] V2 coste 57 <= 100 cr         | [ok] W2 espectrometro a bordo       |
| [ok] V3 lector de registros        | [ok] W3 lampara: ruta de sombra     |
| [ok] V4 energia rele: bateria      | [ok] W4 soporte 67,8 % <= 70 %      |
| [ok] V5 propelente 120 >= 102,6 kg | [!]  W5 sin kit: alineacion         |
| [ok] V6 proteccion de LUMA         |      manual del rele (15 min)       |
+--------------------------------------------------------------------------+
| [Esc] Volver  [F1] Ayuda           | [ LANZAR ] bloqueado si hay [x]     |
+--------------------------------------------------------------------------+
```

### HUD de vuelo (aproximación)

```text
+--------------------------------------------------------------------------+
| DIST 400 m   CIERRE 18,0 m/s   LATERAL 2,5 m/s     PROPELENTE p 118,0 kg |
|                                           UMBRAL p* (reserva) 88,65 kg   |
|                           SOBRANTE PARA CORREGIR = p - p* = 29,35 kg     |
|                                p [##########|######.......]  | = p*      |
|                                                                          |
|          .  -  -  -  .                                                   |
|       -     zona       -        CONDICIONES DE DESCENSO                  |
|      |    (discontinua)  |      [x]  velocidad    18,0 / 1,0 m/s         |
|       -       /\       -        [ok] orientacion   +0 / 15 grados        |
|          .  - || -  .           [x]  dentro de zona                      |
|               ''  ---> vector   MANTENER [....] 0,0 / 2,0 s              |
|                                                                          |
| [W] impulso [S] retro [A/D] girar [Q/E] lateral [X] igualar [Mayus] fino |
| ASISTENTE: apagado (gasta +10 %)                          [Esc] pausa    |
+--------------------------------------------------------------------------+
```

### HUD de superficie

```text
+--------------------------------------------------------------------------+
| SOPORTE [#########|#####------] 96,6 / 120 min    R| regreso 16,4 min    |
| ENERGIA 449 Wh   LAMPARA apagada (en luz)                                |
|                                                    OBJETIVOS             |
|                                                    [ok] Registro (04)    |
|                                                    [ ]  Rele (05)        |
|            (mundo isometrico)                      [ ]  Entrega (06)     |
|                                                    --- cientifico ---    |
|                                                    [ok] Observacion (02) |
|                                                    [ ]  Escaneo (03)     |
|                                                    [ ]  Muestra (06)     |
| +---------------------------+       +------------------------------+     |
| | Registro recogido         |       | LUMA ~ "Ella toco esto. Su   |     |
| | pendiente de autenticacion|       |        marca."               |     |
| | [Tab] diario              |       +------------------------------+     |
| +---------------------------+                                            |
| [M] mapa  [Tab] diario  [F1] ayuda   Despegar: en POI-01   [Esc] pausa   |
+--------------------------------------------------------------------------+
```

El momento es justo después de `ItemRecovered` en la configuración A, siguiendo el ritmo nominal del [diseño de nivel](../02-Diseno/DEMO/DISENO-DE-NIVEL.md#8-ritmo-minuto-a-minuto): 96,6 min de 120, regreso de 78,9 m a POI-01 y 460 − 11,0 Wh de un cruce de sombra. La burbuja reproduce `LUMA-R-CACHE` de [mecánicas](../02-Diseno/DEMO/MECANICAS.md), sin tildes por ser ASCII, y no revela contenido del registro. «Despegar» solo aparece en POI-01: entregar no inicia el ascenso.

### Informe

```text
+--------------------------------------------------------------------------+
| INFORME · MIS-DEMO-01 · config. A               Resultado: EXITO PARCIAL |
+--------------------------------------------------------------------------+
| RECURSO          PREVISTO    REAL       DIFERENCIA  CAUSA                |
| Soporte vital    81,4 min    96,0 min   +14,6 min   desvios y esperas    |
| Energia          123,7 Wh    124,5 Wh   +0,8 Wh     3 m mas en sombra    |
| Sobrante aprox.  29,35 kg    26,35 kg   -3,0 kg     correccion tardia    |
+--------------------------------------------------------------------------+
| OBJETIVO PRINCIPAL  [ok] registro  [ok] rele  [ok] entrega               |
| CIENTIFICO 2/3      [ok] observacion  [ok] escaneo  [x] muestra          |
|                     -> recogida pero no entregada en POI-06              |
| ESTADO DEL REGISTRO recogido · pendiente de autenticacion                |
| ASCENSO             manual desde POI-01 (RequestAscent)                  |
+--------------------------------------------------------------------------+
| FUENTES Y SIMPLIFICACIONES  [O] gravedad lunar  [S] tiempo x20  [F] ...  |
| [ Reintentar con otra configuracion ]  [ Ver fuente ]  [ Terminar ]      |
+--------------------------------------------------------------------------+
```

Las cifras de los wireframes usan los valores vigentes de [parámetros](../02-Diseno/DEMO/PARAMETROS-DEMO.md):

- Planificador: configuración A con TANK-M. Carga 40 kg, 57 cr, exigido 102,6 kg, margen 34,5 %, soporte vital 81,4 de 120 min (67,8 %). W5 aparece porque A no lleva kit.
- HUD de vuelo: inicio de la aproximación de A con la corrección a tiempo (120 − 2 = 118,0 kg; p* = 88,65 kg; sobrante 29,35 kg).
- Informe: resultado ilustrativo de A con una corrección tardía (sobrante −3 kg).

Los valores del juego los calcula la simulación.

## Tipografía

- **Greybox:** pila de sistema, sin descargas ni dependencia de red: `system-ui, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif`. Las cifras del HUD usan `font-variant-numeric: tabular-nums` para que no salten al cambiar.
- **Opcional en la ventana autorizada:** Atkinson Hyperlegible (Google Fonts), autoalojada para funcionar sin conexión. La licencia se comprueba antes de incluirla. No se decide ahora.

## Reemplazo por el arte final sin tocar la simulación

1. La simulación solo emite **IDs de entidad, estados y orientación** (`ent.luma`, `state.inRange`, `dir.SE`). Nunca nombres de textura ([arquitectura](../03-Arquitectura/ARQUITECTURA.md)).
2. El `EntityVisualRegistry` de la [estructura de código](../03-Arquitectura/ESTRUCTURA-CODIGO.md) asocia cada ID o tipo de entidad con una fábrica visual. En la demo, PreloadScene genera texturas greybox con las primitivas de esta nota. El arte final se carga como atlas con la clave de fotograma `<entidad>/<estado>/<dir>/<frame>`, y cambiarlo solo afecta al registro y a PreloadScene.
3. El `MapDefinition` (exportación de Tiled) lleva por celda `height`, `blocked`, `shadow` y `terrain`, que es solo una clave visual (`lit`, `shadow`, `rock`, `rim`, `elevation`). Las reglas leen `blocked` y `shadow`, no el índice de tile. Cambiar el tileset no cambia las reglas.
4. Una prueba de contenido compara la cobertura: cada entidad y estado del registro greybox debe tener su clave en el atlas final antes de sustituirlo.
5. Las paletas de LUMA se aplican como variante del atlas por etapa (`luma.c2`). El greybox usa una sola variante.

## Accesibilidad

- **Texto:** mínimo 16 px CSS en el HUD y 18 px en las cifras principales (`uiMinFontPx` = 16). La escala de la interfaz va del 100 al 150 % sin solapes. Todo el texto esencial cumple AA y el texto principal, AAA (tabla de contraste).
- **Movimiento reducido:** se respeta `prefers-reduced-motion` y hay un ajuste en el juego. Con él se quitan parpadeos, pulsos y saltos y se usan las alternativas estáticas de la tabla de estados. Tampoco hay sacudidas de cámara ni paralaje, y el desvanecido de oclusión es instantáneo.
- **Destellos:** nunca más de 3 por segundo. El pulso más rápido es de 1 Hz (`blinkHz` = 1).
- **Sin depender del color:** cada estado tiene forma, contorno o patrón e icono. La sombra se lee por el patrón aunque se ignore la luminancia.
- **Subtítulos:** las reacciones de LUMA y los avisos tienen texto equivalente ([animaciones y audio](ANIMACIONES-Y-AUDIO.md)).
- **Foco de teclado:** el foco visible usa las esquinas «[ ]» y un contorno blanco de 2 px. Mientras un panel tiene el foco, el personaje no se mueve ([GDD](../02-Diseno/DEMO/GDD-DEMO.md#máquina-de-estados-del-juego)).

## Parámetros

Este greybox no propone parámetros nuevos. Usa estos valores, aceptados en [parámetros](../02-Diseno/DEMO/PARAMETROS-DEMO.md#parámetros-técnicos-aceptados-de-otras-esquemáticas):

- `lampVisualRadiusM` = 3 m
- `uiMinFontPx` = 16 px
- `blinkHz` = 1 Hz
- `outlinePx` = 1 / 2 px
- `shadowHatchSpacingPx` = 4 px
- Del diseño de nivel: `sunFromDirection` = W.
- De físicas: `occluderAlpha` = 0,4 y `occluderFadeS` = 0,15 s.

La clave de fotograma `<entidad>/<estado>/<dir>/<frame>` es un contrato de render y no afecta al balance.
