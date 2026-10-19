---
id: fisicas-demo
type: diseno
status: especificado-sin-implementar
updated: 2026-10-19
tags: [orbita, demo, fisicas]
---
# Físicas de la demo

Esquemática técnica del bucle de simulación, del crucero acelerado, del vuelo local de aproximación, del movimiento y las colisiones en superficie y de la proyección isométrica. **No hay código.** Las fórmulas y el pseudocódigo describen reglas que se implementarán en la ventana autorizada ([ADR 003](../../03-Arquitectura/ADR/ADR-003.md)).

Todos los nombres y valores salen de [parámetros de la demo](PARAMETROS-DEMO.md). Los parámetros técnicos que introdujo esta esquemática ya figuran allí, en «Parámetros técnicos aceptados de otras esquemáticas», y aquí solo se citan por su nombre. Contexto: [GDD de la demo](GDD-DEMO.md), [mecánicas](MECANICAS.md), [diseño de nivel](DISENO-DE-NIVEL.md), [viaje espacial](../VIAJE-ESPACIAL.md), [arquitectura](../../03-Arquitectura/ARQUITECTURA.md), [ADR 001](../../03-Arquitectura/ADR/ADR-001.md), [ADR 004](../../03-Arquitectura/ADR/ADR-004.md), [guardado](../../03-Arquitectura/GUARDADO.md) y [datos y realismo](../../04-Ciencia/DATOS-Y-REALISMO.md).

**Advertencia de realismo.** Las trayectorias y maniobras son un modelo educativo simplificado. No se ofrecen como cálculo operativo real. No hay n-cuerpos ni Sistema Solar en una escena ([ADR 004](../../03-Arquitectura/ADR/ADR-004.md)).

## Constantes derivadas

No son parámetros nuevos. Se calculan a partir de [PARAMETROS-DEMO](PARAMETROS-DEMO.md) y se usan en todo el documento.

| Símbolo | Definición | Valor |
| --- | --- | --- |
| `dt` | `1 / simHz` | 1/60 s ≈ 0,016667 s |
| `ve` | `ispS × g0` | 311 × 9,80665 ≈ 3049,9 m/s |
| `mdotMain` | `mainThrustN / ve` | 1500 / 3049,9 ≈ 0,4918 kg/s |
| `mdotRcs` | `rcsThrustN / ve` | 200 / 3049,9 ≈ 0,06558 kg/s |
| `cAscent` | `1 − exp(−ascentDeltaVMps / ve)` | 1 − exp(−0,19673) ≈ 0,17859 |
| `kReserve` | `cAscent × (1 + propellantMarginHard)` | 0,17859 × 1,15 ≈ 0,20538 |
| `cruiseSteps` | `cruiseRealSeconds × simHz` | 30 × 60 = 1800 pasos |
| `landingHoldSteps` | `landingHoldS × simHz` | 2,0 × 60 = 120 pasos |
| `kx`, `ky` | `isoTileWidthPx / (2 × tileSizeM)`, `isoTileHeightPx / (2 × tileSizeM)` | 16 px/m y 8 px/m |
| `missionSecPerStep` | `surfaceTimeScale × dt` | 20/60 ≈ 0,3333 s de misión por paso |

## 1. Principios

1. **Simulación separada del render.** La simulación es TypeScript puro, sin Phaser, React ni red ([arquitectura](../../03-Arquitectura/ARQUITECTURA.md)). Recibe comandos y tiempo en pasos fijos y produce estado y eventos. Phaser 4 (WebGL) solo presenta: interpola, ordena profundidad, anima y aplica efectos. Ninguna animación concede progreso ni decide un resultado ([GDD](GDD-DEMO.md)).
2. **Unidades SI** en todo el estado: metros, segundos, kilogramos, newtons y m/s. Los ángulos del estado se guardan en grados normalizados `[0, 360)` por legibilidad y serialización; las funciones trigonométricas convierten a radianes. Los píxeles solo existen en la presentación.
3. **Marcos separados.** Cada modo tiene su propio marco y su propio estado. Nunca se mezclan ([ADR 004](../../03-Arquitectura/ADR/ADR-004.md)).

| Modo | Marco | Estado físico |
| --- | --- | --- |
| CRUISE | Escalar: progreso del tramo `f ∈ [0, 1]` | `cruiseStep`, hitos resueltos, propelente. Sin posición. |
| APPROACH | 2D local relativo al objetivo, objetivo en el origen. `+y` = dirección nominal de llegada; `+x` a su derecha. En pantalla, `+y` hacia arriba. | `r`, `v`, `θ`, propelente; masa derivada. |
| SURFACE | Lógico isométrico: origen en la esquina noroeste del mapa, `x` hacia el este, `y` hacia el sur, en metros. 112 × 88 m. | Posiciones y velocidades de jugador y LUMA, celda actual, reloj de misión. |
| LANDING, ASCENT | Sin marco: secuencias automáticas resueltas en un paso. | Solo masas y propelente. |

El marco de vuelo es un esquema de la aproximación final. No está georreferenciado con el mapa de superficie (simplificación S).

### Conversión al cambiar de modo

| Transición | Qué se conserva | Qué se descarta o se fija |
| --- | --- | --- |
| PLANNER → CRUISE | Masa seca, carga de módulos (`payloadKg`) y tanque elegido. | `f = 0`; propelente = capacidad del tanque. |
| CRUISE → APPROACH | Propelente tras la corrección. | La posición **no** se hereda del crucero: el estado inicial de vuelo es fijo (§4.6). Se escribe CP-APPROACH. |
| APPROACH → LANDING → SURFACE | Propelente menos el descenso automático (§4.9). | `r`, `v` y `θ` se descartan. El origen del marco de vuelo corresponde al módulo en DEMO-POI-01 (20, 70). El jugador aparece en el punto libre definido por [diseño de nivel](DISENO-DE-NIVEL.md). Se escribe CP-SURFACE. |
| SURFACE → ASCENT | Propelente, masa y carga de retorno realmente llevada. | Solo con `RequestAscent` en DEMO-POI-01 o con `EmergencyReturn` (§10). La posición de superficie no interviene en el cálculo. |

La masa nunca se guarda por separado: `massKg = landerDryMassKg + payloadKg + propellantKg`. Así no pueden divergir masa y propelente.

## 2. Bucle de paso fijo

### Reglas

- **Paso fijo:** `dt = 1/simHz`. La simulación nunca usa la duración del frame.
- **Acumulador:** cada frame suma su duración real; se ejecutan pasos mientras quede al menos `dt`, hasta `maxSubstepsPerFrame` (5). Si sobra tiempo, se descarta y se contabiliza como ralentización. Con 5 subpasos, la simulación va a tiempo real hasta frames de 83,3 ms (12 FPS); por debajo, va más lenta que el reloj en vez de entrar en espiral de muerte.
- **Interpolación:** la presentación dibuja `lerp(prev, curr, alpha)` con `alpha = acc / dt ∈ [0, 1)`. Los ángulos se interpolan por el camino corto. Una teletransportación (reubicación de LUMA, reintento, cambio de modo) marca `teleported` y no se interpola.
- **Pausa:** `PAUSE`, `DIALOGUE`, `JOURNAL`, `MAP` o un campo de texto con foco congelan la simulación. No se acumula tiempo y, al reanudar, se reinicia la marca del frame: no hay ráfaga de recuperación. Los controles mantenidos se leen como cero mientras otro elemento tiene el foco.
- **Pestaña oculta:** el evento de visibilidad pone la pausa, solicita un guardado del estado estable ([guardado](../../03-Arquitectura/GUARDADO.md)) y, al volver, muestra la superposición `PAUSE`. El jugador reanuda manualmente. Importa sobre todo en la aproximación.
- **Modo de 30 FPS:** la simulación sigue a 60 Hz; se ejecutan dos subpasos por frame.

### Determinismo

