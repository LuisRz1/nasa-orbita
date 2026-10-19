---
id: parametros-demo
type: especificacion
status: provisional-sin-implementar
updated: 2026-10-19
tags: [orbita, demo, balance, parametros]
---
# Parámetros de la demo

**Fuente única** de nombres, unidades y valores iniciales para todas las esquemáticas de la demo. Si otro documento necesita un valor, lo cita por su nombre técnico. Si hace falta uno nuevo, se propone aquí antes de usarlo.

**Todos los valores son provisionales.** Son hipótesis de balance calculadas el 19 de octubre para comprobar que las reglas encajan. El balance final se ajusta en la ventana autorizada, después de revisar el enunciado completo (28 de octubre) y de las pruebas con jugadores. Ver [bucle y misiones](../BUCLE-Y-MISIONES.md).

Columna **Nivel**: O = observado (con fuente), S = simplificación de juego, F = ficción de diseño. Ver [datos y realismo](../../04-Ciencia/DATOS-Y-REALISMO.md).

## Simulación y tiempo

| Nombre | Valor | Unidad | Nivel | Uso |
| --- | --- | --- | --- | --- |
| simHz | 60 | Hz | S | Paso fijo de la simulación; `dt = 1/simHz`. |
| maxSubstepsPerFrame | 5 | pasos | S | Límite del acumulador para evitar espiral de muerte. |
| surfaceTimeScale | 20 | s misión / s real | S | En superficie, 1 s real equivale a 20 s de misión. |
| cruiseRealSeconds | 30 | s reales | S | Duración real del crucero acelerado sin contar pausas ni escenas. |
| cruisePhysicalHours | 76 | h | O | Duración física mostrada. Apollo 11 entró en órbita lunar a unas 75 h 50 min del lanzamiento ([NASA-APOLLO11](../../04-Ciencia/FUENTES.md)). Se redondea y se declara como referencia histórica, no como trayectoria calculada. |
| rngSeed | 20261114 | entero | S | Semilla fija de la demo; solo variaciones cosméticas. |

## Vehículo y propulsión

| Nombre | Valor | Unidad | Nivel | Uso |
| --- | --- | --- | --- | --- |
| landerDryMassKg | 300 | kg | F | Masa en seco del módulo de aproximación ficticio. |
| payloadCapacityKg | 60 | kg | F | Carga máxima de módulos (restricción V1). |
| returnCargoKg | 3 | kg | F | Muestra + registro de Aurora. La validación (V5) y la reserva en vuelo lo suman siempre (caso conservador). El ascenso real usa la carga que se lleve de verdad (0–3 kg). |
| ispS | 311 | s | S | Impulso específico de motor y RCS. Valor de juego del orden de motores de propelentes almacenables; no se atribuye a un motor concreto. Simplificación: el mismo Isp para ambos. |
| g0 | 9.80665 | m/s² | O | Gravedad estándar para `ve = ispS × g0` (≈ 3049,9 m/s). |
| mainThrustN | 1500 | N | F | Empuje del motor principal. |
| rcsThrustN | 200 | N | F | Empuje de traslación lateral o retro por eje. |
| turnRateDegPerS | 90 | °/s | S | Giro directo sin momento angular. |
| precisionFactor | 0.25 | — | S | Multiplicador de empuje y giro con Mayús. |
| midcourseNowKg | 2 | kg | F | Corrección a mitad de crucero si se hace a tiempo. |
| midcourseLateKg | 5 | kg | F | Corrección si se retrasa. |
| approachNominalDeltaVMps | 40 | m/s | S | Δv estimado para una aproximación correcta (incluye descenso final asistido). |
| ascentDeltaVMps | 600 | m/s | S | Δv de juego del ascenso automático. **No es el Δv real de un ascenso lunar**, que es del orden de km/s. |
| propellantMarginHard | 0.15 | fracción | S | Margen mínimo exigido (V5) y reserva en vuelo. |
| propellantMarginWarn | 0.30 | fracción | S | Por debajo, aviso W1. |

Fórmula de consumo: `propellantForDeltaV(m, dv) = m × (1 − exp(−dv / ve))`, la ecuación del cohete ([NASA-ROCKET-EQ](../../04-Ciencia/FUENTES.md)). Ver [físicas](FISICAS.md).

