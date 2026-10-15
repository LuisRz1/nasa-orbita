---
id: historia-principal
type: narrativa
status: especificacion-con-spoilers
updated: 2026-10-14
tags: [orbita, historia, aurora, luma, spoilers]
---
# Historia principal — Órbita: La última luz

**SPOILERS de la campaña completa.** Documento de diseño narrativo; no es un guion de producción ni contenido implementado.

## La razón para partir y volver

**Encontrar a la expedición desaparecida Aurora, rescatar a la hermana del protagonista y traer a la tripulación superviviente a la Tierra.**

La ciencia permite seguir su rastro y preparar un rescate viable. Cada misión principal debe revelar una pista, abrir una ruta o aportar una capacidad necesaria para recuperar a las personas. El regreso es la culminación del rescate.

## Premisa sin revelar el final

En un futuro cercano hipotético, Aurora desapareció después de desviarse para investigar una señal desconocida. Tu hermana formaba parte de la tripulación. La búsqueda no logró localizarla y se suspendió.

Meses después, una estación cercana a la Tierra recupera una cápsula de emergencia identificada como parte de Aurora. Dentro hay registros de navegación dañados y un pequeño animal luminoso: LUMA.

La cápsula prueba que alguien sobrevivió al incidente el tiempo suficiente para enviarla, pero no demuestra todavía que siga con vida. El programa autoriza una misión para reconstruir la ruta y buscar supervivientes. Tú, piloto de una nave de apoyo con formación para esas operaciones, quedas a cargo del vehículo de recuperación. Control de misión, estaciones y depósitos ofrecen apoyo; la misión tiene autorización, plan y límites de recursos.

## Personajes y relación

- **Protagonista:** piloto competente, con un motivo personal y responsabilidad sobre la seguridad de quienes debe rescatar. Nombre y aspecto se definirán durante producción.
- **Hermana:** integrante de Aurora con conocimientos de comunicaciones y operación de campo. Sus registros muestran decisiones, dudas y esfuerzos por mantener a sus compañeros a salvo. Tiene agencia antes y después del reencuentro.
- **LUMA:** animal extraterrestre ficticio que viajó con Aurora y fue enviado en la cápsula protegida. Su traductor interpreta sonidos y gestos. Recuerda experiencias y personas, pero no conoce coordenadas precisas ni repara datos mediante poderes.
- **Tripulación de Aurora:** supervivientes en un refugio conectado a su nave averiada. El manifiesto completo y el número de personas serán parámetros escritos de contenido, visibles antes de validar la capacidad de evacuación. El rescate principal incluye a todos los supervivientes confirmados.
- **Control de misión:** da contexto, contrasta registros y ayuda a planificar. La demora de comunicaciones impide tratarlo como una voz omnisciente durante cada maniobra.

## Qué ocurrió con Aurora

Aurora trabajó primero en Marte. Detectó una baliza de procedencia desconocida y desvió su expedición hacia un pequeño cuerpo: Bennu, referencia real elegida para el diseño. Allí recuperó a LUMA en un contenedor biológico protegido. La baliza pertenecía al contenedor; no era una señal emitida por una estrella.

Una avería de la nave dejó a la tripulación aislada junto a un módulo de servicio ficticio. Se refugiaron, reciclaron suministros y enviaron una cápsula de emergencia con LUMA y una copia incompleta de sus registros. También transmitieron un mensaje fechado que el relé marciano almacenó sin llegar a reenviarlo a la Tierra. Los datos anteriores al desvío permanecieron en los equipos de campo de Marte. La cápsula volvió a una ruta de recuperación terrestre, mientras los enlaces de Aurora fallaron.

La causa técnica concreta de la avería y el calendario de vuelos se detallarán y validarán durante implementación. El diseño requiere que la autonomía del refugio, la llegada de la cápsula y las ventanas del rescate sean compatibles. No se afirmará que una ruta ficticia ya ha sido calculada.

El origen remoto de LUMA puede seguir siendo una pregunta científica. El final sí resuelve qué fue la señal, dónde está Aurora, quién sobrevivió y qué se logró rescatar. No depende de una secuela para entender el objetivo principal.

## Los cuatro capítulos

| Capítulo | Pregunta que impulsa al jugador | Respuesta al cerrarlo | Estado de LUMA |
| --- | --- | --- | --- |
| [1. Primera luz](CAPITULOS/C1-PRIMERA-LUZ.md) | ¿La cápsula pertenece a Aurora y por dónde empezar? | Identidad validada; el archivo de un relé lunar conserva el siguiente tramo. | Blanco-dorado, curioso; aprende rutinas y acepta al jugador. |
| [2. Bajo otro cielo](CAPITULOS/C2-BAJO-OTRO-CIELO.md) | ¿Qué perdió el enlace lunar y adónde fue Aurora? | Un registro recuperado y el relé reparado confirman la escala marciana. | Naranja; necesita más descanso y reconoce molestias. |
| [3. El mundo rojo](CAPITULOS/C3-EL-MUNDO-ROJO.md) | ¿Por qué se desvió y queda alguien a quien rescatar? | Archivo de campo y mensaje de emergencia verifican Bennu y supervivientes, incluida la hermana. | Rojo; el deterioro se reconoce y los cuidados lo alivian. |
| [4. Lo que queda de la luz](CAPITULOS/C4-LO-QUE-QUEDA.md) | ¿Podemos alcanzarlos y llevarlos de vuelta? | Tripulación recuperada y regreso terrestre completado; LUMA se sacrifica para salvar al protagonista. | Blanco azulado; despedida íntima y muerte definitiva. |