- `step(state, commands)` es una función pura del estado y de los comandos de ese paso. Con el mismo estado inicial, `rngSeed` y registro de comandos, el estado final es idéntico.
- Los comandos llevan el índice de paso en el que se aplican, no la hora del reloj. Las teclas mantenidas se muestrean en cada paso. Las pulsaciones (X, E, R, corregir) se encolan y las consume el primer paso siguiente.
- Prohibido `Math.random`, `Date.now` o `performance.now` dentro de la simulación. El PRNG con semilla se divide en flujos independientes derivados de `rngSeed` y de un identificador de flujo, de modo que añadir una variación cosmética no altera otros flujos. La física de la demo no consume aleatoriedad. Las variaciones cosméticas de la presentación usan su propio PRNG y no tocan el estado de la simulación.
- Temporizadores en **pasos enteros** (`cruiseStep`, `holdSteps`, reloj de misión), no en segundos flotantes acumulados.
- El determinismo bit a bit está garantizado en el mismo motor de JavaScript (V8 en Vitest y en Chrome). Entre motores distintos, `Math.exp` o `Math.sin` pueden diferir en el último bit. Las pruebas entre motores usan tolerancia.

### Orden estable de sistemas en un paso

1. Aplicar comandos encolados para este paso.
2. Según el modo:
   - CRUISE: `cruiseSystem`.
   - APPROACH: `flightControl` → `flightIntegrate` → `flightChecks`.
   - SURFACE: `playerMove` → `playerCollide` → `lumaSystem` → `interactQuery` → `environmentFlags` (celda, sombra, lámpara) → `missionClock`. Las reglas de soporte vital y energía de [mecánicas](MECANICAS.md) se ejecutan después, en el mismo paso.
3. Emitir eventos en orden `(stepIndex, orden del sistema, secuencia)`.
4. `stepIndex += 1`.

### Pseudocódigo

```text
onFrame(nowMs):
  if paused:
    lastMs = nowMs                    # sin acumulación ni recuperación posterior
    render(curr, alpha = 0)
    return
  acc += (nowMs - lastMs) / 1000
  lastMs = nowMs
  n = 0
  while acc >= dt and n < maxSubstepsPerFrame:
    prev = interpolables(curr)
    curr = step(curr, commandQueue.take(curr.stepIndex))
    acc -= dt
    n += 1
  if acc >= dt:                       # se alcanzó el límite
    metrics.droppedSeconds += acc - (acc mod dt)
    acc = acc mod dt
  render(interpolate(prev, curr, alpha = acc / dt))

onVisibilityHidden():
  paused = true
  requestSave(stableSnapshot(curr))
  showOverlay(PAUSE) al volver
```

## 3. Crucero acelerado

### Modelo por tramo

El crucero es un tramo parametrizado, no una integración física ([ADR 004](../../03-Arquitectura/ADR/ADR-004.md)). Avanza un contador entero:

- `cruiseStep` de 0 a `cruiseSteps` (1800). Avanza 1 por paso sin pausa.
- Progreso `f = cruiseStep / cruiseSteps`.
- Tiempo real sin pausas = `f × cruiseRealSeconds` (30 s).
- Tiempo físico mostrado = `f × cruisePhysicalHours` (76 h). Ejemplo: en `f = 0,5` se muestran 38 h. Es una referencia histórica redondeada de Apollo 11 ([NASA-APOLLO11](../../04-Ciencia/FUENTES.md)), no una trayectoria calculada.

No hay deriva de posición ni consumo continuo. El único consumo del tramo es la corrección a mitad de crucero.

### Hitos

La regla de la decisión está en [mecánicas](MECANICAS.md) y los tiempos en PARAMETROS: `midcourseDecisionOpenS` (8 s) y `midcourseDecisionCloseS` (20 s). Aquí se traducen a fracciones y pasos enteros: `paso = round(s × simHz)` y `f = paso / cruiseSteps`.

| Hito | Tiempo real | Fracción | Paso | Tiempo físico mostrado | Efecto |
| --- | --- | --- | --- | --- | --- |
| Se abre la decisión | `midcourseDecisionOpenS` = 8 s | 0,267 | 480 | 20,3 h | «Corregir ahora (2 kg)» / «Más tarde (5 kg)». |
| Se cierra la decisión | `midcourseDecisionCloseS` = 20 s | 0,667 | 1200 | 50,7 h | Si sigue pendiente, se aplica `midcourseLateKg` automáticamente. No hay fallo. |
| Escena de LUMA | A petición | — | — | — | `LUMA-R-CRUISE-REST` ([mecánicas](MECANICAS.md)): superposición `DIALOGUE` que pausa el crucero y no cambia recursos. Una vez por intento. |
| Llegada | 30 s | 1,00 | 1800 | 76 h | Cambio a APPROACH y escritura de CP-APPROACH. |

«Corregir ahora» durante la ventana cuesta `midcourseNowKg` (2 kg) en ese paso. «Más tarde» o no responder cuesta `midcourseLateKg` (5 kg) en el paso de cierre. Con la opción `pauseOnDecisions`, el crucero se congela mientras la ventana está abierta (`cruiseStep` no avanza) y la corrección tardía solo se aplica si el jugador la elige.

La escena de LUMA en el crucero es siempre a petición. No ocupa ningún hito del tramo.

### Consumo idempotente

```text
estado del tramo: cruiseStep, midcourseResolved ∈ {pending, now, late}, firedMilestones (conjunto de IDs)

cruiseSystem(paso):
  if comando CorrectNow and midcourseResolved == pending and openStep <= cruiseStep < closeStep:
     applyMidcourse(now)
  if midcourseResolved == pending and cruiseStep >= closeStep:     # «Más tarde» o sin respuesta
     applyMidcourse(late)
  for hito in hitos con step == cruiseStep y no en firedMilestones:
     dispararHito(hito); firedMilestones.add(hito.id)
  if cruiseStep == cruiseSteps: entrarEnAproximacion(); return
  if not (pauseOnDecisions and ventanaAbierta): cruiseStep += 1

applyMidcourse(kind):
  kg = kind == now ? midcourseNowKg : midcourseLateKg
  propellantKg -= kg                    # en el mismo paso que cambia el indicador
  midcourseResolved = kind
  emitir MidcourseCorrected { kind, kg }
  solicitar guardado de hito
```

- El consumo y el indicador `midcourseResolved` cambian en el mismo paso y forman parte de la misma instantánea. Si un guardado no termina, la partida restaurada no contiene ni el consumo ni el indicador: no hay doble consumo ni consumo perdido.
- Un `MidcourseCorrected` repetido (recarga, doble pulsación) no se aplica porque `midcourseResolved ≠ pending`.
- La pausa no avanza `cruiseStep` y el consumo es por evento, no por tiempo; pausar y reanudar no puede duplicar nada.
- La corrección se expresa en kilogramos fijos (F), sin Δv asociado.

### Guardado de fase

Se guardan `cruiseStep`, `midcourseResolved` y `firedMilestones` al disparar cada hito, al ocultar la pestaña y al llegar. Al reanudar (`RESUME → CRUISE` del [GDD](GDD-DEMO.md)), se restauran `cruiseStep` y las decisiones ya aplicadas. Los hitos registrados no se repiten y no se vuelve a consumir. La escena de LUMA procesada también la registra el director narrativo.

## 4. Vuelo local de aproximación

### 4.1 Estado

| Campo | Unidad | Nota |
| --- | --- | --- |
| `r = (x, y)` | m | Posición relativa al objetivo, que está en el origen. |
| `v = (vx, vy)` | m/s | Velocidad relativa. |
| `thetaDeg` | ° | Rumbo; `0°` = `+x`, sentido antihorario. Vector de rumbo `h = (cos θ, sin θ)`; lateral izquierdo `l = (−sin θ, cos θ)`. |
| `propellantKg` | kg | La masa se deriva de él. |
| `assistActive` | — | Asistente X. |
| `holdSteps` | pasos | Pasos consecutivos cumpliendo las tres condiciones. |
| `inZonePrev` | — | Para detectar la entrada en la zona. |

Fuera del estado restaurable: contadores para el informe (intentos, segundos de asistente, propelente gastado con asistente).

### 4.2 Integración (Euler semi-implícito)