**Reserva en vuelo**, evaluada en cada paso de la aproximación con la masa actual `m(t)`, propelente incluido:

`ascentReserveKg(t) = propellantForDeltaV(m(t) + returnCargoKg, ascentDeltaVMps) × (1 + propellantMarginHard)`

Si el propelente restante baja de `ascentReserveKg(t)` se produce `ApproachAborted`. Como `m(t)` incluye el propelente `p(t)`, la condición equivale a un **umbral fijo**:

`p* = k × (landerDryMassKg + carga + returnCargoKg) / (1 − k)`, con `k = (1 − exp(−ascentDeltaVMps / ve)) × (1 + propellantMarginHard)` ≈ 0,20538.

"Sobrante para corregir" = `p(t) − p*`. Es lo que muestra el HUD. Ver [físicas](FISICAS.md).

| Nombre | Valor | Unidad | Nivel | Uso |
| --- | --- | --- | --- | --- |
| midcourseDecisionOpenS | 8 | s reales de crucero | S | Se abre la decisión de corrección (≈ 20 h físicas). |
| midcourseDecisionCloseS | 20 | s reales de crucero | S | Si no se ha decidido, se aplica `midcourseLateKg` automáticamente (≈ 51 h físicas). |
| landingSequenceRealS | 4 | s | S | Duración de la secuencia automática LANDING. |
| ascentSequenceRealS | 10 | s | S | Duración de la secuencia automática ASCENT. |

### Tanques (se elige uno)

| ID | Propelente (kg) | Coste (cr) |
| --- | --- | --- |
| TANK-S | 96 | 8 |
| TANK-M | 120 | 12 |
| TANK-L | 150 | 16 |

## Presupuesto y módulos

| Nombre | Valor | Unidad | Nivel |
| --- | --- | --- | --- |
| budgetCredits | 100 | cr | F |

| ID | Nombre visible | Masa (kg) | Coste (cr) | Efecto | Obligatorio |
| --- | --- | --- | --- | --- | --- |
| MOD-DATA-READER | Lector de registros | 5 | 10 | Permite recuperar el registro de la caché (POI-04). | Sí (V3) |
| MOD-LUMA-SHIELD | Protección ambiental de LUMA | 7 | 0 | LUMA puede salir a superficie. Preinstalado y no removible. | Sí (V6) |
| MOD-BATTERY-PACK | Batería portátil | 14 | 12 | +`batteryPackWh` a la energía de equipo. | Una fuente de energía para el relé (V4) |
| MOD-SOLAR-PANEL | Panel solar desplegable | 9 | 18 | Genera `solarPanelW` solo en zona iluminada. | Una fuente de energía para el relé (V4) |
| MOD-SCANNER | Espectrómetro portátil | 8 | 15 | Permite escanear (POI-03) y completar el objetivo científico. | No (W2) |
| MOD-TOOLKIT | Kit de reparación | 10 | 10 | Reduce la alineación del relé de `relayAlignManualMin` a `relayAlignToolkitMin`. | No |
| MOD-LAMP | Lámpara térmica de sombra | 6 | 8 | Habilita la ruta de sombra; consume `lampW` en sombra. | No (W3) |
| MOD-O2-RESERVE | Reserva de soporte vital | 12 | 14 | +`o2ReserveMin` de soporte vital. | No |

Comprobación: todos los módulos suman 71 kg y 87 cr más el tanque. No se puede llevar todo (V1 y V2), así que elegir es obligatorio.

## Superficie: soporte vital, energía y actividades