Ruta: **Tierra → Luna → Marte → Bennu → Tierra**. El nodo marciano sirve para reabastecer y reconfigurar el rescate. Las aproximaciones se pilotan en 2.5D; los trayectos largos se planifican y aceleran. No hay una escala terrestre intermedia.

## Cadena de pruebas y revelaciones

Los siguientes IDs son referencias documentales para contratos, diario, guardado y QA. Solo un evento de validación escrito puede confirmarlos.

| ID de evidencia | Dónde se obtiene y valida | Qué permite afirmar |
| --- | --- | --- |
| AUR-E01 | Cápsula y registro de recepción; C1-M03 y M04. | Es una cápsula de Aurora; LUMA viajó con la expedición. No prueba supervivencia actual. |
| AUR-E02 | Registrador exterior de la estación; C1-M05. | Parte del envío puede reconstruirse desde el archivo lunar. |
| AUR-E03 | Registro de campo lunar, caché protegida y relé; C2-M03–M06. | Aurora pasó por Marte; las piezas se autentican juntas al cerrar el informe. |
| AUR-E04 | Hábitat y archivo de campo de Aurora; C3-M03 y M04. | La desviación siguió una baliza desconocida; los datos previos al viaje aún no explican la llegada de LUMA. Falta autenticar el destino completo. |
| AUR-E05 | Mensaje recuperado por el relé marciano; C3-M05 y M06. | El mensaje enviado después de la avería registra supervivientes, posición en Bennu y recuperación del contenedor de LUMA, con fecha conocida. No es una conversación en directo ni garantía de autonomía infinita. |
| AUR-E06 | Contacto y comprobación presencial; C4-M02 y M03. | Hermana y supervivientes localizados vivos; manifiesto de evacuación confirmado. |
| AUR-E07 | Extracción y recepción terrestre; C4-M04–M06. | Todos los supervivientes confirmados han sido trasladados y recibidos a salvo. |

M03–M05 de la Luna y M04–M05 de Marte mantienen el orden libre especificado. Sus piezas se reúnen en el informe final; encontrar una parte antes no revela conclusiones futuras. Un registro corrupto puede estar recogido sin estar validado.

## Clímax y despedida

El jugador encuentra a su hermana y a los otros supervivientes en C4-M03. Se reconocen, comparten lo que sucedió y participan en la preparación de la salida. En M04 todos los rescatados quedan a bordo con soporte de vida y reservas comprobados.

Durante la última desconexión del módulo, el mecanismo de retorno se atasca y aísla al protagonista en el lado exterior. El accionamiento manual está al otro lado de un conducto que una persona con traje no puede atravesar. La capacidad de LUMA para recorrerlo con protección se ha mostrado antes. Las herramientas disponibles no alcanzan el accionamiento desde esa posición; no se retiran arbitrariamente mejoras compradas para forzar el momento.

LUMA decide cruzar y liberar el retorno. No se le ordena morir ni hay una decisión oculta de afecto que pueda salvarlo. El protagonista alcanza a recogerlo y llevarlo al compartimento seguro. El esfuerzo agrava su desgaste de forma irreversible. Muere acompañado, después de haber salvado al jugador; la extracción de Aurora puede concluir.

La hermana participa en el regreso y en la despedida posterior. La misión de rescate se cumple. La pérdida de LUMA conserva su peso: no hay resurrección, copia consciente ni chat nuevo que continúe su personalidad. Ver [arco de LUMA](ARCO-LUMA-SPOILERS.md).

## Epílogo y exploración posterior

En la Tierra se recibe a los supervivientes y se entregan datos, muestras y registros. El informe distingue rescate humano, conocimiento científico y lo que todavía se desconoce sobre LUMA. Su lugar de descanso en la nave queda como recuerdo.

Continuar la partida permite expediciones y colecciones pendientes con las herramientas del jugador. Son viajes posteriores al rescate, preparados desde nodos de apoyo; no teletransportes al pasado. Las grabaciones de LUMA se identifican como archivos. La hermana y los demás personajes vivos pueden reconocer lo ocurrido mediante diálogos escritos.

## Reglas de ritmo y alcance

- La urgencia se comunica por hitos narrativos y reservas de expedición. Las secundarias no matan a LUMA ni a la tripulación por un reloj real oculto.
- Los cruceros muestran tiempo físico y de juego. La progresión no presupone viajes instantáneos ni atención médica humana realista ya verificada para biología alienígena.
- Una mala maniobra admite reintento. No sustituye el final escrito por una muerte aleatoria.
- Ninguna pista esencial depende de coleccionables, IA online o una puntuación perfecta.
- La demo lunar solo recupera una parte del rastro de Aurora y cierra su misión local. No incluye el rescate final ni revela la muerte de LUMA.
- Los hechos humanos futuros, el contenedor, la biología de LUMA y el traductor son ficción explícita. Las fuentes científicas describen los cuerpos y fenómenos reales.

## Continuación para agentes

Las [cuatro fichas de capítulo](MUNDO-Y-CAMPANA.md) fijan acciones, pruebas de cierre y desbloqueos de las 24 misiones. El [backlog](../06-Produccion/BACKLOG.md) enlaza sus tareas de producción. Revisar también contratos, guardado y aceptación antes de modificar una revelación o un acontecimiento de LUMA.

Siguiente paso permitido: revisión documental con el equipo y contraste con el enunciado completo. Guion de escenas, sprites, mapas y código se producen dentro de las fases autorizadas.