```text
flightControl:
  sp = Mayús ? precisionFactor : 1
  giro = (A ? +1 : 0) + (D ? -1 : 0)            # A = antihorario (izquierda en pantalla)
  thetaDeg = wrap360(thetaDeg + giro × turnRateDegPerS × sp × dt)
  h, l = vectores del rumbo nuevo
  if assistActive: Fmain, Fretro, Flat = asistente(v, h, l, m)   # §4.4
  else:
    Fmain  = (W y no S) ? mainThrustN × sp : 0
    Fretro = (S y no W) ? rcsThrustN × sp : 0
    Flat   = (Q y no E) ? +rcsThrustN × sp : (E y no Q) ? -rcsThrustN × sp : 0
  if propellantKg <= 0: Fmain = Fretro = Flat = 0

flightIntegrate:
  m = landerDryMassKg + payloadKg + propellantKg   # masa al inicio del paso
  F = h × (Fmain - Fretro) + l × Flat
  mdot = (Fmain + Fretro + |Flat|) / ve × (assistActive ? assistPropellantPenalty : 1)
  v = v + (F / m) × dt                               # primero la velocidad
  rPrev = r
  r = r + v × dt                                     # después la posición, con v nueva
  propellantKg = max(0, propellantKg - mdot × dt)
```

- **Motor principal** en la dirección del rumbo (W). **RCS** retro en `−h` (S) y lateral en `±l` (Q izquierda, E derecha). Teclas opuestas a la vez se anulan y no consumen.
- **Giro directo** a `turnRateDegPerS` (1,5° por paso; 0,375° con precisión), sin momento angular ni consumo (S). Girar 180° tarda 2,0 s.
- **Precisión** (Mayús) multiplica empuje y giro por `precisionFactor`. El consumo por unidad de Δv no cambia, porque `ṁ = F/ve`. Un paso de motor da 0,0546 m/s a 458 kg; con precisión, 0,0136 m/s.
- **Mismo Isp para motor y RCS** (S): frenar un Δv dado cuesta el mismo propelente con cualquiera de los dos. Solo cambia el tiempo (el RCS tarda 7,5 veces más). La masa decreciente reproduce la ecuación del cohete, salvo con el asistente (§4.4).
- **Sin gravedad en el plano** (S). El modelo solo contiene velocidad relativa y empuje. Soltar los mandos conserva el movimiento ([viaje espacial](../VIAJE-ESPACIAL.md)).

### 4.3 Ejemplo numérico: frenar 18 m/s con el motor a 458 kg

Configuración A con TANK-M: masa seca 300 + carga 40 + propelente 118 (120 − 2 de corrección a tiempo) = 458 kg.

| Magnitud | Cálculo | Resultado |
| --- | --- | --- |
| Aceleración inicial | 1500 / 458 | 3,275 m/s² (≈ 3,28) |
| Propelente (ecuación del cohete) | 458 × (1 − exp(−18 / 3049,9)) | 2,695 kg |
| Tiempo exacto con masa decreciente | 2,695 / 0,4918 | 5,48 s |
| Tiempo a masa constante | 18 / 3,275 | 5,50 s |
| Distancia de frenado | 18² / (2 × 3,275) | ≈ 49,5 m |
| Simulación discreta a 60 Hz | Euler semi-implícito | 329 pasos (5,483 s); 2,697 kg; 49,2 m; rebasa 0,011 m/s |

Antes de frenar hay que girar 180° (2,0 s), en los que la nave avanza 36 m más a 18 m/s. Frenar 18 m/s con el RCS retro sin girar cuesta lo mismo (2,695 kg) pero dura 41,2 s y recorre unos 371 m. Cancelar la deriva lateral de 2,5 m/s con RCS cuesta 0,375 kg en 5,7 s.

### 4.4 Asistente «igualar velocidad» (X)

- X activa o desactiva el asistente. Se cancela al pulsar W, S, Q o E, al llegar a `|v| ≤ assistDoneSpeedMps` (0,05 m/s) o al terminar la aproximación. A y D siguen girando la nave: orientar el motor contra la velocidad es decisión del jugador.
- Aceleración deseada: `aDes = −assistGainPerS × v` (0,8 1/s; constante de tiempo 1,25 s).
- Se descompone `FDes = m × aDes` en ejes de la nave: `fPar = FDes·h` y `fLat = FDes·l`. Capacidades: `fPar ≤ mainThrustN` si es positiva (motor principal con empuje continuo entre 0 y 1, simplificación S) o `|fPar| ≤ rcsThrustN` si es negativa (retro); `|fLat| ≤ rcsThrustN`.
- Se aplica un único factor `s = min(1, capPar/|fPar|, capLat/|fLat|)` a ambos ejes. La fuerza conserva la dirección de `−v` y la trayectoria no se curva.
- El empuje físico es el real. Solo el consumo se multiplica: `ṁ = (|fPar·s| + |fLat·s|) / ve × assistPropellantPenalty`.

| Caso (458 kg, desde la geometría inicial) | Duración | Propelente | Equivalente manual |
| --- | --- | --- | --- |
| Nave girada (θ = 270°), `v = (0, 18)` | ≈ 9,7 s (583 pasos) | ≈ 2,96 kg | 2,70 kg × 1,10 |
| Nave girada, `v = (2,5, 18)` | ≈ 9,9 s | ≈ 3,36 kg | (2,70 + 0,37) × 1,10 |
| Morro hacia el objetivo (θ = 90°), `v = (2,5, 18)` | ≈ 42,8 s; recorre ≈ 374 m | ≈ 3,36 kg | Solo dispone del RCS retro |

En el segundo caso limita el RCS lateral: `s = min(1500/6595, 200/916) = 0,218`. El motor trabaja al 96 %.

### 4.5 Reserva de propelente en vuelo

Regla de [PARAMETROS](PARAMETROS-DEMO.md): `ascentReserveKg(t) = propellantForDeltaV(m(t) + returnCargoKg, ascentDeltaVMps) × (1 + propellantMarginHard)`. Si el propelente baja de `ascentReserveKg(t)`, se produce `ApproachAborted { reason: reserve }`. Se usa siempre `returnCargoKg` completo (caso conservador).

En vuelo, `M_seco = landerDryMassKg + payloadKg` es constante y la masa es `M_seco + p`. La condición `p < kReserve × (M_seco + p + returnCargoKg)` equivale al umbral fijo de PARAMETROS:

`p* = kReserve × (M_seco + returnCargoKg) / (1 − kReserve)`;   sobrante para corregir = `p − p*`

| Configuración | `M_seco` | Masa al empezar | `p*` | Propelente al inicio (a tiempo / tarde) | Sobrante (a tiempo / tarde) | Δv equivalente |
| --- | --- | --- | --- | --- | --- | --- |
| A · TANK-M | 340 kg | 458 kg | 88,65 kg | 118 / 115 kg | 29,35 / 26,35 kg | ≈ 202 / 182 m/s |
| B · TANK-M | 351 kg | 469 kg | 91,49 kg | 118 / 115 kg | 26,51 / 23,51 kg | ≈ 177 / 158 m/s |
| BATTERY-PACK + LAMP · TANK-S | 332 kg | 426 kg | 86,58 kg | 94 / 91 kg | 7,42 / 4,42 kg | ≈ 54 / 32 m/s |

Las cifras coinciden con las de PARAMETROS (recalculadas: 88,650; 91,493; 86,583 kg). La reserva se reduce mientras se quema propelente, porque baja la masa. Por eso el sobrante real es mayor que la diferencia instantánea (en A, 29,35 kg frente a 118 − 94,68 = 23,32 kg). El HUD muestra el sobrante real.

**El sobrante es positivo al empezar en toda configuración válida.** De las 93 combinaciones que superan V1–V5 con los tanques actuales, la más ajustada es SOLAR-PANEL + O2-RESERVE con TANK-S (33 kg de carga, margen del 15,2 %): 7,16 kg con corrección a tiempo (≈ 52 m/s) y 4,16 kg con corrección tardía (≈ 30 m/s). Basta para el frenado mínimo (≈ 18,2 m/s) y la aproximación nunca se aborta en el primer paso. Las misiones ligeras con TANK-S obligan a pilotar con cuidado y a usar poco el asistente; con TANK-M el sobrante perdona un pilotaje torpe. Ver T-FIS-18.

### 4.6 Condiciones de inicio (geometría exacta)

| Magnitud | Valor |
| --- | --- |
| Posición `r0` | `(0, −approachStartDistM)` = (0, −400) m |
| Velocidad `v0` | `(approachStartLateralMps, approachStartClosingMps)` = (2,5, 18) m/s |
| Rumbo `θ0` | `approachStartHeadingDeg` = 90° (morro hacia el objetivo) |
| Propelente | El del final del crucero |
| `assistActive`, `holdSteps` | `false`, 0 |