| Nombre | Valor | Unidad | Nivel | Uso |
| --- | --- | --- | --- | --- |
| lifeSupportBaseMin | 120 | min misión | F | Soporte vital del traje. |
| o2ReserveMin | 60 | min misión | F | Aporte de MOD-O2-RESERVE. |
| toolBatteryBaseWh | 60 | Wh | F | Energía de equipo integrada en el traje. |
| batteryPackWh | 400 | Wh | F | Aporte de MOD-BATTERY-PACK. |
| solarPanelW | 600 | W | F | Generación en luz. 0 en sombra. |
| relaySolarChargeMin | 10 | min misión | F | Derivado: `relayEnergyWh / solarPanelW × 60`. |
| lampW | 80 | W | F | Consumo de la lámpara en sombra. |
| scannerW | 30 | W | F | Consumo del espectrómetro durante el escaneo. |
| relayEnergyWh | 100 | Wh | F | Energía que necesita el relé (INC-RELAY-01). |
| lifeSupportWarnFactor | 1.25 | × | S | Aviso si el soporte restante es menor que el tiempo de regreso × factor + `lifeSupportWarnBufferMin`. |
| lifeSupportWarnBufferMin | 5 | min misión | S | Colchón del aviso de regreso. |
| lifeSupportPlanWarnFraction | 0.70 | fracción | S | W4 si el soporte vital estimado en el planificador supera esta fracción de la capacidad. |

| Actividad | Nombre | Min misión | s reales (a ×20) |
| --- | --- | --- | --- |
| Observar desde el mirador | observeMin | 2 | 6 |
| Escanear punto geológico | scanMin | 3 | 9 |
| Tomar muestra | sampleMin | 3 | 9 |
| Recuperar registro de la caché | cacheRetrieveMin | 6 | 18 |
| Conectar batería al relé | relayBatteryMin | 1 | 3 |
| Desplegar panel solar | relaySolarDeployMin | 2 | 6 |
| Cargar relé con panel (100 Wh / 600 W) | relaySolarChargeMin | 10 | 30 |
| Alinear relé con kit | relayAlignToolkitMin | 5 | 15 |
| Alinear relé a mano | relayAlignManualMin | 15 | 45 |
| Entregar en terminal | deliverMin | 2 | 6 |

La carga solar y la alineación se solapan: la reparación con panel dura `relaySolarDeployMin + max(relaySolarChargeMin, alineación)`. Con kit son 2 + max(10, 5) = 12 min; a mano, 2 + max(10, 15) = 17 min. Con batería son 1 + 5 = 6 min o 1 + 15 = 16 min. El kit ahorra 5 min con panel y 10 min con batería.

## Movimiento en superficie

| Nombre | Valor | Unidad | Nivel | Uso |
| --- | --- | --- | --- | --- |
| moonGravityMps2 | 1.62 | m/s² | O | [NASA-MOON-FACTSHEET](../../04-Ciencia/FUENTES.md). Solo informa y justifica la baja tracción; no hay saltos en la demo. |
| walkSpeedMps | 1.6 | m/s | S | Velocidad de juego, comprimida respecto a una marcha real. |
| walkAccelMps2 | 2.5 | m/s² | S | Aceleración baja: sensación de poca tracción. |
| walkDecelMps2 | 3.0 | m/s² | S | Frenado al soltar. |
| playerRadiusM | 0.4 | m | S | Círculo de colisión del astronauta. |
| lumaRadiusM | 0.3 | m | S | Círculo de LUMA (no bloquea al jugador). |
| lumaFollowDistM | 1.6 | m | S | Distancia de seguimiento. |
| lumaSpeedMps | 1.9 | m/s | S | Velocidad para alcanzar al jugador. |
| lumaRegroupDistM | 8 | m | S | Si queda más lejos o atascado, se reubica junto al jugador con un fundido (`lumaRegroupFadeS`). Con zoom 1 se ven unos ±21 m, así que reubicarlo «fuera de cámara» no es posible. |
| interactRadiusM | 1.5 | m | S | Alcance de la interacción (E). |
| scanRadiusM | 3.0 | m | S | Alcance del escaneo (R). |

Derivado: `minPerMeter = surfaceTimeScale / (walkSpeedMps × 60)` ≈ 0,208 min de misión por metro.

## Mapa y proyección

| Nombre | Valor | Unidad | Uso |
| --- | --- | --- | --- |
| tileSizeM | 2 | m | Lado lógico de una celda. |
| mapWidthTiles × mapHeightTiles | 56 × 44 | celdas | 112 × 88 m lógicos. Mapa comprimido y artístico, no reproducción geográfica. |
| isoTileWidthPx × isoTileHeightPx | 64 × 32 | px | Rombo isométrico 2:1. |
| internalResolution | 960 × 540 | px | [Guía visual](../../05-Arte/GUIA-VISUAL.md). |
| heightStepPx | 16 | px | Desplazamiento vertical por nivel de altura visual. |

