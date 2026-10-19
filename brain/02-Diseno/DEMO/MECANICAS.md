---
id: mecanicas-demo
type: diseno
status: especificado-sin-implementar
updated: 2026-10-19
tags: [orbita, demo, mecanicas]
---
# Mecánicas de la demo

Esquemática de reglas de `MIS-DEMO-01`: recursos, planificador, crucero, reserva de aproximación, superficie, incidencia `INC-RELAY-01`, soporte vital, reacciones de LUMA, informe causal y progreso idempotente. **No hay código**: el pseudocódigo describe reglas para la implementación autorizada.

- Índice, máquina de estados, transiciones e IDs: [GDD de la demo](GDD-DEMO.md).
- Todos los nombres y valores numéricos: [parámetros de la demo](PARAMETROS-DEMO.md). Si este documento y PARAMETROS discrepan, prevalece PARAMETROS y este documento se corrige. Lo que aún no está en PARAMETROS aparece en [Parámetros propuestos](#14-parámetros-propuestos).
- Vuelo, movimiento, colisiones, campo de distancias y navegación: [físicas](FISICAS.md). Mapa, celdas de sombra, tramos y disparadores: [diseño de nivel](DISENO-DE-NIVEL.md).
- Canon: [LUMA](../LUMA.md), [bucle y misiones](../BUCLE-Y-MISIONES.md), [capítulo 2](../CAPITULOS/C2-BAJO-OTRO-CIELO.md), [demo y entrega](../../07-Calidad/DEMO-Y-ENTREGA.md), [contratos](../../03-Arquitectura/CONTRATOS.md), [guardado](../../03-Arquitectura/GUARDADO.md), [datos y realismo](../../04-Ciencia/DATOS-Y-REALISMO.md).

Convenciones: "min" = minutos de misión; "s reales" = segundos de reloj del jugador. En superficie, 1 min = 60 / `surfaceTimeScale` = 3 s reales. `minPerMeter = surfaceTimeScale / (walkSpeedMps × 60)` = 0,2083 min/m. `ve = ispS × g0` = 3049,87 m/s. `p(m, dv)` = `propellantForDeltaV(m, dv) = m × (1 − exp(−dv / ve))`.

## 1. Resumen y relación con el reto

Pregunta del reto: **¿puedes diseñar una misión que funcione?** ([requisitos NASA](../../04-Ciencia/REQUISITOS-NASA.md)). La demo la convierte en tres comprobaciones encadenadas:

1. **Antes de salir**, el planificador calcula en vivo carga, coste, propelente, soporte vital y energía. Bloquea los planes inviables con una causa legible (V1–V6) y advierte de sus debilidades (W1–W5).
2. **Durante la misión**, cada recurso se consume por reglas visibles: la corrección de crucero, el pilotaje, la ruta elegida, las actividades y la solución de la incidencia.
3. **Al final**, el informe compara lo previsto con lo ocurrido y atribuye cada diferencia a una decisión o acción concreta. "Reintentar con otra configuración" permite comprobar la hipótesis del jugador.

Ninguna regla castiga con muerte. LUMA no tiene estadística ni salud afectada por el jugador. La IA no participa en ninguna regla de este documento.

## 2. Modelo de recursos

### 2.1 Recursos y consumidores

| Recurso | Unidad | Capacidad inicial | Quién consume y cuándo | Dónde se muestra |
| --- | --- | --- | --- | --- |
| Carga | kg | `payloadCapacityKg` | Suma de masas de módulos. Solo cuenta en el planificador. | Planificador: barra "carga / 60 kg". |
| Coste | cr | `budgetCredits` | Suma de módulos más tanque. Solo planificador. | Planificador: barra "coste / 100 cr". |
| Propelente | kg | Tanque elegido (`TANK-S/M/L`) | Corrección de crucero (`midcourseNowKg` o `midcourseLateKg`); aproximación y descenso según [físicas](FISICAS.md), con `assistPropellantPenalty` en tramos asistidos; ascenso. No se consume en superficie. | Planificador (necesario, exigido, margen). Crucero y aproximación: kg y "sobrante para corregir". Informe. |
| Soporte vital | min | `lifeSupportBaseMin` (120) + `o2ReserveMin` (60) con `MOD-O2-RESERVE` | Solo en superficie, desde `Landed` hasta el inicio del ascenso: 1 min por cada min del reloj de misión (caminar, actividades, espera). | HUD de superficie: barra y número. Planificador: estimación. Informe. |
| Energía de equipo | Wh | `toolBatteryBaseWh` + `batteryPackWh` con `MOD-BATTERY-PACK` | Lámpara (`lampW` mientras el jugador está en sombra), espectrómetro (`scannerW` durante `scanMin`), relé con batería (`relayEnergyWh` de una vez). | HUD de superficie: Wh disponibles y Wh reservados. Planificador: estimación. Informe. |
| Generación solar | W | `solarPanelW` en luz, 0 en sombra | No es un depósito. Solo alimenta al relé tras desplegar el panel (sección 7). | Panel del relé: barra de carga. |
| Reloj de misión | min | — | Avanza en superficie a `surfaceTimeScale`. Se congela en `PAUSE`, `DIALOGUE`, `JOURNAL`, `MAP` y en los paneles de decisión (sección 12). | HUD de superficie: "T+ 00:45". |

Masa en vuelo: `massKg = landerDryMassKg + payloadKg + propellantKg`. Carga de retorno, según PARAMETROS:

- La validación (V5) y la reserva en vuelo suman siempre `returnCargoKg` completo (caso conservador).
- El ascenso real suma la carga que se lleve de verdad: `returnCargoLogKg` si el registro está a bordo y `returnCargoSampleKg` si lo está la muestra (0–3 kg; reparto propuesto en la sección 14). Un objeto entregado en la terminal se considera a bordo, porque el contenedor vuelve con el módulo.

### 2.2 Reserva de energía para el relé

```
reservedRelayWh = relayEnergyWh   si hasBattery y no hasSolar y relé no alimentado
                = 0               en otro caso
availableWh     = energyWh − reservedRelayWh
```

La lámpara y el espectrómetro solo gastan `availableWh`. Una configuración con la batería como única fuente siempre conserva los 100 Wh del relé. Con batería y panel no se reserva nada: si la energía baja de 100 Wh, el relé se resuelve con el panel.

### 2.3 Redondeo y visualización

- Las comparaciones (V1–V6, W1–W5, reserva en vuelo, aviso de soporte vital) usan valores sin redondear. Se redondea una sola vez, al mostrar.
- Propelente, márgenes y estimaciones: 1 decimal, al más cercano. Carga y coste: enteros (exactos).
- HUD de soporte vital y energía: entero hacia abajo, para no mostrar nunca más de lo disponible. Entre 0 y 1 se muestra "< 1 min" o "< 1 Wh".
- Duraciones: min de misión y, entre paréntesis, s reales ("Escanear · 3 min (9 s)").
- Separador decimal según idioma: coma en español, punto en inglés. Los IDs no se traducen.
- Ningún recurso se comunica solo con color: barra, número y texto de estado ("normal", "aviso", "reserva").

## 3. Planificador

### 3.1 Catálogo

Valores de [PARAMETROS](PARAMETROS-DEMO.md#presupuesto-y-módulos), repetidos para leer las reglas.

| ID | Nombre ES | Nombre EN (propuesto) | kg | cr | Consecuencia que muestra la ficha |
| --- | --- | --- | --- | --- | --- |
| MOD-DATA-READER | Lector de registros | Log reader | 5 | 10 | Necesario para recuperar el registro de `DEMO-POI-04` (V3). |
| MOD-LUMA-SHIELD | Protección ambiental de LUMA | LUMA environmental shield | 7 | 0 | Preinstalado, bloqueado. LUMA puede salir a superficie (V6). |
| MOD-BATTERY-PACK | Batería portátil | Portable battery | 14 | 12 | +400 Wh. Relé en 1 min (−100 Wh). |
| MOD-SOLAR-PANEL | Panel solar desplegable | Deployable solar panel | 9 | 18 | 600 W con luz. Relé en 2 min + 10 min de carga, solapable con la alineación. |
| MOD-SCANNER | Espectrómetro portátil | Portable spectrometer | 8 | 15 | Permite `OBS-GEO-01`. Sin él, ciencia máxima 2/3 (W2). |
| MOD-TOOLKIT | Kit de reparación | Repair kit | 10 | 10 | Alineación en 5 min en lugar de 15: ahorra 10 min con batería y 5 con panel (W5). |
| MOD-LAMP | Lámpara térmica de sombra | Shadow thermal lamp | 6 | 8 | Abre la ruta de sombra: −80 m (−16,7 min) frente al rodeo; 80 W en sombra (W3). |
| MOD-O2-RESERVE | Reserva de soporte vital | Life-support reserve | 12 | 14 | +60 min de soporte vital. |

| Tanque | Propelente | Coste |
| --- | --- | --- |
| TANK-S | 96 kg | 8 cr |
| TANK-M | 120 kg | 12 cr |
| TANK-L | 150 kg | 16 cr |

### 3.2 Selección

- Estado inicial de un intento nuevo: `MOD-LUMA-SHIELD` (bloqueado) y `TANK-M`. Aparecen V3, V4, W2, W3 y W5, lo que enseña desde el primer momento qué falta.
- Al reintentar desde el informe se precarga la configuración del intento anterior.
- Comandos de la UI y eventos de la simulación ([estructura de código §3](../../03-Arquitectura/ESTRUCTURA-CODIGO.md)): `SelectModule` → `ModuleSelected`; `DeselectModule` → `ModuleDeselected`; `SelectTank` → `TankSelected`. Siempre hay exactamente un tanque. Un comando rechazado no emite evento ni consume nada.
- `DeselectModule(MOD-LUMA-SHIELD)` se rechaza: no cambia el estado, no emite `ModuleDeselected` y muestra "Preinstalado: LUMA necesita esta protección para salir." / "Pre-installed: LUMA needs this protection to go outside."
- Cada cambio recalcula todo con la función pura `computePlan(config)`. No hay botón "calcular".
- Teclado y ratón: lista navegable con flechas, Espacio o Intro para alternar, fichas accesibles también con foco.

### 3.3 Cálculo en vivo

```
computePlan(config):
  payloadKg  = Σ masa(módulos)
  costCr     = Σ coste(módulos) + coste(tanque)
  tankKg     = propelente(tanque)

  m0         = landerDryMassKg + payloadKg + tankKg
  mApproach  = m0 − midcourseNowKg
  approachKg = p(mApproach, approachNominalDeltaVMps)
  mAscent    = mApproach − approachKg + returnCargoKg
  ascentKg   = p(mAscent, ascentDeltaVMps)
  neededKg   = midcourseNowKg + approachKg + ascentKg
  requiredKg = neededKg × (1 + propellantMarginHard)
  margin     = tankKg / neededKg − 1
  pStarKg    = umbral de reserva en vuelo (sección 5)

  estimate   = estimateSurface(config)     // 3.5; nula si falta fuente para el relé (V4)
  return { payloadKg, costCr, tankKg, neededKg, requiredKg, margin, pStarKg, estimate,
           issues: validate(config, ...) }
```

El planificador muestra también el "sobrante para corregir" previsto en `CP-APPROACH`: `tankKg − midcourseNowKg − pStarKg`.

### 3.4 Validación

```
validate(config, plan):
  issues = []
  if plan.payloadKg > payloadCapacityKg:                   add V1
  if plan.costCr > budgetCredits:                          add V2
  if MOD-DATA-READER ∉ config:                             add V3
  if MOD-BATTERY-PACK ∉ config y MOD-SOLAR-PANEL ∉ config: add V4
  if plan.tankKg < plan.requiredKg:                        add V5
  if MOD-LUMA-SHIELD ∉ config:                             add V6
  if plan.margin < propellantMarginWarn:                   add W1
  if MOD-SCANNER ∉ config:                                 add W2
  if MOD-LAMP ∉ config:                                    add W3
  if plan.estimate ≠ nula y
     plan.estimate.lifeSupportMin / plan.estimate.lifeSupportCapMin
       > lifeSupportPlanWarnFraction:                      add W4   // con V4 no se evalúa
  if MOD-TOOLKIT ∉ config:                                 add W5
  ordenar: V1..V6 y después W1..W5
  return issues

on LaunchMission:                         // comando del botón "Lanzar"
  issues = validate(config, computePlan(config))
  emit PlanValidated { errors, warnings, estimate }
  if no hay ningún V#: emit LaunchApproved { attemptId, config, estimate } → CRUISE
  else: el lanzamiento se rechaza y el foco va a la lista de errores
```

Todas las reglas se evalúan siempre: el jugador ve todos los problemas a la vez. El botón "Lanzar" indica el número de errores pendientes.

Mensajes (plantillas con valores interpolados de configuraciones reales de PARAMETROS):

| Código | Ejemplo ES | Ejemplo EN |
| --- | --- | --- |
| V1 | Carga 71 kg de 60 kg: sobran 11 kg. Quita módulos opcionales. *(Todo)* | Payload 71 kg of 60 kg: 11 kg over. Remove optional modules. |
| V2 | Coste 103 cr de 100 cr: te pasas en 3 cr. Elige un tanque o módulos más baratos. *(Todo con TANK-L)* | Cost 103 cr of 100 cr: 3 cr over budget. Pick a cheaper tank or modules. |
| V3 | Sin lector de registros no podrás recuperar el registro de la caché. Añade el lector (5 kg, 10 cr). | Without the log reader you cannot retrieve the cache log. Add the reader (5 kg, 10 cr). |
| V4 | El relé necesita 100 Wh y el traje solo aporta 60 Wh. Añade la batería portátil (+400 Wh) o el panel solar (600 W con luz). | The relay needs 100 Wh and the suit only provides 60 Wh. Add the portable battery (+400 Wh) or the solar panel (600 W in light). |
| V5 | Propelente 96 kg; se exigen 97,4 kg (84,7 necesarios + 15 % de margen). Elige un tanque mayor o reduce carga. *(A con TANK-S)* | Propellant 96 kg; 97.4 kg required (84.7 needed + 15% margin). Pick a larger tank or reduce payload. |
| V6 | Falta la protección ambiental de LUMA: no podría salir a superficie. Es preinstalada; restablece la configuración. *(solo partida importada o dañada)* | LUMA's environmental shield is missing: LUMA could not go outside. It is pre-installed; reset the configuration. |
| W1 | Margen de propelente 15,4 % (aviso por debajo de 30 %). Te quedarán 7,4 kg para corregir; corregir tarde resta 3 kg y el asistente gasta un 10 % más. *(BATTERY-PACK + LAMP con TANK-S)* | Propellant margin 15.4% (warning below 30%). You will have 7.4 kg spare for corrections; a late correction takes 3 kg and the assist uses 10% more. |
| W2 | Sin espectrómetro no podrás escanear el punto geológico: la ciencia quedará en 2 de 3 como máximo. | Without the spectrometer you cannot scan the geology site: science will reach 2 of 3 at most. |
| W3 | Sin lámpara, la ruta de sombra está cerrada: el rodeo iluminado añade 80 m (16,7 min de soporte vital). | Without the lamp the shadow route is closed: the lit detour adds 80 m (16.7 min of life support). |
| W4 | Soporte vital previsto 94,0 de 120 min (78,4 %, aviso por encima de 70 %). Cualquier desvío te acerca al límite: considera la lámpara o la reserva (+60 min). *(B sin O2-RESERVE)* | Planned life support 94.0 of 120 min (78.4%, warning above 70%). Any detour brings you close to the limit: consider the lamp or the reserve (+60 min). |
| W5 (con batería) | Sin kit, la alineación del relé es manual: 15 min en lugar de 5 (la reparación pasa de 6 a 16 min). | Without the kit, relay alignment is manual: 15 min instead of 5 (repair goes from 6 to 16 min). |
| W5 (solo panel) | Sin kit, la alineación es manual: 15 min en lugar de 5. Con el panel se solapa con la carga, y la reparación pasa de 12 a 17 min. | Without the kit, alignment is manual: 15 min instead of 5. With the panel it overlaps the charge, and repair goes from 12 to 17 min. |

### 3.5 Estimación previa de superficie

Usa la mejor ruta disponible con la configuración y la solución más corta de la incidencia, con las longitudes objetivo de PARAMETROS.

```
estimateSurface(config):
  if no hasBattery y no hasSolar: return nula                    // V4
  if hasLamp:  route = 01→02→04 (sombra)→05→04→02 (sombra)→03→06 = 237 m, 80 m en sombra
  else:        route = 01→02→03→04 (rodeo)→05→04→03 (rodeo)→06   = 317 m, 0 m en sombra
  walkMin  = route.m × minPerMeter
  align    = hasToolkit ? relayAlignToolkitMin : relayAlignManualMin
  relayMin = hasBattery ? relayBatteryMin + align
                        : relaySolarDeployMin + max(relaySolarChargeMin, align)
  actsMin  = observeMin + sampleMin + cacheRetrieveMin + deliverMin + relayMin
           + (hasScanner ? scanMin : 0)
  lifeSupportMin    = walkMin + actsMin
  lifeSupportCapMin = lifeSupportBaseMin + (hasO2 ? o2ReserveMin : 0)
  energyWh    = route.shadowM × minPerMeter / 60 × lampW
              + (hasScanner ? scanMin / 60 × scannerW : 0)
              + (hasBattery ? relayEnergyWh : 0)
  energyCapWh = toolBatteryBaseWh + (hasBattery ? batteryPackWh : 0)
```

Con batería y panel, la estimación usa la batería (6 o 16 min frente a 12 o 17). La ruta 06→01 (unos 7 m hasta el módulo para `RequestAscent`) no se incluye, igual que en PARAMETROS. La estimación se congela en `LaunchApproved` y el informe la compara con lo real.

### 3.6 Configuraciones de referencia

Propelente (kg; necesario sin margen; exigido = necesario × 1,15):

| Config. | Módulos | Carga | Coste (S / M / L) | TANK-S (96) | TANK-M (120) | TANK-L (150) | Avisos con M/L |
| --- | --- | --- | --- | --- | --- | --- | --- |
| A · Sombra rápida | DATA-READER, LUMA-SHIELD, BATTERY-PACK, LAMP, SCANNER | 40 kg | 53 / 57 / 61 cr | Necesario 84,7; exigido 97,4 → **V5**; margen 13,4 % (W1) | Necesario 89,2; exigido 102,6 → válida; margen 34,5 % | Necesario 94,9; exigido 109,1 → válida; margen 58,0 % | W5 |
| B · Luz y paciencia | DATA-READER, LUMA-SHIELD, SOLAR-PANEL, TOOLKIT, SCANNER, O2-RESERVE | 51 kg | 75 / 79 / 83 cr | Necesario 86,8; exigido 99,8 → **V5**; margen 10,6 % (W1) | Necesario 91,3; exigido 105,0 → válida; margen 31,4 % | Necesario 97,0; exigido 111,5 → válida; margen 54,7 % | W3 |
| Todo | Los ocho | 71 kg | 95 / 99 / 103 cr | **V1, V5** (exigido 104,1); W1 (6,0 %) | **V1**; W1 (26,2 %) | **V1, V2**; margen 48,8 % | — |

TANK-S solo es válido con 33 kg de carga o menos. Ejemplo ligero: DATA-READER, LUMA-SHIELD, BATTERY-PACK y LAMP (32 kg, 38 cr): exigido 95,6; margen 15,4 % (W1); avisos W1, W2 y W5; sobrante para corregir 7,42 kg.

Superficie (estimación de 3.5):

| Config. | Ruta | Recorrido | Caminar | Actividades | Total | Capacidad | Uso | Holgura | Energía |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| A | Sombra | 237 m | 49,4 min | 32 min (relé 1 + 15) | 81,4 min | 120 min | 67,8 % | 38,6 min | 123,7 de 460 Wh |
| B | Rodeo iluminado | 317 m | 66,0 min | 28 min (relé 2 + max(10, 5)) | 94,0 min | 180 min | 52,2 % | 86,0 min | 1,5 de 60 Wh |
| B sin O2-RESERVE | Rodeo iluminado | 317 m | 66,0 min | 28 min | 94,0 min | 120 min | 78,4 % → **W4** | 26,0 min | 1,5 de 60 Wh |

Energía de A: lámpara 80 m × 0,2083 = 16,7 min × 80 W = 22,2 Wh; espectrómetro 3 min × 30 W = 1,5 Wh; relé 100 Wh.

W4 en las combinaciones válidas: todas las que no llevan ni LAMP ni O2-RESERVE lo activan (70,9–82,5 %); ninguna con LAMP o con O2-RESERVE lo activa (máximo 68,6 %).

### 3.7 Por qué ninguna es dominante

| Criterio | A | B |
| --- | --- | --- |
| Carga y coste | Mejor: 40 kg, 57 cr | 51 kg, 79 cr |
| Margen y sobrante (M) | Mejor: 34,5 %; 29,35 kg | 31,4 %, cerca de W1; 26,51 kg |
| Tiempo en superficie | Mejor: 81,4 min | 94,0 min |
| Holgura de soporte vital | 38,6 min: un desvío largo la agota | Mejor: 86,0 min; tolera desvíos y errores |
| Energía | Depende de 123,7 Wh y de no derrochar lámpara | Casi no usa energía |
| Riesgo | La sombra exige gestionar energía; alineación manual 15 min | Recorrido largo; el relé obliga a esperar la carga |

A es eficiente pero tiene poco margen ante los desvíos; B es robusta y cara. Las dos completan la misión y la ciencia 3/3. Dentro de B, el kit ahorra 5 min en el relé a cambio de 10 kg y 10 cr; sin él (41 kg, 69 cr) la reparación dura 17 min y el total 99,0 min (55,0 %). Es una decisión de verdad, no una opción dominada. "Todo" es inviable (V1), así que elegir es obligatorio.

## 4. Crucero

- Duración: `cruiseRealSeconds` (30 s reales) representan `cruisePhysicalHours` (76 h). El HUD muestra horas físicas: `h = t / 30 × 76`.
- La decisión de corrección se abre en `midcourseDecisionOpenS` (8 s, ≈ 20 h) y se cierra en `midcourseDecisionCloseS` (20 s, ≈ 51 h). Texto: "Corregir ahora (2 kg)" / "Más tarde (5 kg)".
- Comando `ChooseMidcourse { kind }`. "Corregir ahora" (`now`, solo dentro de la ventana): resta `midcourseNowKg` y emite `MidcourseCorrected { kind: now, kg: 2 }`.
- "Más tarde" (`late`) o ignorar la ventana: al cerrarse se aplica automáticamente `midcourseLateKg` y se emite `MidcourseCorrected { kind: late, kg: 5 }`. No hay fallo ni pérdida de trayectoria.
- La corrección se aplica una sola vez por intento (`midcourseResolved`). `RESUME → CRUISE` restaura el tramo y la decisión ya aplicada sin volver a consumir ([físicas](FISICAS.md)).
- La validación (V5) asume la corrección a tiempo. Elegir tarde reduce en 3 kg el sobrante para corregir (A con M: 118,0 → 115,0 kg en `CP-APPROACH`; sobrante 29,35 → 26,35 kg).
- Escena `LUMA-R-CRUISE-REST`, a petición: botón "Descansar con LUMA" (comando `RequestLumaRest`) disponible durante todo el crucero. Abre `DIALOGUE`, que pausa el crucero y no consume recursos. Una vez por intento.
- Esc pausa el crucero y la ventana de decisión. La opción de accesibilidad `pauseOnDecisions` (sección 12) congela el crucero mientras la decisión está abierta; en ese caso solo se aplica "tarde" si el jugador lo elige.
- Al terminar el crucero se guarda `CP-APPROACH` con el propelente de ese momento.

## 5. Aproximación desde el punto de vista de recursos

La dinámica, los controles, el asistente, las condiciones de descenso y el orden de comprobaciones están en [físicas](FISICAS.md). Aquí solo se fija cómo afectan al propelente.

- Todo consumo de motor, RCS, asistente (× `assistPropellantPenalty`) y de la secuencia `LANDING` se resta del mismo depósito.
- **Reserva en vuelo** (regla de PARAMETROS): `ascentReserveKg(t) = p(m(t) + returnCargoKg, ascentDeltaVMps) × (1 + propellantMarginHard)`. Como `m(t)` incluye el propelente, equivale al umbral fijo:

```
k  = (1 − exp(−ascentDeltaVMps / ve)) × (1 + propellantMarginHard)     ≈ 0,20538
p* = k × (landerDryMassKg + payloadKg + returnCargoKg) / (1 − k)
if propellantKg < p*: emit ApproachAborted { reason: reserve, propellantKg }
spareKg = propellantKg − p*                                            // "sobrante para corregir"
```

| Configuración | p* | Sobrante en CP-APPROACH (a tiempo / tarde) |
| --- | --- | --- |
| A con TANK-M | 88,65 kg | 29,35 / 26,35 kg |
| B con TANK-M | 91,49 kg | 26,51 / 23,51 kg |
| BATTERY-PACK + LAMP con TANK-S | 86,58 kg | 7,42 / 4,42 kg |

- El HUD muestra `spareKg` en kg ("Sobrante para corregir: 29,3 kg" en A con TANK-M), no un umbral oculto.
- **Fallo de pilotaje o aborto** (`ApproachFailed { reason: overspeed | lost }`; `ApproachAborted { reason: reserve }` o, por el comando `AbortApproach`, `{ reason: player }`): el modo sigue en APPROACH con el fallo pendiente y **no** se emite `ReportReady`. Se ofrecen dos comandos: `RetryApproach` ("Reintentar desde CP-APPROACH") y `GoToReport` ("Terminar e ir al informe", resultado Aborto). `RetryApproach` restaura **toda** la instantánea de `CP-APPROACH`, propelente incluido; no se acumulan consumos de intentos fallidos. Solo `approachRetries` persiste para el informe.
- Tras el ascenso (A con M, aproximación nominal, registro y muestra a bordo) quedan 30,8 kg; el informe muestra ese sobrante.
- **Tensión con TANK-S**: con corrección tardía, las misiones ligeras con TANK-S dejan 4,2–7,3 kg de sobrante, unos 30–54 m/s. Según FISICAS, la parte pilotada de la aproximación (25 m/s) consume ≈ 3,4 kg antes del descenso automático, de modo que la holgura real queda en 0,7–3,9 kg. Un pilotaje limpio aterriza; uno torpe, o con mucho asistente, aborta por reserva. Es la consecuencia buscada en PARAMETROS, y el aviso W1 la anticipa.

## 6. Superficie

### 6.1 Interacción

- **E** (comando `Interact { targetId, actionId }`): interactúa con el punto activo más cercano a menos de `interactRadiusM` (1,5 m). La indicación muestra acción, requisito y duración: "E · Recuperar registro · 6 min (18 s)".
- **R** (comando `Scan { targetId }`): escanea el objetivo geológico a menos de `scanRadiusM` (3,0 m). Requiere `MOD-SCANNER`. Sin él: "No llevas espectrómetro." / "You did not bring the spectrometer." (sin consumo de tiempo). Fuera de alcance: "Acércate al punto geológico (3 m)."
- Las actividades se inician con **una pulsación**, sin mantener teclas. Durante la actividad el personaje no se mueve; el reloj avanza a `surfaceTimeScale` y la barra muestra min y s reales restantes.
- "Cancelar" (E o botón; comando `CancelActivity`) detiene la actividad: no se conserva progreso y el tiempo transcurrido sí se consume. Esc pausa sin cancelar.
- Los requisitos se comprueban **antes** de empezar: una acción imposible no consume tiempo.

### 6.2 Inventario mínimo

Conjunto sin masa gestionable: `ITEM-AUR-LOG-PARTIAL` e `ITEM-SAMPLE-REGOLITH-01`. Las observaciones (`OBS-TERRAIN-01`, `OBS-GEO-01`) van al diario, no al inventario. Cada ID está como máximo una vez.

### 6.3 Regla de sombra

- Sin `MOD-LAMP`, las celdas de sombra ([diseño de nivel](DISENO-DE-NIVEL.md)) no son transitables. La simulación detiene al jugador en el borde (`blockedBy = shadow`, [físicas](FISICAS.md)) y muestra: "Sombra profunda: sin lámpara no puedes cruzar. Ruta iluminada por el rodeo." / "Deep shadow: you cannot cross without a lamp. Lit route via the detour." Sin daño ni consumo.
- Con lámpara, se enciende sola al entrar y consume `lampW` por cada min en sombra.
- Para entrar en un tramo de sombra se exige `availableWh` ≥ energía para cruzarlo (tramo de 40 m: 8,3 min × 80 W = 11,1 Wh). Si no alcanza, se bloquea igual (`blockedBy = shadowEnergy`) con "Energía insuficiente para cruzar la sombra (11,1 Wh)." / "Not enough energy to cross the shadow (11.1 Wh)."
- **Salida guiada**: si estando en sombra `availableWh` llega a la energía necesaria para salir a la celda iluminada más cercana, la simulación pausa, lo explica y lleva al jugador automáticamente a esa celda. El tramo consume tiempo y energía normalmente. Nadie queda atrapado.

### 6.4 Lugares

Duraciones en min de misión (s reales).

| Lugar | Pregunta | Acción | Requisito | Duración | Evento | Resultado | LUMA |
| --- | --- | --- | --- | --- | --- | --- | --- |
| DEMO-POI-01 Módulo y refugio | ¿Con qué cuento y cómo vuelvo? | E "Revisar estado": recursos reales frente a la estimación, ruta de regreso y protección de LUMA. E "Subir al módulo": comando `RequestAscent { confirmed }`. | — | Revisar: panel con pausa, 0 min. Subir: secuencia ASCENT (`ascentSequenceRealS`). | `Landed` al llegar (guarda `CP-SURFACE`); `AscentCompleted` al final del ascenso. | Orientación; único punto de salida de la superficie. | `LUMA-R-LANDING` tras `Landed`. |
| DEMO-POI-02 Mirador | ¿Qué acceso a la caché es viable con mi equipo? | E observar. | — | `observeMin` 2 (6 s) | `ObservationRecorded { OBS-TERRAIN-01 }` | El mapa añade a los dos accesos su longitud, metros en sombra y minutos estimados con tu equipo. | `LUMA-R-VIEWPOINT` |
| DEMO-POI-03 Punto geológico | ¿Qué materiales forman el terreno de esta zona? | R escanear; E tomar muestra. | Escanear: `MOD-SCANNER`. Muestra: ninguno. | `scanMin` 3 (9 s) con `scannerW`; `sampleMin` 3 (9 s) | `ObservationRecorded { OBS-GEO-01 }`; `SampleCollected { ITEM-SAMPLE-REGOLITH-01 }` | Lectura espectral registrada "para análisis en laboratorio", sin conclusión prefijada; muestra en el inventario. | `LUMA-R-SAMPLE` |
| DEMO-POI-04 Caché protegida | ¿Qué dejó aquí la expedición de Aurora? | E recuperar registro. | `MOD-DATA-READER` (garantizado por V3) | `cacheRetrieveMin` 6 (18 s) | `ItemRecovered { ITEM-AUR-LOG-PARTIAL }` | Registro en el inventario. Estado: recogido, **no validado**. Texto parcial sin destino confirmado. | `LUMA-R-CACHE`; `LUMA-R-SHADOW` en el borde de sombra (disparador del diseño de nivel) |
| DEMO-POI-05 Relé | ¿Por qué no hay enlace y cómo se recupera? | E abrir el panel de reparación (sección 7). | Batería o panel (V4) | Ver sección 7 | `RelayPowered`, `RelayAligned`, `LinkRestored` | Enlace restablecido; habilita transmitir el registro. | `LUMA-R-RELAY` |
| DEMO-POI-06 Terminal de retorno | ¿Qué llevo de vuelta y qué queda confirmado? | E entregar todo lo entregable del inventario en una sola acción. | Registro: `LinkRestored`. Muestra: ninguno. | `deliverMin` 2 (6 s) por acción | `ItemDelivered { itemId }` por objeto | Registro transmitido: objetivo principal completo. El juego sigue en SURFACE. | `LUMA-R-DELIVER` |

Reglas de la terminal y del ascenso:

- Si el inventario solo contiene el registro y no hay enlace, la acción no empieza: "Sin enlace no se puede transmitir el registro. Restablece el relé." / "No link: the log cannot be transmitted. Restore the relay."
- Si hay muestra y registro sin enlace, se entrega la muestra y el registro se queda en el inventario con el mismo mensaje.
- Texto de entrega del registro: "Registro parcial transmitido. Es una pieza: su autenticación necesita más pruebas." / "Partial log transmitted. It is one piece: authenticating it needs more evidence." No valida AUR-E03, no revela Marte como destino ni el final.
- Tras entregar, el HUD indica "Objetivo principal completo. Sube al módulo (POI-01) cuando quieras." / "Main objective complete. Board the lander (POI-01) whenever you want." **Entregar nunca inicia el ascenso.** Se puede seguir explorando y entregar la muestra después; el soporte vital sigue corriendo.
- `RequestAscent` (a `interactRadiusM` o menos de DEMO-POI-01) con el objetivo principal completo: ascenso directo. Con el objetivo principal incompleto exige `confirmed = true`; la UI pide antes la confirmación "Si subes ahora la misión se abortará. Pendiente: {lista}." / "If you board now the mission will be aborted. Pending: {list}." Cancelar (`ClosePanel`) devuelve a SURFACE sin consumo.
- El objetivo principal se cumple con `ItemDelivered { ITEM-AUR-LOG-PARTIAL }`, que exige antes `ItemRecovered` y `LinkRestored`.
- Orden libre: POI-02 a POI-06 se pueden visitar en cualquier orden y repetir.

## 7. Incidencia `INC-RELAY-01`

El relé tiene dos subproblemas independientes, **energía** y **alineación**. `LinkRestored` se emite en cuanto ambos están resueltos, aunque el jugador ya no esté en el relé.

### 7.1 Energía

| Solución | Requisito | Acción | Consumo | Evento |
| --- | --- | --- | --- | --- |
| Batería | `MOD-BATTERY-PACK` y `energyWh` ≥ `relayEnergyWh` | E "Conectar batería" | `relayBatteryMin` 1 min (3 s); −100 Wh | `RelayPowered { source: battery }` al terminar |
| Panel | `MOD-SOLAR-PANEL`; el lugar debe estar iluminado (POI-05 lo está) | E "Desplegar panel" | `relaySolarDeployMin` 2 min (6 s); después carga en segundo plano a `solarPanelW`: `relaySolarChargeMin` = 100 Wh / 600 W = 10 min (30 s) | `RelayPowered { source: solar }` cuando el relé acumula 100 Wh |

- La carga solar avanza con el reloj de misión aunque el jugador se aleje. En sombra la generación es 0 (regla general; en la demo el panel solo se despliega en el relé).
- Con batería y panel, el panel de reparación ofrece las dos con su duración: "Batería · 1 min · −100 Wh" / "Panel · 2 min + 10 min de carga (alinea mientras carga)".
- Si `energyWh` < 100 y hay panel, la opción de batería aparece desactivada: "Batería con {X} Wh: el relé necesita 100 Wh."
- **Mensaje defensivo**, no alcanzable gracias a V4 (solo partida dañada): "El relé necesita 100 Wh. No llevas batería ni panel y el traje solo tiene 60 Wh. El enlace no se puede restablecer en este intento; vuelve al módulo y el informe lo explicará." / "The relay needs 100 Wh. You have no battery or panel and the suit only holds 60 Wh. The link cannot be restored in this attempt; return to the lander and the report will explain."

### 7.2 Alineación

La alineación no necesita que el relé tenga energía: la señal se lee con el receptor del traje (simplificación S).

| Solución | Requisito | Interacción | Duración | Evento |
| --- | --- | --- | --- | --- |
| Con kit | `MOD-TOOLKIT` | E "Alinear con la guía del kit" (una acción) | `relayAlignToolkitMin` 5 min (15 s) | `RelayAligned { method: toolkit }` |
| Manual | — | Panel de orientación (abajo) y confirmación | `relayAlignManualMin` 15 min (45 s) | `RelayAligned { method: manual }` |

Alineación manual, comprensible y sin habilidad fina:

1. E abre el panel "Orientación de la antena". El reloj de misión **se congela** mientras el panel está abierto.
2. Un dial muestra `relayAlignPositions` posiciones (8, cada 45°). ←/→, A/D o dos botones giran **un paso** por pulsación (comando `RotateRelayDial { direction }`).
3. Un medidor de señal indica de 0 a 4 barras, con texto ("Señal 2 de 4") y un tono que sube con la señal. La posición de señal máxima es fija y la define el diseño de nivel.
4. "Confirmar" (`ConfirmRelayAlignment`) solo se activa con señal 4 de 4. Al confirmar empieza la actividad de 15 min (45 s reales).
5. Alternativa de un botón: "Barrido automático" (`AutoSweepRelay`) gira paso a paso hasta la señal máxima y deja listo "Confirmar". Cuesta lo mismo.

La duración es fija: no depende del número de pasos ni de la rapidez del jugador.

### 7.3 Solapamiento y tiempos totales

| Energía | Alineación | Tiempo en el relé | Cálculo |
| --- | --- | --- | --- |
| Batería | Kit | 6 min | 1 + 5 |
| Batería | Manual | 16 min | 1 + 15 |
| Panel | Kit | 12 min | 2 + max(10, 5) |
| Panel | Manual | 17 min | 2 + max(10, 15) |

El kit ahorra 10 min con batería y 5 con panel. El solapamiento exige desplegar el panel **antes** de alinear; el panel de reparación ofrece primero "Despliega el panel y alinea mientras carga". Si el jugador alinea primero, el total es alineación + 2 + 10 (17 con kit, 27 a mano) y el informe lo explica (C-05).

## 8. Soporte vital

- Capacidad: `lifeSupportBaseMin` (120) + `o2ReserveMin` (60) con `MOD-O2-RESERVE`.
- Consumo continuo de 1 min por min de misión, desde `Landed` hasta el inicio del ascenso. Caminar, actividades y espera consumen. `PAUSE`, `DIALOGUE`, `JOURNAL`, `MAP` y los paneles de decisión (relé, revisar estado, confirmar ascenso) no consumen.
- **Punto de regreso: DEMO-POI-01** (módulo), según PARAMETROS y el GDD.
- **Aviso**:

```
returnMin    = shortestAvailablePathM(posición, DEMO-POI-01) × minPerMeter
thresholdMin = returnMin × lifeSupportWarnFactor + lifeSupportWarnBufferMin
if lifeSupportMin < thresholdMin y no lifeSupportWarned:
    emit LifeSupportWarning { remainingMin, thresholdMin }; lifeSupportWarned = true
```

  "Ruta disponible" incluye la sombra solo si hay lámpara y `availableWh` para cruzarla. El camino lo calcula el campo de distancias de [físicas](FISICAS.md). Ejemplos desde el relé (POI-05):
  - Con lámpara, por los tramos de PARAMETROS 05→04→02→01: 101 m → 21,0 min → umbral 31,3 min.
  - Sin lámpara, el camino iluminado medido por el [diseño de nivel](DISENO-DE-NIVEL.md) es de 129,1 m → 26,9 min → umbral 38,6 min.
- El evento se emite una vez por intento. El HUD mantiene el estado "aviso" (barra, texto "Vuelve al módulo: {X} min de camino" e icono) mientras la condición se cumpla.
- **Regreso de emergencia**: al llegar a 0 se interrumpe la actividad en curso sin completarla y se emite `EmergencyReturn`. El traje pasa a su reserva de emergencia (ficción F): el astronauta y LUMA vuelven automáticamente a DEMO-POI-01, sin consumo ni control del jugador. Después: ASCENT → informe con resultado "Aborto".
- En el aborto, lo no entregado no cuenta: el registro o la muestra que sigan en el inventario no suman. Las observaciones registradas se conservan.
- Recorridos de referencia, sin desvíos:
  - A termina con 38,6 min sobrantes: caché a los 23,8 min, relé resuelto a los 45,0 min.
  - B termina con 86,0 min sobrantes: relé resuelto a los 60,8 min.
  - B sin O2-RESERVE termina con 26,0 min.
  - En ningún caso se dispara el aviso.

## 9. LUMA en la demo

LUMA no tiene salud, energía ni estadística. Su paleta (ámbar y naranja del capítulo 2) y su conducta vienen de hitos escritos, no del jugador. En la demo solo reacciona con texto escrito. Sigue al jugador según [físicas](FISICAS.md) (`lumaFollowDistM`, `lumaSpeedMps`, `lumaRegroupDistM`) y no bloquea el paso.

Reglas comunes:

- Las reacciones observan eventos; nunca los generan ni cambian recursos o progreso. Cada reacción mostrada emite `LumaReaction { reactionId }`, solo para registro y guardado.
- Formato: gesto + traducción breve en una burbuja no modal que no bloquea controles. Duración `lumaBubbleS` (5 s). Separación global mínima entre burbujas `lumaReactionMinGapS` (8 s). La misma ID no se repite antes de `lumaReactionCooldownS` (20 s).
- Cola de 2 como máximo. Si se llena, se descarta la más antigua, salvo `LUMA-R-LOWLS`, que tiene prioridad.
- No se muestran durante la aproximación ni interrumpen una actividad: se encolan hasta que termina.
- Límite por defecto: una vez por intento (`attemptId`), además del enfriamiento por ID.

| ID | Disparador exacto | ES | EN | Límite |
| --- | --- | --- | --- | --- |
| LUMA-R-CRUISE-REST | El jugador elige "Descansar con LUMA" en el crucero | LUMA se acurruca junto a la ventanilla. «Ruido bajo. Tranquilo. Bien.» | LUMA curls up by the window. «Low hum. Calm. Good.» | Una por intento; a petición |
| LUMA-R-LANDING | `Landed` | LUMA toca el suelo con una pata, despacio. «Suelo gris. Ligero.» | LUMA taps the ground slowly. «Grey ground. Light.» | Una por intento |
| LUMA-R-VIEWPOINT | `ObservationRecorded { OBS-TERRAIN-01 }` | Mira a un lado y a otro. «Allí, corto y oscuro. Allí, largo y con luz.» | It looks one way, then the other. «There: short and dark. There: long and bright.» | Una por intento |
| LUMA-R-SHADOW | Primer contacto con un disparador de borde de sombra del diseño de nivel. Variante A sin lámpara (bloqueo); variante B con lámpara. | A: Se detiene y gime. «Frío. Sin luz, no.» B: Se pega a tu pierna. «Luz pequeña. Juntos.» | A: It stops and whines. «Cold. Not without light.» B: It presses against your leg. «Small light. Together.» | Una por intento; enfriamiento por ID |
| LUMA-R-SAMPLE | `SampleCollected { ITEM-SAMPLE-REGOLITH-01 }` | Olfatea la bolsa y estornuda. «Polvo viejo. No huele a nada.» | It sniffs the bag and sneezes. «Old dust. Smells of nothing.» | Una por intento |
| LUMA-R-CACHE | `ItemRecovered { ITEM-AUR-LOG-PARTIAL }` | Apoya la frente en la carcasa marcada y emite un sonido largo. «Ella tocó esto. Su marca.» | It rests its forehead on the marked casing and hums. «She touched this. Her mark.» | Una por intento |
| LUMA-R-RELAY | `LinkRestored` | Levanta los apéndices hacia la antena. «La voz vuelve.» | It raises its feelers to the antenna. «The voice is back.» | Una por intento |
| LUMA-R-LOWLS | `LifeSupportWarning` | Tira suavemente del traje hacia el módulo. «Poco aire. Volver.» | It tugs the suit toward the lander. «Little air. Go back.» | Una por intento; prioridad |
| LUMA-R-DELIVER | `ItemDelivered { ITEM-AUR-LOG-PARTIAL }` | Se sienta junto a la terminal. «Guardado. Para ella.» | It sits by the terminal. «Kept. For her.» | Una por intento |

`LUMA-R-CACHE` reconoce un objeto de la expedición de la hermana: su marca en la carcasa. No menciona destino, supervivientes ni desenlace, y LUMA no aporta coordenadas; la pista procede del registro.

## 10. Informe causal

`ReportReady { attemptId, outcome }` se emite al entrar en REPORT, que solo ocurre al terminar el ascenso o con el comando `GoToReport` tras un fallo o aborto de aproximación. `ApproachFailed` y `ApproachAborted` por sí solos no lo emiten.

### 10.1 Resultado

| Resultado | Condición |
| --- | --- |
| Éxito | `ItemDelivered { ITEM-AUR-LOG-PARTIAL }`, ciencia 3/3 y ascenso terminado. |
| Éxito parcial | Objetivo principal completo y ascenso terminado, con ciencia por debajo de 3/3. El informe dice qué faltó y por qué. |
| Aborto | Aproximación terminada tras fallo o aborto, regreso de emergencia, o `RequestAscent` confirmado sin el registro entregado. |

Puntuación científica (0–3): +1 por `OBS-TERRAIN-01` registrada en el intento, +1 por `OBS-GEO-01` registrada en el intento y +1 por `ITEM-SAMPLE-REGOLITH-01` **entregada** en el intento. Se calcula a partir de conjuntos, nunca con contadores incrementales.

### 10.2 Estructura

1. **Objetivo principal**: registro recogido / enlace restablecido / registro entregado, cada uno con su estado. Línea fija: "Registro parcial de Aurora: entregado, pendiente de autenticación."
2. **Objetivo científico**: puntuación y lista de lo obtenido.
3. **Recursos, previsto frente a real**:

| Recurso | Previsto | Real | Desglose |
| --- | --- | --- | --- |
| Propelente | Necesario del plan | Corrección + aproximación final + ascenso; sobrante | Kg asistidos; reintentos (sin consumo acumulado); carga real del ascenso |
| Soporte vital | Estimación 3.5 | Min usados | Metros caminados, actividades, cancelaciones |
| Energía | Estimación 3.5 | Wh usados | Lámpara, espectrómetro, relé |
| Tiempo en el relé | `relayMin` estimado | Real | Fuente, método de alineación y solapamiento |

4. **Decisiones → consecuencias**: frases de la tabla 10.3 cuya condición se cumple, ordenadas por impacto (primero las que cambian el resultado).
5. **Fuentes y simplificaciones** (10.4).
6. **Comparación con el intento anterior** (10.5) y botones "Reintentar con otra configuración" (comando `RetryPlanning`: `attemptId` + 1, PLANNER con `CP-PLAN` y la configuración precargada) y "Terminar" (`FinishMission`).

### 10.3 Reglas causales

| ID | Condición | ES | EN |
| --- | --- | --- | --- |
| C-01 | Sin `MOD-SCANNER` | No llevabas espectrómetro: el punto geológico no se pudo escanear (ciencia {score}/3). | You had no spectrometer: the geology site could not be scanned (science {score}/3). |
| C-02 | Sin `MOD-LAMP` | Sin lámpara la sombra estaba cerrada: el rodeo iluminado añadió 80 m (16,7 min de soporte vital). | Without a lamp the shadow was closed: the lit detour added 80 m (16.7 min of life support). |
| C-03 | Lámpara usada | La lámpara gastó {lampWh} Wh en {shadowMin} min de sombra. | The lamp used {lampWh} Wh over {shadowMin} min in shadow. |
| C-04 | Relé con batería | La batería alimentó el relé en 1 min y gastó 100 Wh. | The battery powered the relay in 1 min and used 100 Wh. |
| C-05 | Relé con panel | El panel cargó el relé en 10 min; {overlap ? "alineaste mientras cargaba" : "alineaste antes de desplegarlo y no se solaparon"} (total {relayMin} min). | The panel charged the relay in 10 min; {overlap ? "you aligned while it charged" : "you aligned before deploying it, so they did not overlap"} (total {relayMin} min). |
| C-06 | Alineación manual | Sin kit, alinear a mano alargó la reparación {extraMin} min (de {withKitMin} a {relayMin}). | Without the kit, manual alignment lengthened the repair by {extraMin} min (from {withKitMin} to {relayMin}). |
| C-07 | `MidcourseCorrected.kind = late` | Corregiste tarde: la trayectoria costó 5 kg en lugar de 2 y el sobrante para corregir bajó 3 kg. | You corrected late: the trajectory cost 5 kg instead of 2 and your spare for corrections dropped by 3 kg. |
| C-08 | Asistente usado | El asistente gastó {assistKg} kg, {assistExtraKg} más que pilotando a mano. | The assist used {assistKg} kg, {assistExtraKg} more than manual piloting. |
| C-09 | `approachRetries` > 0 | Necesitaste {n} reintentos de aproximación; cada uno partió del propelente del punto de control. | You needed {n} approach retries; each one started from the checkpoint propellant. |
| C-10 | `ApproachAborted.reason = reserve` | La aproximación se abortó: el propelente bajó de la reserva para el ascenso ({pStar} kg). | The approach was aborted: propellant fell below the ascent reserve ({pStar} kg). |
| C-11 | Soporte vital real > previsto × (1 + `reportDeltaNoteFraction`) | Usaste {realMin} min de soporte vital frente a {estMin} previstos: {extraM} m más de recorrido y {cancelMin} min en acciones canceladas. | You used {realMin} min of life support versus {estMin} planned: {extraM} m of extra walking and {cancelMin} min of cancelled actions. |
| C-12 | `MOD-O2-RESERVE` y uso > 120 min | La reserva de soporte vital evitó el regreso de emergencia: usaste {usedMin} min, más que los 120 del traje. | The life-support reserve prevented an emergency return: you used {usedMin} min, more than the suit's 120. |
| C-13 | `MOD-O2-RESERVE` y uso ≤ 120 min | La reserva de soporte vital no fue necesaria: 12 kg y 14 cr que podían ir a otro módulo. | The life-support reserve was not needed: 12 kg and 14 cr that could have gone to another module. |
| C-14 | `EmergencyReturn` | El soporte vital se agotó y volviste automáticamente. {items} no se entregó y no cuenta; tus observaciones se conservan. | Life support ran out and you returned automatically. {items} was not delivered and does not count; your observations are kept. |
| C-15 | W1 aceptado | Saliste con un margen de propelente del {margin} %, por debajo del 30 % recomendado. | You launched with a {margin}% propellant margin, below the recommended 30%. |
| C-16 | Batería y panel, uno sin usar | Llevaste batería y panel y solo usaste {used}: {unusedKg} kg sin uso. | You carried battery and panel and only used {used}: {unusedKg} kg unused. |
| C-17 | `MOD-TOOLKIT` usado | El kit ahorró {savedMin} min en el relé ({relayMin} en lugar de {manualMin}). | The kit saved {savedMin} min at the relay ({relayMin} instead of {manualMin}). |
| C-18 | `RequestAscent` confirmado sin entregar el registro | Subiste al módulo sin transmitir el registro: el objetivo principal no se completó. | You boarded without transmitting the log: the main objective was not completed. |
| C-19 | Registro entregado | El registro parcial de Aurora llegó a la terminal. Es una pieza: su autenticación requiere más pruebas. | Aurora's partial log reached the terminal. It is one piece: authenticating it needs more evidence. |
| C-20 | W4 aceptado y `EmergencyReturn` | Saliste con aviso de soporte vital ({planPct} %) y el soporte se agotó. | You launched with a life-support warning ({planPct}%) and it ran out. |

Valores de C-06 y C-17: con batería, 6 frente a 16 (10 min); con panel solapado, 12 frente a 17 (5 min).

### 10.4 Fuentes y simplificaciones

Cada cifra mostrada lleva su nivel (O/S/F) y, si es O, enlace a [fuentes](../../04-Ciencia/FUENTES.md):

- **O**: `g0`; `moonGravityMps2` ([NASA-MOON-FACTSHEET](../../04-Ciencia/FUENTES.md)); `cruisePhysicalHours` como referencia histórica de Apollo 11 ([NASA-APOLLO11](../../04-Ciencia/FUENTES.md)), no como trayectoria calculada.
- **S**: ecuación del cohete ([NASA-ROCKET-EQ](../../04-Ciencia/FUENTES.md)) con un `ispS` único para motor y RCS; `ascentDeltaVMps` de juego (un ascenso lunar real necesita del orden de km/s); tiempo comprimido en superficie y crucero; reserva del 15 %; lectura de señal del relé con el receptor del traje; mapa comprimido y artístico.
- **F**: módulo de aproximación, módulos, tanques, presupuesto, soporte vital, energía de equipo, reserva de emergencia del traje, expedición Aurora; LUMA y su traductor (licencia de ciencia ficción, no atribuible a NASA).

Aviso fijo: "Simplificación educativa; no certifica una misión real." / "Educational simplification; it does not certify a real mission."

### 10.5 Comparación con el intento anterior

Se guarda un `AttemptSummary` por intento con configuración, tanque, ruta usada, fuente del relé, método de alineación, corrección, reintentos, resultado, ciencia, propelente sobrante y soporte vital y energía usados. El informe compara solo con el intento inmediatamente anterior de la misma partida:

- Diferencias de decisión: módulos añadidos o quitados, tanque, ruta, solución del relé.
- Diferencias de resultado con signo: "Soporte vital: 81,4 → 94,0 min (+12,6)".
- Una frase que une ambas cuando existe una regla causal aplicable: "Al cambiar la lámpara por el panel, el recorrido creció 80 m y el relé pasó de 16 a 12 min."

## 11. Idempotencia, intentos y progreso

- Cada evento de progreso tiene la clave `eventKey = attemptId : type : subjectId` (ej.: `3:ItemDelivered:ITEM-AUR-LOG-PARTIAL`). El reductor de la simulación ignora una clave ya procesada.
- El estado se guarda en conjuntos, no en contadores: `observations`, `inventory`, `recovered`, `delivered`, `reactionsShown`, `processedEventKeys`. La ciencia y el objetivo principal se derivan de esos conjuntos.
- **Intentos** (GDD): `RetryPlanning` (`REPORT → PLANNER`) abre un `attemptId` nuevo. Recursos, inventario, estado del relé, `lifeSupportWarned` y `reactionsShown` se reinician con el nuevo plan.
- **Conocimiento**: las observaciones registradas se conservan entre intentos en el diario del perfil (`knownObservations`, unión de todos los intentos). La puntuación de cada intento cuenta solo lo registrado en ese intento, para que el informe atribuya bien la causa (sin espectrómetro no hay `OBS-GEO-01` en ese intento).
- **Recompensa de entrega**: una vez por misión (`rewardsGranted`, conjunto de IDs). Entregar de nuevo el registro en otro intento cuenta para el resultado de ese informe, pero la pista y la recompensa no se conceden dos veces.
- La corrección de crucero, el ascenso y el regreso de emergencia tienen una bandera de "resuelto" por intento; recargar no repite su consumo.
- Las actividades en curso no se guardan. Al recargar se vuelve al último hito con su reloj, sus recursos y la posición del lugar del hito. La actividad interrumpida ni se completa ni se cobra de nuevo.
- Autoguardado de hito tras `Landed` (`CP-SURFACE`), `ObservationRecorded`, `SampleCollected`, `ItemRecovered`, `RelayPowered`, `RelayAligned`, `LinkRestored` e `ItemDelivered`, como escritura transaccional ([guardado](../../03-Arquitectura/GUARDADO.md)) y respetando `autosaveMinIntervalS`.

Estado estable a guardar:

| Bloque | Campos |
| --- | --- |
| Intento | `attemptId`, configuración, tanque, estimación congelada, `AttemptSummary` previo |
| Vuelo | `midcourseResolved` y tipo, propelente en `CP-APPROACH`, `approachRetries`, kg asistidos |
| Superficie | reloj de misión, soporte vital restante, `energyWh`, `lifeSupportWarned`, lugar del último hito, metros caminados y en sombra, min cancelados |
| Relé | `powered` y fuente, `aligned` y método, panel desplegado, Wh solares acumulados |
| Progreso | conjuntos de la lista anterior; `AuroraEvidenceState` con la pieza "recuperada", nunca "validada" |
| Perfil | `rewardsGranted`, `knownObservations`, opciones de accesibilidad |

No se guardan velocidades, burbujas visibles, colas de reacciones ni objetos del motor.

## 12. Accesibilidad mecánica

- En superficie no hay acciones de reflejos: actividades de una pulsación, alineación por pasos con barrido automático, sin temporizadores de habilidad.
- Esc pausa siempre: crucero, aproximación, superficie y actividades. Los paneles de decisión congelan el reloj de misión.
- Todas las duraciones se muestran en min de misión y s reales; los recursos, con número, barra y texto.
- Opción `pauseOnDecisions` (desactivada por defecto): congela el crucero mientras la decisión de corrección está abierta.
- La aproximación es la única fase con pilotaje; el asistente y los reintentos sin coste acumulado están en [físicas](FISICAS.md).
- Las reacciones de LUMA no son modales y se pueden ocultar con una opción; no contienen información necesaria para jugar.
- Los bloqueos (sombra, terminal sin enlace, falta de equipo, ascenso con objetivo incompleto) siempre dicen la causa y una alternativa.

## 13. Casos de prueba de reglas

Aptos para Vitest sobre funciones puras. Tolerancia: ±0,01 kg, ±0,01 min y ±0,01 Wh sobre valores sin redondear; los textos se comprueban por código e interpolación.

| ID | Entrada | Resultado esperado |
| --- | --- | --- |
| T-MEC-01 | A + TANK-M | carga 40, coste 57, necesario 89,23, exigido 102,62, margen 34,48 %; errores []; avisos [W5]. |
| T-MEC-02 | A + TANK-S | coste 53; necesario 84,69; exigido 97,39 > 96 → [V5]; avisos [W1 (13,36 %), W5]. |
| T-MEC-03 | A + TANK-L | coste 61; exigido 109,15 (se muestra "109,1"); margen 58,05 %; errores []; avisos [W5]. |
| T-MEC-04 | B + TANK-M | carga 51, coste 79, exigido 105,01, margen 31,42 %; errores []; avisos [W3]. |
| T-MEC-05 | B + TANK-S | coste 75; necesario 86,77; exigido 99,79 > 96 → [V5]; avisos [W1 (10,64 %), W3]. |
| T-MEC-06 | B + TANK-L | coste 83; exigido 111,54; margen 54,65 %; errores []; avisos [W3]. |
| T-MEC-07 | Todo + TANK-M | carga 71, coste 99 → [V1]; sin V2; W1 (26,18 %). |
| T-MEC-08 | Todo + TANK-L | coste 103 → [V1, V2]; margen 48,84 %, sin W1. |
| T-MEC-09 | Todo + TANK-S | coste 95 → [V1, V5] (exigido 104,14); W1 (6,01 %). |
| T-MEC-10 | Estado inicial (LUMA-SHIELD + TANK-M) | errores [V3, V4]; avisos [W2, W3, W5]; sin V5 (exigido 95,43); estimación nula y W4 sin evaluar. |
| T-MEC-11 | DATA-READER, LUMA-SHIELD, SCANNER, LAMP + TANK-M | errores [V4]; mensaje con "100 Wh" y "60 Wh". |
| T-MEC-12 | Comando `DeselectModule(MOD-LUMA-SHIELD)` en PLANNER; configuración importada sin LUMA-SHIELD | El comando se rechaza: estado sin cambios, ningún `ModuleDeselected`, mensaje de preinstalado en `lastRejection`. La configuración importada da [V6]. |
| T-MEC-13 | Frontera de TANK-S: BATTERY-PACK + LAMP (32 kg) / BATTERY-PACK + SCANNER (34 kg) | 32 kg: válida, exigido 95,65, margen 15,42 %, avisos [W1, W2, W5]. 34 kg: [V5] (exigido 96,08 > 96). |
| T-MEC-14 | 59 kg (todo salvo O2-RESERVE) + TANK-M | válida; W1 con margen 29,27 %. |
| T-MEC-15 | `estimateSurface(A)` | 237 m, 49,375 min caminando, 32 min de actividades, 81,375 min, capacidad 120, 67,81 % (sin W4); energía 123,72 de 460 Wh. |
| T-MEC-16 | `estimateSurface(B)` | 317 m, 66,04 min, 28 min, 94,04 min, capacidad 180, 52,25 %; energía 1,5 de 60 Wh. |
| T-MEC-17 | B sin O2-RESERVE | 94,04 / 120 = 78,37 % → W4. |
| T-MEC-18 | Propiedad W4 sobre todas las combinaciones sin V1 ni V4 | sin LAMP ni O2-RESERVE: W4 siempre (70,9–82,5 %); con LAMP o con O2-RESERVE: nunca (máximo 68,6 %). |
| T-MEC-19 | Sin lámpara, movimiento hacia una celda de sombra | posición final en la última celda iluminada; `blockedBy = shadow`; energía sin cambio; `LUMA-R-SHADOW` (variante A) una sola vez aunque se insista. |
| T-MEC-20 | Con lámpara, cruzar 40 m de sombra | 8,33 min; −11,11 Wh. |
| T-MEC-21 | Con lámpara y `availableWh` = 10 frente a un tramo de 40 m | entrada bloqueada (`blockedBy = shadowEnergy`) con el mensaje de energía insuficiente. |
| T-MEC-22 | Batería sin panel, relé pendiente, `energyWh` = 460 | `reservedRelayWh` = 100; lámpara y espectrómetro no bajan `energyWh` de 100. |
| T-MEC-23 | Despliegue del panel iniciado en t0 y alineación iniciada al terminar el despliegue | Con kit: `RelayAligned` en t0 + 7; `RelayPowered` y `LinkRestored` en t0 + 12. A mano: `RelayPowered` en t0 + 12; `RelayAligned` y `LinkRestored` en t0 + 17. |
| T-MEC-24 | Alineación manual iniciada en t0 y despliegue del panel al terminarla | `LinkRestored` en t0 + 15 + 2 + 10 = t0 + 27; el informe incluye C-05 sin solapamiento. |
| T-MEC-25 | Batería + manual / batería + kit | `LinkRestored` a +16 / +6 min; −100 Wh. |
| T-MEC-26 | Generador solar con iluminación = sombra | 0 W. |
| T-MEC-27 | Crucero: decisión "ahora" / sin respuesta hasta `midcourseDecisionCloseS` | −2 kg `kind: now` / −5 kg `kind: late` a los 20 s; recargar tras el evento no resta otra vez. |
| T-MEC-28 | Reserva: p* para A y B; A con TANK-M y propelente 88,64 / 88,66 kg | p* = 88,65 y 91,49; aborta / continúa; sobrante en CP-APPROACH 29,35 (A) y 26,51 (B). |
| T-MEC-29 | A + TANK-M: gasta 20 kg y `ApproachFailed`; o `ApproachAborted { reserve }` | El modo sigue en APPROACH sin `ReportReady`. `RetryApproach` restaura 118,0 kg y `approachRetries` = 1. `GoToReport` pasa a REPORT y emite un `ReportReady { outcome: abort }`. |
| T-MEC-30 | Jugador en POI-05 con lámpara (101 m hasta POI-01) y 31 / 32 min de soporte vital | `LifeSupportWarning` (umbral 31,30) / sin aviso. Un segundo cruce no emite otro evento. |
| T-MEC-31 | El soporte vital llega a 0 con el registro recogido, la muestra entregada y OBS-TERRAIN-01 | `EmergencyReturn` hacia POI-01; resultado Aborto; `delivered` = {muestra}; ciencia 2/3; C-14 menciona el registro. |
| T-MEC-32 | `ItemDelivered(ITEM-AUR-LOG-PARTIAL)` procesado dos veces; después recarga del hito y nueva entrega | un único elemento en `delivered`; `rewardsGranted` sin duplicados; un solo `LUMA-R-DELIVER`. |
| T-MEC-33 | Terminal con solo el registro y sin `LinkRestored` | acción rechazada, 0 min consumidos, el registro sigue en el inventario. |
| T-MEC-34 | Entrega del registro | `AuroraEvidenceState`: pieza recuperada; AUR-E03 sin validar; destinos revelados sin cambios. |
| T-MEC-35 | Entrega del registro y después 5 min sin acción | el modo sigue en SURFACE y el soporte vital baja 5 min. `RequestAscent` en POI-01 → ASCENT sin confirmación. |
| T-MEC-36 | `RequestAscent { confirmed: false }` sin el registro entregado; después `{ confirmed: true }` | El primero se rechaza: confirmación con la lista pendiente y SURFACE sin consumo. El segundo pasa a ASCENT, resultado Aborto y C-18. |
| T-MEC-37 | 60 s reales en `PAUSE` o `DIALOGUE` en superficie | soporte vital, energía y carga solar sin cambio. |
| T-MEC-38 | Recorrido ideal A / B completo | soporte vital restante 38,625 / 85,96 min; sin `LifeSupportWarning`; Éxito con ciencia 3/3 tras `RequestAscent`. |
| T-MEC-39 | Mismo recorrido sin MOD-SCANNER | ciencia 2/3; Éxito parcial; C-01 presente. |
| T-MEC-40 | Dos `ObservationRecorded(OBS-TERRAIN-01)` en el mismo intento | un elemento en `observations`; una sola `LumaReaction(LUMA-R-VIEWPOINT)`. |
| T-MEC-41 | Intento 2 tras entregar el registro en el intento 1 | `knownObservations` conservadas; recursos, inventario y relé reiniciados; nueva entrega del registro sin segunda recompensa. |
| T-MEC-42 | Ascenso real de A con TANK-M tras una aproximación nominal, con ambos objetos / sin ninguno | 81,26 kg con 30,77 restantes / 80,73 kg con 31,31 restantes. |
| T-MEC-43 | B con kit y relé por panel solapado | C-17 con 5 min (12 en lugar de 17); C-06 ausente. |
| T-MEC-44 | `LUMA-R-SHADOW` variante B pedida 10 s después de mostrarse la variante A | se descarta: ya se mostró en el intento y no han pasado `lumaReactionCooldownS`. |

## 14. Parámetros propuestos

Ya están en PARAMETROS y se usan por nombre: `lifeSupportPlanWarnFraction`, `relaySolarChargeMin`, `midcourseDecisionOpenS`, `midcourseDecisionCloseS`, `relayAlignPositions`, `lumaReactionMinGapS`, `lumaBubbleS`, `lumaReactionCooldownS` y `reportDeltaNoteFraction`.

Pendientes de incorporar:

| Nombre | Valor | Unidad | Nivel | Motivo |
| --- | --- | --- | --- | --- |
| returnCargoLogKg | 1 | kg | F | PARAMETROS dice que el ascenso real usa la carga que se lleve (0–3 kg), pero no reparte `returnCargoKg` entre los objetos. Debe cumplirse `returnCargoLogKg + returnCargoSampleKg = returnCargoKg`. |
| returnCargoSampleKg | 2 | kg | F | Ídem. |

Además, sin valor de balance: los nombres en inglés de los módulos (sección 3.1) y la opción de accesibilidad `pauseOnDecisions`.