Sin intervención, la nave pasa a 55,0 m del objetivo a los 21,8 s (no entra en la zona) y se pierde a los 65,7 s (paso 3944), al superar `lostDistanceM`.

### 4.7 Condiciones de descenso

Se evalúan cada paso, después de integrar. Son inclusivas:

1. **Zona:** `|r| ≤ landingZoneRadiusM` (12 m).
2. **Velocidad:** `|v| ≤ landingMaxSpeedMps` (1,0 m/s).
3. **Rumbo:** `|wrap180(θ − landingSiteHeadingDeg)| ≤ landingHeadingTolDeg` (15°). La orientación del lugar es la marcada en el suelo: `landingSiteHeadingDeg` = 90°, alineada con la llegada. Con ese valor, quien frena girando la nave debe volver a girar 180° antes de descender.

`holdSteps` aumenta mientras se cumplan las tres y vuelve a 0 en cuanto falla una. Al llegar a `landingHoldSteps` (120), el modo pasa a LANDING. Durante esos 2 s, a 1 m/s como máximo, la nave se desplaza 2 m como máximo.

### 4.8 Fallos y reintento

Orden de comprobación en `flightChecks` (fijo; la reserva va primero porque la demostración del ascenso, §10, lo necesita):

1. **Reserva:** `propellantKg < p*` → `ApproachAborted { reason: reserve }`.
2. **Exceso de velocidad al entrar:** el segmento `rPrev → r` entra en el disco de la zona (prueba segmento-círculo, de modo que ni un paso rápido ni un roce se escapan) y `|v| > overspeedFailMps` (4,0 m/s, estricto) → `ApproachFailed { reason: overspeed }`. Solo se evalúa al entrar; dentro de la zona se puede acelerar sin fallo, pero entonces no se cumple la condición 2.
3. **Pérdida:** `|r| > lostDistanceM` (800 m) → `ApproachFailed { reason: lost }`.
4. **Descenso:** condiciones de §4.7.

Abortar voluntariamente produce `ApproachAborted { reason: player }`.

**Reintento.** Tras `ApproachFailed` o `ApproachAborted` (por reserva o voluntario), el [GDD](GDD-DEMO.md) ofrece dos opciones: reintentar desde CP-APPROACH o terminar e ir al informe (REPORT). Se restaura la instantánea completa del punto de control (estado inicial §4.6 y propelente de ese momento); el propelente gastado en el intento fallido desaparece con él. La instantánea no se modifica al reintentar y el consumo no se aplica dos veces. Solo el contador de intentos del informe persiste. Si se reanuda una partida guardada durante la aproximación, se reanuda desde CP-APPROACH: no se guarda un estado de vuelo intermedio.

### 4.9 Descenso final automático

LANDING es una secuencia automática. En el paso de transición, la simulación:

1. Aplica una sola vez `propellantForDeltaV(massKg, landingDescentDeltaVMps)` (15 m/s; ≈ 2,0–2,5 kg según la masa).
2. Emite `Landed` y escribe CP-SURFACE.

La animación dura `landingSequenceRealS` (4 s), se reproduce después y no altera el resultado. Si se recarga durante la animación, la partida se reanuda en CP-SURFACE.

El coste del descenso ya está incluido en `approachNominalDeltaVMps` (40 m/s), que se reparte en unos 25 m/s de pilotaje (mínimo físico ≈ 18,2 m/s de frenado y deriva, más correcciones) y 15 m/s de descenso. Con 458 kg, 25 m/s cuestan 3,74 kg y el descenso posterior, 2,23 kg: 5,97 kg en total, igual a los 40 m/s del planificador. No hay gravedad en el plano: el descenso es una abstracción con coste fijo en Δv (S).

## 5. HUD de vuelo (datos que expone la simulación)

La simulación publica una vista derivada de solo lectura. Phaser la lee en cada frame; React la recibe a `hudUpdateHz` (10 Hz), sin un render de React por frame ([arquitectura](../../03-Arquitectura/ARQUITECTURA.md)).

| Campo | Definición |
| --- | --- |
| `distanceM` | `|r|` |
| `relSpeedMps` | `|v|` |
| `closingMps` | `−(r·v)/|r|`; positivo si se acerca. |
| `lateralMps` | `v · (uy, −ux)` con `u = −r/|r|`: deriva hacia la derecha de la línea de mira. |
| `headingDeg`, `siteHeadingDeg`, `headingErrorDeg` | `θ`, `landingSiteHeadingDeg` y `wrap180(θ − sitio)`. |
| `propellantKg` | Propelente total. |
| `reserveKg` | `p*` (umbral fijo de aborto). |
| `spareKg` | `propellantKg − p*`: «sobrante para corregir». |
| `stopDistanceMainM` | `|v|² / (2 × mainThrustN / m)`: ayuda para decidir cuándo frenar. |
| `conditions` | `inZone`, `speedOk`, `headingOk` y `holdProgress = holdSteps / landingHoldSteps`. |
| `assistActive` | Estado del asistente; el HUD recuerda su penalización (×1,10). |
| `overspeedRisk` | `true` si `|v| > overspeedFailMps` y `stopDistanceMainM > |r| − landingZoneRadiusM`. Es un aviso de presentación, no una regla. |

El HUD no muestra la reserva como un número oculto: si `spareKg` baja de cero, la simulación ya ha abortado.

## 6. Movimiento en superficie

### 6.1 Entrada de pantalla a ejes lógicos

Se elige **W = arriba en pantalla**. El jugador espera que W suba el personaje en la pantalla, no que avance en diagonal por la rejilla. Con la proyección de §8, «arriba en pantalla» es el **noroeste lógico** `(−1, −1)/√2`.

Vector de entrada en ejes de pantalla: `i = (D − A, S − W)`, con `y` de pantalla hacia abajo. Dirección lógica:

```text
d = M · i / |i|,   M = (1/√2) · [ [ 1,  1 ],
                                  [-1,  1 ] ]
```

| Teclas | `i` | Dirección lógica `d` | En pantalla |
| --- | --- | --- | --- |
| W | (0, −1) | (−1, −1)/√2 noroeste | Recto arriba |
| S | (0, 1) | (1, 1)/√2 sureste | Recto abajo |
| D | (1, 0) | (1, −1)/√2 noreste | Recto a la derecha |
| A | (−1, 0) | (−1, 1)/√2 suroeste | Recto a la izquierda |
| W+D | (1, −1) | (0, −1) norte | Arriba a la derecha, por el borde del rombo (26,57°) |
| S+D | (1, 1) | (1, 0) este | Abajo a la derecha, por el borde del rombo |

`M` es una rotación de 45°, de modo que las teclas cardinales coinciden exactamente con las direcciones de pantalla y las diagonales siguen los ejes de la rejilla, que son los de los muros. **Normalización diagonal:** como `M` es ortonormal, normalizar `i` basta para que la velocidad sea la misma en las ocho direcciones. La velocidad lógica es uniforme; en pantalla, el movimiento vertical recorre la mitad de píxeles por metro (escorzo 2:1), y es correcto que así sea.

### 6.2 Dinámica

```text
playerMove:
  if |i| > 0:
     vTarget = walkSpeedMps × d
     v = v + clampLen(vTarget - v, walkAccelMps2 × dt)
  else:
     v = v × max(0, |v| - walkDecelMps2 × dt) / |v|     # frenado hacia 0
  # |v| <= walkSpeedMps siempre
```

| Magnitud | Cálculo | Resultado |
| --- | --- | --- |
| Alcanzar `walkSpeedMps` desde parado | 1,6 / 2,5 | 0,64 s (paso 39); 0,51 m |
| Parar al soltar | 1,6 / 3,0 | 0,533 s (32 pasos); 0,43 m |
| Desplazamiento máximo por paso | 1,6 / 60 | 0,0267 m |

`moonGravityMps2` (1,62 m/s², ≈ 0,165 g0; [NASA-MOON-FACTSHEET](../../04-Ciencia/FUENTES.md)) **no entra en ninguna ecuación de la demo**. Solo justifica en el texto la sensación de poca tracción que producen la aceleración y el frenado bajos. No hay saltos, caídas ni dinámica vertical.