Origen lógico: esquina noroeste; `x` hacia el este y `y` hacia el sur, en metros.

### Lugares de interés (coordenadas lógicas en metros)

| ID | Lugar | Centro (x, y) | Iluminación |
| --- | --- | --- | --- |
| DEMO-POI-01 | Módulo y refugio | (20, 70) | Luz |
| DEMO-POI-02 | Mirador del terreno | (38, 52) | Luz, elevado |
| DEMO-POI-03 | Punto geológico | (60, 66) | Luz |
| DEMO-POI-04 | Caché protegida | (82, 32) | Luz, junto al borde de sombra |
| DEMO-POI-05 | Relé | (98, 14) | Luz, borde del cráter |
| DEMO-POI-06 | Terminal de retorno | (26, 74) | Luz, junto al módulo |

### Longitudes de ruta objetivo (el diseño de nivel debe cumplirlas ±10 %)

Las longitudes medidas sobre el mapa con un camino más corto y el radio del jugador están en [diseño de nivel](DISENO-DE-NIVEL.md). Todas cumplen la tolerancia; el recorrido A mide 232,9 m frente a 237. Las estimaciones del planificador usan esta tabla objetivo. Si el mapa cambia, se vuelven a medir.

| Tramo | Longitud (m) | Metros en sombra |
| --- | --- | --- |
| POI-01 → POI-02 | 26 | 0 |
| POI-02 → POI-04 por la sombra | 50 | 40 |
| POI-02 → POI-03 | 26 | 0 |
| POI-03 → POI-04 por el rodeo iluminado | 90 | 0 |
| POI-04 → POI-05 | 25 | 0 |
| POI-03 → POI-06 | 35 | 0 |
| POI-02 → POI-06 | 28 | 0 |

## Pilotaje de aproximación

| Nombre | Valor | Unidad | Uso |
| --- | --- | --- | --- |
| approachStartDistM | 400 | m | Distancia inicial al objetivo. |
| approachStartClosingMps | 18 | m/s | Velocidad inicial hacia el objetivo. |
| approachStartLateralMps | 2.5 | m/s | Deriva lateral inicial. |
| landingZoneRadiusM | 12 | m | Radio de la zona de descenso. |
| landingMaxSpeedMps | 1.0 | m/s | Velocidad relativa máxima para iniciar el descenso. |
| landingHeadingTolDeg | 15 | ° | Tolerancia respecto a la orientación marcada del lugar de descenso. |
| landingHoldS | 2.0 | s | Tiempo dentro de las condiciones para confirmar. |
| overspeedFailMps | 4.0 | m/s | Entrar en la zona por encima de esto es aproximación fallida; se reintenta. |
| lostDistanceM | 800 | m | Alejarse más es aproximación perdida; se reintenta. |
| assistGainPerS | 0.8 | 1/s | Ganancia del asistente "igualar velocidad". |
| assistPropellantPenalty | 1.10 | × | El asistente gasta un 10 % más que un pilotaje equivalente. |

## Configuraciones de referencia (calculadas)

Se calcularon con las fórmulas anteriores, que las esquemáticas deben reproducir. "Necesario" = `midcourseNowKg` + aproximación nominal sobre la masa tras la corrección + ascenso sobre la masa tras la aproximación más `returnCargoKg`. "Exigido" = necesario × (1 + `propellantMarginHard`). "Margen" = propelente cargado / necesario − 1. Las cifras se redondean una sola vez, al final.

| Configuración | Módulos (además de DATA-READER y LUMA-SHIELD) | Carga | Coste sin tanque | TANK-S (96 kg) | TANK-M (120 kg) | TANK-L (150 kg) |
| --- | --- | --- | --- | --- | --- | --- |
| A · Sombra rápida | BATTERY-PACK, LAMP, SCANNER | 40 kg | 45 cr | Exigido 97,4 → **V5**, margen 13,4 % | Exigido 102,6 → válida, margen 34,5 %, 57 cr | Exigido 109,1 → válida, margen 58,0 %, 61 cr |
| B · Luz y paciencia | SOLAR-PANEL, TOOLKIT, SCANNER, O2-RESERVE | 51 kg | 67 cr | Exigido 99,8 → **V5**, margen 10,6 % | Exigido 105,0 → válida, margen 31,4 %, 79 cr | Exigido 111,5 → válida, margen 54,7 %, 83 cr |
| Todo | Los seis opcionales | 71 kg | 87 cr | **V1 + V5** (exigido 104,1), 95 cr | **V1**, 99 cr | **V1 + V2**, 103 cr |