**Altura.** La altura visual es un nivel entero por celda. Solo se cambia de nivel por celdas de rampa (altura visual interpolada a lo largo de la celda). Los bordes de una meseta son celdas bloqueantes, marcadas en el [diseño de nivel](DISENO-DE-NIVEL.md). La altura no altera velocidad ni consumo.

### 6.3 Tiempo de misión

En SURFACE, cada paso sin pausa suma `missionSecPerStep` (1/3 s de misión). Se guarda el número entero de pasos y los segundos de misión se derivan de él. Caminar 1 m a `walkSpeedMps` cuesta `minPerMeter` = 0,208 min de misión. Las actividades (`observeMin`, etc.) avanzan el mismo reloj; sus reglas están en [mecánicas](MECANICAS.md).

## 7. Colisiones

### 7.1 Formas

- Jugador: círculo de `playerRadiusM` (0,4 m). LUMA: círculo de `lumaRadiusM` (0,3 m).
- Celdas bloqueantes: AABB de `tileSizeM` × `tileSizeM`. Las celdas fuera del mapa se consideran bloqueantes.
- **Sombra sin lámpara:** sin MOD-LAMP, las celdas con indicador de sombra son no transitables y se resuelven exactamente igual que un muro. La simulación publica `blockedBy = shadow` para que la interfaz explique el motivo. No hay daño ni consumo. Con lámpara se transitan.
- **Sombra sin energía:** con lámpara pero sin `availableWh` suficiente para cruzar el tramo, la sombra se bloquea igual, con `blockedBy = shadowEnergy`. La regla y el mensaje son de [mecánicas](MECANICAS.md).
- **No encerrar:** el bloqueo por sombra solo se aplica a celdas que el círculo no solapa en ese momento. Si la lámpara se apaga dentro de la sombra, el jugador puede salir pero no volver a entrar. Es la red de seguridad física bajo la «salida guiada» de mecánicas. Para esa salida, la simulación calcula la celda iluminada más cercana con el campo de distancias de §7.6 y conduce al jugador por él a `walkSpeedMps`, con el reloj de misión avanzando normalmente.

### 7.2 Resolución por ejes separados con deslizamiento

Candidatas: las celdas que solapan la AABB del círculo. Como el diámetro (0,8 m) es menor que una celda (2 m), son 4 como máximo.

```text
playerCollide(posición p, velocidad v, radio rad):
  # Pase X
  p.x += v.x × dt
  for celda bloqueante c en candidatas:
     if c.minY <= p.y <= c.maxY and solapa(p, rad, c):         # caso de cara
        p.x = v.x > 0 ? c.minX - rad : c.maxX + rad
        v.x = 0
  # Pase Y (simétrico, con p.x en [c.minX, c.maxX])
  # Pase de esquina
  for celda bloqueante c en candidatas:
     q = punto de c más cercano a p
     if q es un vértice de c and |p - q| < rad - collisionEpsilonM:
        n = (p - q) / |p - q|
        p = q + n × rad
        v = v - min(0, v·n) × n                                # quita solo la componente entrante
```

- **Deslizamiento:** cada pase anula solo la componente perpendicular a la cara; la paralela continúa. Contra un muro, el movimiento diagonal se convierte en deslizamiento a lo largo del muro.
- **Esquinas convexas:** el pase de esquina empuja según la normal vértice → centro. El círculo rodea la esquina en lugar de engancharse.
- **Sin enganches en uniones:** a lo largo de dos celdas contiguas, el vértice compartido queda a distancia ≥ `rad` tras el pase de cara, y el pase de esquina no actúa (con margen `collisionEpsilonM`).
- **Esquinas cóncavas:** los dos pases de cara las resuelven sin pase de esquina.

### 7.3 Sin atravesar a alta velocidad

Desplazamiento máximo por paso: jugador `δ = walkSpeedMps × dt = 0,0267 m`; LUMA `lumaSpeedMps × dt = 0,0317 m`.

1. `|v|` está limitada a `walkSpeedMps` (§6.2) y el paso es fijo. Un frame lento ejecuta más pasos, nunca un paso más largo (§2).
2. Al empezar cada paso no hay solapamiento, porque el paso anterior lo resolvió. Tras moverse, la penetración es como mucho `δ`.
3. Para atravesar una cara, el centro tendría que cruzar la línea de esa cara, lo que exige penetrar al menos `rad`. Como `δ / rad = 0,067` para el jugador y `0,106` para LUMA, la penetración es muy inferior a `rad`. El signo del movimiento indica la cara correcta y la resolución nunca empuja hacia el otro lado.
4. Una celda mide 2 m frente a 0,0267 m por paso: harían falta más de 75 pasos para cruzarla.

### 7.4 LUMA

LUMA no bloquea al jugador y no colisiona con él. Sí colisiona con las celdas bloqueantes y con la sombra sin lámpara, con el mismo algoritmo. Si se atasca, se reubica (§9); nunca queda atrapada.

### 7.5 Objetos interactuables

- Son círculos o huellas no bloqueantes. Si el objeto ocupa celdas bloqueantes (el relé, la terminal), su huella es esa AABB.
- Distancia de interacción: del centro del jugador al punto más cercano de la huella. Así se alcanza un objeto multicelda desde cualquier lado. Con una huella puntual en el centro de una celda bloqueante, una esquina quedaría a 1,81 m, fuera del alcance.
- E: candidatos con distancia `≤ interactRadiusM` (1,5 m). R: candidatos escaneables con distancia `≤ scanRadiusM` (3,0 m).
- Se selecciona el más cercano. En caso de empate, el ID menor en orden lexicográfico. La selección se publica para resaltar el objetivo en pantalla.

### 7.6 Distancia de regreso

El punto de regreso en superficie es **DEMO-POI-01** (módulo), según PARAMETROS. El aviso `LifeSupportWarning` (regla en [mecánicas](MECANICAS.md)) y `EmergencyReturn` miden la distancia hasta él, y `RequestAscent` solo se acepta allí, con el jugador a `interactRadiusM` o menos de la huella del módulo. La entrega se hace en DEMO-POI-06, a unos 7 m.

La simulación implementa `shortestAvailablePathM(posición, DEMO-POI-01)`. Es un campo de distancias calculado con Dijkstra **desde DEMO-POI-01** sobre las celdas transitables, con 8 vecinos (2 m en ortogonal y 2√2 m en diagonal) y sin cortar esquinas. La sombra cuenta como transitable solo con lámpara y energía para cruzarla. El campo se recalcula cuando cambia la transitabilidad (lámpara, energía); son dos variantes precalculables. La métrica octil supera a la euclídea en un 8 % como máximo y por eso es conservadora para el aviso. Tiempo de regreso = `distancia × minPerMeter`. Las distancias que dé el campo deben coincidir con las longitudes de ruta de PARAMETROS dentro de la tolerancia del ±10 % del [diseño de nivel](DISENO-DE-NIVEL.md).

`EmergencyReturn` no se simula como caminata: según mecánicas, es un traslado automático a DEMO-POI-01 sin consumo, seguido de ASCENT.

## 8. Proyección isométrica y profundidad

### 8.1 Fórmulas

Lógico (m) → mundo (px), con nivel de altura `h`:

```text
sx = kx × (x − y)                       kx = 16 px/m
sy = ky × (x + y) − heightStepPx × h     ky = 8 px/m
```

Mundo (px) → lógico, para una altura `h` dada:

```text
a = sx / kx
b = (sy + heightStepPx × h) / ky
x = (a + b) / 2
y = (b − a) / 2
```

Pantalla: `screen = zoom × (mundo − cámara) + (480, 270)`, siendo la cámara el punto del mundo en el centro de la resolución interna de 960 × 540.

Esquinas del mapa (`h = 0`): (0, 0) → (0, 0); (112, 0) → (1792, 896); (0, 88) → (−1408, 704); (112, 88) → (384, 1600). El rombo ocupa 3200 × 1600 px de mundo.

**Selección con ratón** (interfaz y depuración; la superficie se controla con teclado): se prueba `h` desde el nivel máximo hasta 0. Para cada valor, se calcula `(x, y)` y se acepta la primera celda cuyo nivel coincide con `h`.