TANK-S es válido con una carga de 33 kg o menos: misiones ligeras como BATTERY-PACK + LAMP (32 kg) o SOLAR-PANEL + SCANNER (29 kg).

Aproximación con TANK-M:

| Configuración | Masa al empezar | Aceleración del motor | Umbral p* | Sobrante para corregir |
| --- | --- | --- | --- | --- |
| A | 458 kg | ≈ 3,28 m/s² | 88,65 kg | 29,35 kg |
| B | 469 kg | ≈ 3,20 m/s² | 91,49 kg | 26,51 kg |
| BATTERY-PACK + LAMP con TANK-S | 426 kg | ≈ 3,52 m/s² | 86,58 kg | 7,42 kg |

La masa del planificador cambia poco el manejo. Su consecuencia en vuelo es el **propelente sobrante para corregir**, que el HUD muestra en kg. Con una corrección tardía (+3 kg) el sobrante baja en esa cantidad.

Con TANK-M el sobrante (≈ 175–200 m/s) perdona un pilotaje torpe; la decisión del tanque pesa sobre todo en el planificador. Una misión ligera con TANK-S deja ≈ 50 m/s y obliga a pilotar con cuidado o a usar poco el asistente. Esa es la tensión buscada entre el planificador y el vuelo.

### Estimación en superficie (recorrido nominal, sin desvíos)

| Configuración | Ruta | Recorrido | Caminar | Actividades | Total | Capacidad | Uso |
| --- | --- | --- | --- | --- | --- | --- | --- |
| A | Sombra, ida y vuelta | 237 m | 49,4 min | 32 min (batería + alineación manual) | 81,4 min | 120 min | 67,8 % |
| B | Rodeo iluminado | 317 m | 66,0 min | 28 min (panel + kit) | 94,0 min | 180 min | 52,2 % |
| B sin O2-RESERVE | Rodeo iluminado | 317 m | 66,0 min | 28 min | 94,0 min | 120 min | 78,4 % → **W4** |

Actividades = observar 2 + escanear 3 (si hay SCANNER) + muestra 3 + caché 6 + relé + entregar 2.

La estimación termina en la entrega (POI-06). La vuelta al módulo para `RequestAscent` (≈ 7 m, ≈ 1,5 min) no se suma: queda cubierta por el colchón del aviso de soporte vital.

**Riesgo de balance abierto:** con desvíos típicos ([diseño de nivel](DISENO-DE-NIVEL.md)), A termina con unos 8 min de margen y B sin O2-RESERVE llega al regreso de emergencia antes de entregar. Es la lección que anuncia W4, pero puede ser demasiado exigente para un jurado que juega por primera vez. Primer ajuste si las pruebas lo confirman: subir `lifeSupportBaseMin` (por ejemplo, a 135) y volver a calcular W4.

En las 43 combinaciones que superan V1 y V4:

- Toda combinación sin LAMP y sin O2-RESERVE activa W4 (70,9–82,5 %).
- Ninguna combinación con LAMP o con O2-RESERVE lo activa (como máximo, 68,6 % y 55,0 %).

Lección: sin atajo por la sombra, hace falta reserva de soporte vital.

**Energía de equipo de A:**

| Consumo | Energía |
| --- | --- |
| Lámpara: 80 m en sombra ≈ 16,7 min × 80 W | ≈ 22,2 Wh |
| Escáner | 1,5 Wh |
| Relé | 100 Wh |
| **Total** | **123,7 de 460 Wh** |

B solo gasta los 1,5 Wh del escáner, de 60 Wh, porque el relé usa el panel.

## Parámetros técnicos aceptados de otras esquemáticas