### 8.2 Orden de profundidad

- **Capa de suelo:** sin ordenar, siempre detrás.
- **Clave:** `depthKey = xPie + yPie` en metros, proporcional a `sy` del pie. Un rebanado de terreno o de objeto estático en la celda `(i, j)` usa su centro: `tileSizeM × (i + j + 1)`.
- **Desempate estable:** `depth = depthKey + rangoTipo × 1e-3 + índice × 1e-6`, con rango de tipo terreno 0, objeto 1, interactuable 2, LUMA 3 y jugador 4, e índice estable menor que 1000.
- **Objetos multicelda:** se cortan en rebanadas por celda ([greybox](../../05-Arte/GREYBOX.md)). Cada rebanada se ordena con su celda. Una sola clave por objeto falla con entidades a su lado.
- **Entidad sobre una celda elevada:** `depthKey = max(xPie + yPie, claveCelda + 0,01)`. Restricción de nivel: una celda elevada transitable no puede tener una vecina más alta delante (`(i+1, j)`, `(i, j+1)`, `(i+1, j+1)`), salvo una rampa de subida. El [diseño de nivel](DISENO-DE-NIVEL.md) debe respetarla.

### 8.3 Atenuación de oclusores

Un objeto o rebanada se atenúa a `occluderAlpha` (0,4) durante `occluderFadeS` (0,15 s) si se cumplen las tres condiciones:

1. Está marcado como capaz de ocluir (su sprite es más alto que un rombo).
2. Se dibuja después del jugador o de LUMA (`depth` mayor).
3. Su caja en pantalla se solapa con la del sprite del jugador o de LUMA.

Solo se comprueban candidatos a 3 celdas o menos. Es una regla de presentación coherente con la [guía visual](../../05-Arte/GUIA-VISUAL.md).

### 8.4 Cámara

- **Seguimiento suavizado** en la presentación, con el tiempo del frame: `cam += (objetivo − cam) × (1 − exp(−cameraFollowRatePerS × dtFrame))`, con objetivo en la posición interpolada del jugador. Si la distancia es menor de 0,5 px, la cámara se ajusta al objetivo.
- **Límites:** `camX ∈ [−1408 + 480/zoom, 1792 − 480/zoom]` y `camY ∈ [−heightStepPx × hMax + 270/zoom, 1600 − 270/zoom]` (más los márgenes de los sprites). Las esquinas vacías del rombo muestran el fondo.
- **Zoom discreto:** `cameraZoomLevels` = {1, 2}, enteros para conservar los píxeles; 1 por defecto. El mapa completo es la superposición `MAP`, no un nivel de zoom.
- **Redondeo a píxel:** `camPx = round(cam)` y `spritePx = round(mundo) − camPx`, ambos en píxeles del mundo antes del zoom. La cámara y los sprites usan la misma regla para evitar temblores relativos.
- **Visibilidad:** con zoom 1 se ven unos ±21,2 m lógicos en la diagonal horizontal y ±23,9 m en la vertical; con zoom 2, la mitad.

## 9. Seguimiento de LUMA

LUMA es un animal extraterrestre con traductor, no un vehículo ([LUMA](../LUMA.md)). Su movimiento es presencia y lectura emocional: no resuelve rutas por el jugador.

| Estado | Entrada | Comportamiento |
| --- | --- | --- |
| FOLLOW | Por defecto | Con `d = |pJugador − pLUMA|`: parada, arranca si `d > lumaFollowDistM + lumaFollowHysteresisM` (2,0 m). En movimiento, va hacia el jugador con rapidez `lumaSpeedMps × clamp((d − lumaFollowDistM) / lumaFollowHysteresisM, 0, 1)`, sin inercia (S), y se detiene al llegar a `d ≤ lumaFollowDistM` (1,6 m). Resuelve colisiones como en §7. |
| WAIT | Actividad del jugador en curso (escanear, muestra, caché, relé, entrega) | Se detiene y se orienta hacia el objetivo de la actividad. Vuelve a FOLLOW al terminar. |
| REGROUP | `d > lumaRegroupDistM` (8 m), o atasco: en movimiento durante `lumaStuckS` (1,0 s) con un desplazamiento menor que el 25 % del esperado | Se teletransporta en un solo paso (ver más abajo) y vuelve a FOLLOW. |

**Reubicación determinista.** Se prueban 8 candidatos a `lumaFollowDistM` del jugador: detrás de su última dirección de movimiento y después girando ±45°, ±90°, ±135° y 180°, en ese orden. Se acepta el primero que esté dentro del mapa, sin solapar celdas bloqueantes ni sombra sin lámpara, y con línea directa al jugador por celdas transitables. Si ninguno sirve, LUMA se coloca sobre el jugador (no bloquea). La reubicación siempre tiene éxito. La presentación oculta el salto con un fundido de `lumaRegroupFadeS` (0,2 s). No se reubica «fuera de cámara»: a 8 m, LUMA está siempre dentro de cámara (§8.4).

**Convergencia.** Como la rapidez de LUMA crece con la separación, no avanza a tirones. Si el jugador camina en línea recta a 1,6 m/s, el equilibrio está en `d = 1,6 + 0,4 × 1,6/1,9 ≈ 1,94 m`. Cuando el jugador se detiene, LUMA recorre el último tramo frenando hasta 1,6 m. Su velocidad máxima supera a la del jugador en 0,3 m/s, así que no se queda atrás en terreno abierto.

**Sin búsqueda de caminos.** El mapa es abierto, con seis lugares y rutas diseñadas. El camino directo con deslizamiento resuelve el caso normal y la reubicación cubre los demás. Un A* en rejilla de 56 × 44 = 2464 celdas con heurística octil expandiría como máximo 2464 nodos. Se estima por debajo de 1 ms en el equipo de referencia (estimación, no medición). Solo se activaría, recalculado como mucho cada 0,5 s, si las pruebas con jugadores muestran reubicaciones frecuentes y visibles. Una alternativa más barata es reutilizar un campo de distancias como el de §7.6, centrado en el jugador.

## 10. Ascenso automático

**El ascenso nunca empieza al entregar.** Solo se entra en ASCENT de dos formas:

- con `RequestAscent`, que el jugador ordena en DEMO-POI-01 (con confirmación si el objetivo principal no está completo);
- con `EmergencyReturn`, cuando se agota el soporte vital.

El consumo se resuelve en un solo paso:

```text
mAscenso = landerDryMassKg + payloadKg + propellantKg + cargaRetornoReal   # 0–3 kg: muestra y registro realmente llevados
consumo  = propellantForDeltaV(mAscenso, ascentDeltaVMps) = cAscent × mAscenso
propellantKg -= consumo
emitir AscentCompleted (una vez; indicador persistente)
```

La validación (V5) y la reserva en vuelo suman siempre `returnCargoKg` completo. El ascenso usa la carga real, que es menor o igual, de modo que nunca necesita más de lo reservado. Se supone que todos los módulos regresan (caso más pesado). La superficie no consume propelente. Los `ascentSequenceRealS` (10 s) de animación no deciden nada.

### Demostración: el ascenso nunca falla si se respetó la reserva

Sea `M3 = M_seco + p + returnCargoKg` la masa de cálculo en el instante del descenso, con propelente `p`.

1. Al comprobar el descenso no hubo aborto, y la reserva se comprueba antes en ese mismo paso (§4.8): `p ≥ kReserve × M3 = 1,15 × cAscent × M3`.
2. El descenso consume `dd = (M_seco + p) × δ ≤ M3 × δ`, con `δ = 1 − exp(−landingDescentDeltaVMps / ve)`.
3. El ascenso necesita `cAscent × (M3 − dd)`. Queda `p − dd − cAscent × (M3 − dd) ≥ 0,15 × cAscent × M3 − dd × (1 − cAscent)`.
4. Ese resultado es `≥ 0` si `dd ≤ 0,15 × cAscent × M3 / (1 − cAscent)`, lo que se cumple siempre que `δ ≤ 0,15 × 0,17859 / 0,82141 = 0,03261`. Es decir, para cualquier `landingDescentDeltaVMps ≤ −ve × ln(1 − 0,03261) ≈ 101 m/s`.
5. Con `landingDescentDeltaVMps` = 15 m/s, `δ = 0,00491`, muy por debajo del límite.
6. Si la carga real es menor que `returnCargoKg`, la necesidad del ascenso disminuye y la desigualdad se mantiene. ∎