| Nombre | Valor | Unidad | Origen |
| --- | --- | --- | --- |
| hudUpdateHz | 10 | Hz | Estructura de código |
| lumaReactionMinGapS | 8 | s reales | Mecánicas (separación global entre burbujas) |
| lumaBubbleS | 5 | s reales | Mecánicas |
| lumaReactionCooldownS | 20 | s reales | Estructura de código (repetición de la misma reacción) |
| relayAlignPositions | 8 | posiciones | Mecánicas (dial de alineación manual) |
| reportDeltaNoteFraction | 0.10 | fracción | Mecánicas (diferencia previsto/real que merece explicación) |
| autosaveMinIntervalS | 2 | s | Estructura de código |
| saveSchemaVersion | 1 | entero | Estructura de código |
| saveBackupCount | 3 | copias | Estructura de código |
| importMaxBytes | 262144 | bytes | Estructura de código |
| testTimeScaleMax | 50 | × | Estructura de código (solo build de pruebas) |
| targetFps / reducedFps | 60 / 30 | FPS | [Aceptación](../../07-Calidad/ACEPTACION-Y-PRUEBAS.md) |
| initialLoadBudgetMB | 15 | MB comprimidos | Aceptación |
| tabMemoryBudgetMB | 500 | MB | Aceptación |
| approachStartHeadingDeg | 90 | ° | Físicas |
| landingSiteHeadingDeg | 90 | ° | Físicas (orientación marcada del lugar de descenso) |
| landingDescentDeltaVMps | 15 | m/s | Físicas (parte de `approachNominalDeltaVMps`: 25 de pilotaje + 15 de descenso automático) |
| assistDoneSpeedMps | 0.05 | m/s | Físicas |
| flightScalePxPerM | 0.5 / 2 / 8 | px/m | Físicas (escala de la vista de vuelo según distancia) |
| flightScaleSwitchM | 300 / 60 | m | Físicas |
| lumaFollowHysteresisM | 0.4 | m | Físicas |
| lumaStuckS | 1.0 | s | Físicas |
| lumaRegroupFadeS | 0.2 | s | Físicas |
| cameraFollowRatePerS | 8 | 1/s | Físicas |
| cameraZoomLevels | 1, 2 | × | Físicas |
| occluderAlpha | 0.4 | — | Físicas |
| occluderFadeS | 0.15 | s | Físicas |
| collisionEpsilonM | 1e-6 | m | Físicas |
| simStepBudgetMs | 1.0 | ms | Físicas (presupuesto estimado, por medir) |
| maxSurfaceDynamicEntities | 32 | entidades | Físicas |
| returnCargoLogKg | 1 | kg | Mecánicas (parte de `returnCargoKg` que es el registro) |
| returnCargoSampleKg | 2 | kg | Mecánicas (parte de `returnCargoKg` que es la muestra) |
| sunFromDirection | W | dirección | Diseño de nivel (sol rasante desde el oeste) |
| poiApproachRadiusM | 6 | m | Diseño de nivel (radio de llegada a un lugar, para tutorial y autosave) |
| heightLevel | 0 / +1 / +2 | niveles | Diseño de nivel (con rampas; sin niveles negativos) |
| lampVisualRadiusM | 3 | m | Greybox |
| uiMinFontPx | 16 | px | Greybox |
| blinkHz | 1 | Hz | Greybox (con alternativa sin animación) |
| outlinePx | 1 / 2 | px | Greybox |
| shadowHatchSpacingPx | 4 | px | Greybox |

Punto de regreso en superficie: **DEMO-POI-01** (módulo). El aviso de soporte vital y `EmergencyReturn` miden la distancia hasta él, y `RequestAscent` se hace allí. La entrega se hace en DEMO-POI-06, a unos 7 m.

ID de la región de la demo: `MAP-DEMO-01`.

## Historial

- 2026-10-19: creación con valores provisionales.
- 2026-10-19: reequilibrio tras la revisión de mecánicas. TANK-S 90 → 96 kg (antes nunca era válido); `lifeSupportBaseMin` 150 → 120 (W4 no se activaba nunca); `solarPanelW` 300 → 600 (el kit no aportaba nada con panel); reserva en vuelo explícita; parámetros de crucero, secuencias y técnicos.