Casos peores numéricos (aterrizaje con `p = p*` y carga de 3 kg):

- A: descenso 2,10 kg → quedan 86,55 kg; el ascenso necesita 76,71 kg; sobran 9,84 kg.
- BATTERY-PACK + LAMP con TANK-S: sobran 9,61 kg.
- En todo el rango de carga válido (12–60 kg), sobran como mínimo 9,03 kg.

Con un aterrizaje nominal de A y TANK-M, tras el ascenso quedan 30,77 kg con 3 kg de carga o 31,31 kg sin carga. El ascenso no depende de la aleatoriedad ni de la animación.

## 11. Simplificaciones y fuentes

O = observado, S = simplificación de juego, F = ficción de diseño ([datos y realismo](../../04-Ciencia/DATOS-Y-REALISMO.md)). IDs de [fuentes](../../04-Ciencia/FUENTES.md).

| Elemento | Nivel | Declaración | Fuente |
| --- | --- | --- | --- |
| Gravedad lunar 1,62 m/s² | O | Dato real; no entra en las ecuaciones de la demo, solo justifica la baja tracción. | NASA-MOON-FACTSHEET |
| Duración física del crucero (76 h) | O | Referencia histórica redondeada (unas 75 h 50 min hasta la órbita lunar), no una trayectoria calculada. | NASA-APOLLO11 |
| Compresión del crucero a 30 s reales | S | Tiempo de juego frente a tiempo físico. | — |
| `g0` y ecuación del cohete | O | Física estándar: Δu = Isp · g0 · ln(MR), usada en `propellantForDeltaV`. | NASA-ROCKET-EQ; NASA-FLIGHT (fundamentos) |
| Isp 311 s, igual en motor y RCS | S | Valor de juego del orden de los motores de propelentes almacenables; no se atribuye a ningún motor concreto. Un solo Isp por simplicidad. | — (valor de juego) |
| Empujes, masas y corrección en kg | F | Vehículo ficticio. | — |
| Vuelo 2D sin gravedad, relativo al objetivo | S | Sin órbitas ni n-cuerpos; no es un cálculo operativo. | NASA-FLIGHT (contexto) |
| Giro sin momento angular | S | Giro directo a velocidad constante y sin consumo. | — |
| Asistente proporcional con penalización ×1,10 | S | El consumo extra rompe a propósito la ecuación del cohete para dar coste al asistente. | — |
| Descenso final automático con Δv fijo | S | Sin dinámica vertical. | — |
| Δv de ascenso de 600 m/s | S | No es el Δv real de un ascenso lunar, que es del orden de km/s. | — |
| Reserva del 15 % | S | Regla de seguridad de juego. | — |
| Marcha a 1,6 m/s con aceleración baja | S | Velocidad comprimida; sin saltos. | — |
| Colisiones por celdas de 2 m | S | Rejilla lógica; mapa comprimido y artístico. | — |
| Sombra no transitable sin lámpara | F | Regla de juego (frío y oscuridad), sin daño. | — |
| Tiempo de superficie ×20 | S | Compresión del reloj de misión. | — |
| Proyección isométrica 2:1 | S | Presentación; no altera la física. | — |
| LUMA | F | Licencia de ciencia ficción; no se atribuye a NASA. | — |

## 12. Presupuesto de rendimiento

Objetivo: 60 FPS en el equipo de referencia (Ryzen 7 4800H, GTX 1650 Ti; ver [entorno](../../06-Produccion/ENTORNO.md)), con opción de 30 FPS ([aceptación y pruebas](../../07-Calidad/ACEPTACION-Y-PRUEBAS.md)). Las cifras siguientes son **estimaciones de diseño**. Se medirán en la ventana autorizada con navegador, resolución, GPU, escena y duración registrados.

| Partida | Coste por paso o frame | Límite |
| --- | --- | --- |
| Crucero | O(1): contador y comparación con 4 hitos | — |
| Vuelo | O(1): unas decenas de operaciones y una prueba segmento-círculo | — |
| Superficie: jugador | 3 pases × 4 celdas como máximo | — |
| Superficie: LUMA | Igual que el jugador, más una ventana de atasco de 60 muestras | — |
| Interactuables | Distancia a cada uno, lineal | ≤ 16 por región |
| Campo de regreso | Dijkstra sobre 2464 celdas solo al cambiar la transitabilidad | Fuera del paso, al cambiar |
| Paso de simulación completo | Estimado < 0,05 ms | `simStepBudgetMs` = 1,0 ms |
| Orden de profundidad (render) | Ordenar las rebanadas y entidades visibles | ≤ 400 elementos visibles |
| Suelo visible | Unos 600 rombos con zoom 1, en capa de tilemap por lotes o prerenderizada por bloques | Un atlas por región |
| Entidades dinámicas | Jugador, LUMA, interactuables y efectos | `maxSurfaceDynamicEntities` = 32 |
| HUD en React | Lectura derivada a `hudUpdateHz` | 10 Hz |

Con 5 subpasos como máximo, el peor frame cuesta como mucho 5 pasos (< 0,25 ms estimado). El coste dominante es el render. El estado de simulación de la demo ocupa unos pocos KB (campo de distancias: 2464 números).

## 13. Casos de prueba (Vitest)

Pruebas de reglas, sin Phaser ni DOM. Usan estados construidos y registros de comandos por paso. Las tolerancias se indican donde intervienen flotantes.

| ID | Escenario | Resultado esperado |
| --- | --- | --- |
| T-FIS-01 | Determinismo: el mismo estado inicial, `rngSeed` y registro de comandos durante 10 000 pasos, ejecutado dos veces. | Hash de estado idéntico. Con otra semilla solo cambian los flujos cosméticos. |
| T-FIS-02 | Frames irregulares (5, 40, 16,7, 100, 1, 33 ms, …) sin alcanzar el límite. | Número de pasos = `floor(Σ / dt)`. Estado final idéntico al de pasos regulares. |
| T-FIS-03 | Un frame de 500 ms. | Exactamente 5 pasos, `acc < dt` después y sin ráfaga en el frame siguiente. |
| T-FIS-04 | 300 frames en pausa en CRUISE, APPROACH y SURFACE. | `stepIndex`, propelente, `cruiseStep` y reloj de misión sin cambios. Al reanudar, sin recuperación. |
| T-FIS-05 | Pestaña oculta. | Pausa y una sola solicitud de guardado. Al volver sigue la pausa. |
| T-FIS-06 | Frenado: 458 kg, `v` de 18 m/s opuesta al rumbo, W mantenida. | Para en el paso 329 ± 1, con 2,697 ± 0,01 kg y 49,2 ± 0,3 m. |
| T-FIS-07 | RCS: S mantenida 60 pasos a 458 kg. | Consumo 0,0656 ± 0,001 kg; `Δv = ve × ln(m0/m1)` ± 1e-6; empuje en `−h`. Q da empuje en `+l`. |
| T-FIS-08 | Mismo Isp: frenar 18 m/s con el RCS y con el motor. | Propelente igual ± 0,5 %. Relación de tiempos 7,5 ± 0,1. |
| T-FIS-09 | Precisión: Mayús con W y con A. | Empuje y giro × 0,25; propelente por unidad de Δv sin cambios. |
| T-FIS-10 | Giro: D durante 120 pasos desde 90°. | `θ = 270°` exacto (−180°). |
| T-FIS-11 | Asistente con θ = 270° y `v = (0, 18)`. | `|v| ≤ 0,05` en 583 ± 3 pasos; consumo 2,956 ± 0,02 kg (1,10 × manual). Se desactiva solo. |
| T-FIS-12 | Asistente con θ = 270° y `v = (2,5, 18)`. | Fuerza paralela a `−v` en cada paso (error < 1e-9 rad); consumo 3,365 ± 0,02 kg; factor `s` inicial 0,218. |
| T-FIS-13 | Asistente activo y se pulsa W. | Se desactiva en ese paso; A y D siguen girando durante la asistencia. |
| T-FIS-14 | Bordes del descenso: `|r| = 12,0`, `|v| = 1,0` y `|Δθ| = 15,0` mantenidos. | LANDING exactamente en el paso 120. Con 12,001, 1,001 o 15,001, nunca. Si falla en el paso 119, `holdSteps` vuelve a 0. |
| T-FIS-15 | Exceso de velocidad al entrar en la zona a 4,01 y a 4,00 m/s; trayectoria que solo roza el disco dentro de un paso. | 4,01 → `ApproachFailed { overspeed }`; 4,00 → sin fallo; el roce se detecta. |
| T-FIS-16 | Sin intervención desde la geometría inicial. | Distancia mínima 55,03 ± 0,05 m sin entrar en la zona; `ApproachFailed { lost }` en el paso 3944. |
| T-FIS-17 | Reserva con la configuración A: quemar desde `p* + 0,001 kg`. | Sin aborto mientras `p ≥ p*` (88,65 kg). `ApproachAborted { reserve }` una sola vez en el primer paso con `p < p*`, con la opción de reintentar o ir al informe. La forma cerrada coincide con `ascentReserveKg(t)` de PARAMETROS en cada paso. |
| T-FIS-18 | Reserva al inicio en las 93 combinaciones que superan V1–V5 (tanques S de 96, M de 120 y L de 150 kg; corrección a tiempo o tarde). | Sobrante > 0 en CP-APPROACH. Mínimo: SOLAR-PANEL + O2-RESERVE con TANK-S, 7,16 kg a tiempo y 4,16 ± 0,01 kg tarde. |
| T-FIS-19 | Reintento tras `ApproachFailed` y tras `ApproachAborted { reserve }`: gastar 5 kg y reintentar tres veces; elegir después «terminar». | Cada intento empieza con el propelente de CP-APPROACH. La instantánea no cambia. Solo aumenta el contador de intentos. «Terminar» lleva a REPORT. |
| T-FIS-20 | Ascenso, prueba de propiedad: `p ∈ [p*, tanque]` para cargas de 12–60 kg y carga de retorno real de 0 y 3 kg. | Propelente tras descenso y ascenso ≥ 0; mínimo 9,03 kg (con 3 kg de carga). Caso peor de A: 9,84 ± 0,05 kg. `AscentCompleted` una vez. |
| T-FIS-21 | Crucero sin doble consumo: «Corregir ahora» en el paso 600; guardar y recargar en el 700; pausar 1000 frames; repetir el comando. | −2 kg una sola vez; `MidcourseCorrected { kind: now }` una vez; nada se reaplica. |
| T-FIS-22 | Límites del crucero: «Corregir ahora» en los pasos 479, 480 y 1199; sin respuesta; con `pauseOnDecisions`. | 479 → rechazado (ventana cerrada); 480 y 1199 → 2 kg; sin respuesta → 5 kg en el paso 1200; con la opción, `cruiseStep` no pasa de 480 hasta decidir. En el paso 900 se muestran 38 h. |
| T-FIS-23 | Proyección ida y vuelta: 10 000 puntos con semilla, alturas 0–3. | Error < 1e-9 m. Las esquinas dan (0, 0), (1792, 896), (−1408, 704) y (384, 1600). |
| T-FIS-24 | Entrada WASD. | W → `(−1, −1)/√2` (recto arriba en pantalla); D → recto a la derecha; W+D → `(0, −1)`; velocidad 1,6 m/s en las 8 direcciones. |
| T-FIS-25 | Aceleración y frenado desde parado con W, y al soltar. | 1,6 m/s en el paso 39 ± 1; parada en 32 ± 1 pasos. |
| T-FIS-26 | Colisión en esquina: diagonal contra la esquina convexa de una celda; deslizamiento a lo largo de dos celdas contiguas. | Rodea la esquina sin penetrar (distancia ≥ radio − 1e-6) ni atascarse; en la unión se conserva la velocidad paralela. |
| T-FIS-27 | Sin túnel: velocidad máxima contra un muro de una celda, 600 pasos y 64 rumbos con semilla. | El centro nunca cruza el muro. |
| T-FIS-28 | Sombra sin lámpara; con lámpara; lámpara apagada dentro de la sombra. | Sin lámpara bloquea como un muro, con `blockedBy = shadow` y sin cambio de recursos. Con lámpara se transita. Apagada dentro, se puede salir y no reentrar. |
| T-FIS-29 | LUMA separada por un muro o a más de 8 m. | REGROUP tras `lumaStuckS` o de inmediato si supera la distancia; posición candidata libre y determinista; marca `teleported`. |
| T-FIS-30 | Interactuables a 1,2 m y 1,4 m; uno a 1,5 m exactos; empate; escaneo. | Se elige el más cercano; 1,5 m incluido; empate por ID; R usa 3,0 m. |
| T-FIS-31 | Profundidad: jugador delante y detrás de una rebanada; empate de clave. | Orden correcto y estable entre ejecuciones. |
| T-FIS-32 | Reloj de superficie: 60 pasos; caminar 1 m a velocidad constante. | 20 s de misión; 0,2083 min ± 0,001. |
| T-FIS-33 | Misión ligera BATTERY-PACK + LAMP con TANK-S al empezar la aproximación. | Masa 426 kg; aceleración del motor 3,52 m/s²; `p*` = 86,58 ± 0,01 kg; sobrante 7,42 kg (4,42 kg con corrección tardía). |
| T-FIS-34 | Ascenso manual: entregar el registro y la muestra en DEMO-POI-06; `RequestAscent` fuera y dentro del alcance de DEMO-POI-01. | La entrega no inicia ASCENT. `RequestAscent` lejos del módulo se rechaza; en DEMO-POI-01 inicia ASCENT (con confirmación si el objetivo principal está incompleto). |
| T-FIS-35 | Campo de distancias desde DEMO-POI-01, con y sin lámpara. | Distancia 0 en el módulo. Las distancias desde cada lugar coinciden con las rutas de PARAMETROS (±10 %). Sin lámpara, ninguna ruta atraviesa sombra. |

## 14. Parámetros propuestos

Ninguno pendiente. Los parámetros técnicos que introdujo esta esquemática están aceptados en [PARAMETROS-DEMO](PARAMETROS-DEMO.md) («Parámetros técnicos aceptados de otras esquemáticas»):

- **Vuelo:** `approachStartHeadingDeg`, `landingSiteHeadingDeg`, `landingDescentDeltaVMps`, `assistDoneSpeedMps`, `flightScalePxPerM` y `flightScaleSwitchM`.
- **LUMA:** `lumaFollowHysteresisM`, `lumaStuckS` y `lumaRegroupFadeS`.
- **Cámara y oclusión:** `cameraFollowRatePerS`, `cameraZoomLevels`, `occluderAlpha` y `occluderFadeS`.
- **Infraestructura y rendimiento:** `collisionEpsilonM`, `simStepBudgetMs`, `maxSurfaceDynamicEntities` y `hudUpdateHz`.

Los tiempos del crucero y de las secuencias (`midcourseDecisionOpenS`, `midcourseDecisionCloseS`, `landingSequenceRealS` y `ascentSequenceRealS`) también están en PARAMETROS. `cruiseLumaSceneFrac` no se aceptó: la escena de LUMA en el crucero es a petición.

## Huecos y decisiones pendientes

- **Resueltos el 19 de octubre:**
  - `RESUME → CRUISE` y el reintento desde CP-APPROACH tras un fallo o `ApproachAborted` están en el [GDD](GDD-DEMO.md).
  - LUMA se reubica con un fundido.
  - La ecuación del cohete se cita con NASA-ROCKET-EQ, e Isp 311 s queda como valor de juego (S).
  - El planificador usa la masa tras la corrección.
  - El punto de regreso es DEMO-POI-01 y el ascenso siempre es manual (`RequestAscent`).
- **Balance:** las siete combinaciones válidas con TANK-S dejan 4,2–10,3 kg de sobrante (≈ 30–76 m/s). Se busca que obliguen a pilotar con cuidado; confirmarlo en las pruebas con jugadores.
- **Alturas:** PARAMETROS define `heightStepPx`, pero el cambio de nivel por rampas, los bordes bloqueantes y la restricción de profundidad de §8.2 deben reflejarse en el [diseño de nivel](DISENO-DE-NIVEL.md).
