# Plan del proyecto: Órbita, La última luz

Versión 1.2 · Decisiones acordadas y propuesta editorial detallada · Actualización documental: 14 de octubre de 2026.

## 1. Resumen y decisiones acordadas

Crear un juego de exploración espacial para navegador de PC, con **pixel art 2.5D isométrico**, astronauta de cuerpo completo, compañero conversable y una campaña cuyo objetivo será **12–20 horas de juego**.

La exploración estará integrada con el diseño de misiones: elegir objetivos, nave, instrumentos y recursos; ejecutar la expedición; evaluar cómo esas decisiones afectaron el resultado. Esto responde al reto [Space Mission Design Game](https://www.spaceappschallenge.org/2026/challenges/space-mission-design-game/).

El equipo previsto es de **2–3 personas**, con ayuda de agentes. Se publicarán el repositorio y la demo; el jugador no necesitará registrarse.

Se trabajará en tres etapas: preparación documental ahora, demo durante la hackathon y expansión posterior. Se acordó conservar la elegibilidad. La [FAQ oficial](https://www.spaceappschallenge.org/resources/participant-faqs/), revisada el 14 de octubre, indica que los equipos no pueden comenzar a trabajar en los retos antes del evento. El enunciado completo se publica el **28 de octubre** y la hackathon será el **14–15 de noviembre de 2026**.

El horario concreto y las condiciones de la sede deberán constar antes de iniciar implementación. La imagen de referencia anuncia el reto y sus recursos; el enunciado oficial prevalece sobre el resumen de esa imagen.

## 2. Diseño del juego

### Experiencia principal

Reconstruir la ruta de la expedición desaparecida Aurora, rescatar a la hermana del protagonista y traer a los supervivientes a la Tierra. El jugador pilota una nave de apoyo en una búsqueda autorizada, controla al astronauta y usa observaciones y recursos para abrir el camino.

Una cápsula recuperada cerca de la Tierra trae a LUMA y registros dañados. La Luna conserva una copia parcial; Marte guarda el motivo del desvío y una prueba fechada de supervivencia. La última expedición permite localizar y evacuar a Aurora. Ver [historia completa y desenlace — SPOILERS](brain/02-Diseno/HISTORIA-PRINCIPAL.md).

1. **Preparar:** escoger objetivo, vehículo de lanzamiento, módulos, instrumentos y reservas.
2. **Viajar:** seleccionar ruta y reservas, pilotar maniobras y aproximaciones 2.5D y acelerar los cruceros largos.
3. **Explorar:** caminar, escanear, recoger muestras, reparar equipos y resolver problemas ambientales.
4. **Decidir:** equilibrar objetivos científicos, seguridad y recursos restantes.
5. **Regresar:** entregar muestras y recibir un informe que explique los resultados.
6. **Progresar:** desbloquear herramientas, rutas, expediciones y conversaciones.

### Mundo y contenido

| Capítulo | Experiencia |
| --- | --- |
| [1. Primera luz — órbita](brain/02-Diseno/CAPITULOS/C1-PRIMERA-LUZ.md) | Preparar nave, validar cápsula de Aurora, conocer a LUMA y recuperar mediante EVA la pista del archivo lunar. |
| [2. Bajo otro cielo — Luna](brain/02-Diseno/CAPITULOS/C2-BAJO-OTRO-CIELO.md) | Alunizar, instalar refugio, recuperar tres piezas del registro y reparar el relé que permite confirmar la escala marciana. |
| [3. El mundo rojo — Marte](brain/02-Diseno/CAPITULOS/C3-EL-MUNDO-ROJO.md) | Reactivar hábitat, investigar el desvío, autenticar un mensaje de supervivientes y reconfigurar la nave de rescate. |
| [4. Lo que queda de la luz — Bennu; spoilers](brain/02-Diseno/CAPITULOS/C4-LO-QUE-QUEDA.md) | Localizar a la hermana y tripulación, evacuar, vivir el sacrificio de LUMA y completar el regreso terrestre. |

La campaña completa tendrá como presupuesto inicial **18 regiones, 24 misiones principales, 32 actividades secundarias y 120 descubrimientos coleccionables**. Son objetivos de producción y duración, todavía sin contenido implementado.

Cada capítulo tiene seis misiones obligatorias con ID, acción, aceptación y aporte al rescate. El diario muestra sus requisitos; secundarias, colecciones y puntuaciones de afecto no bloquean la historia. Ver [distribución y condiciones](brain/02-Diseno/MUNDO-Y-CAMPANA.md).

Ruta narrativa: Tierra → Luna → Marte → Bennu → Tierra. El reabastecimiento previo al rescate ocurre en el nodo marciano. Las ventanas y la autonomía del refugio aún deben validarse; no se han calculado itinerarios reales.

Los escenarios combinarán lugares diseñados manualmente con generación procedural limitada para terreno secundario y variaciones de expediciones. Objetivos, relatos y descubrimientos importantes tendrán diseño individual.

### Movimiento y progresión

- Astronauta completo, ocho orientaciones y animaciones de caminar, desplazamiento lunar, escaneo, recolección, reparación y EVA.
- Cámara isométrica con seguimiento, zoom, profundidad correcta y transparencia de obstáculos que ocultan al personaje.
- Muestras, registros históricos, observaciones, planos y fotografías. Cada colección aportará conocimiento, una mejora, una ruta o una conversación.
- Inventario con apilado, accesos rápidos y transferencia sencilla al vehículo o base.
- Mejoras de instrumentos, autonomía, rover y módulos de misión. Construcción mediante módulos prediseñados.
- Dificultades **Exploración**, con más margen y asistencia, y **Expedición**, con recursos más exigentes.
- Riesgos ambientales y técnicos. Los descubrimientos registrados permanecerán aunque falle una expedición.

### LUMA

Animal extraterrestre ficticio de cuerpo compacto y cúbico. Se comunica con sonidos y gestos mediante un traductor ficticio, acompaña al jugador y necesita protección ambiental. El rover de superficie es un equipo distinto.

Su paleta recorre **blanco-dorado → naranja → rojo → blanco azulado**, inspirada en una estrella similar al Sol. La conducta muestra desgaste progresivo. Es una comparación simbólica, sin temperatura estelar ni supernova.

**SPOILERS de campaña:** LUMA llegó en la cápsula de Aurora y forma un vínculo mediante rutinas, cuidados y recuerdos. El deterioro avanza por hitos, sin castigar la exploración opcional. Su sacrificio salva al protagonista cuando los supervivientes ya están a bordo; la muerte es definitiva y el epílogo permite seguir explorando sin él.

Los diálogos escritos cubren la historia; la IA online es opcional. Ver [LUMA](brain/02-Diseno/LUMA.md) y [arco narrativo con SPOILERS](brain/02-Diseno/ARCO-LUMA-SPOILERS.md).

La IA no otorgará objetos, completará misiones ni controlará el progreso. Su indisponibilidad no bloqueará la partida.

### Ciencia

Futuro cercano hipotético basado en conocimiento actual del Sistema Solar. Los datos observados tendrán fuentes consultables y las bases y expediciones inventadas se identificarán como ficción.

La especie de LUMA y su traductor son licencias de ciencia ficción explícitas, no descubrimientos de NASA. Bennu es la referencia real del capítulo final; los equipos e itinerarios humanos son hipotéticos.

Gravedad, iluminación, energía, combustible y comunicaciones influirán en las decisiones. Las distancias y tiempos se comprimirán para jugar, mostrando también las magnitudes reales.

Se evaluarán datos de NASA, topografía lunar de [LOLA](https://pds-geosciences.wustl.edu/missions/lro/lola.htm), datos marcianos de MOLA y efemérides de [JPL Horizons](https://ssd-api.jpl.nasa.gov/doc/horizons.html). Se procesarán y empaquetarán durante implementación; las partidas no dependerán de APIs en directo.

### Referentes y críticas

La investigación abarcará No Man's Sky, Outer Wilds, Elite Dangerous, Starfield, Kerbal Space Program 1 y 2, Astroneer, Space Engineers, Star Citizen, Everspace 2, Empyrion, The Planet Crafter, FTL, Starbound, Space Haven y Out There.

El dossier registrará versión, fecha, fortalezas, críticas y decisiones aplicables. Separará reseñas históricas de observaciones recientes. Es una muestra por categorías; no una auditoría de todos los juegos publicados.

| Problema | Respuesta propuesta |
| --- | --- |
| Repetición al explorar | Descubrimientos con consecuencias y regiones con mecánicas distintas. |
| Inventario incómodo | Apilado, transferencia rápida y pocas categorías. |
| Controles frustrantes | Introducción gradual, ayudas opcionales y acciones consistentes. |
| Viajes que fragmentan la experiencia | Transiciones breves con contexto y decisiones durante el trayecto. |
| Complejidad difícil de aprender | Explicaciones visuales y comparación inmediata de alternativas. |

Estas decisiones son hipótesis de diseño que deberán probarse. Véase la [investigación y sus fuentes](brain/01-Producto/INVESTIGACION-JUEGOS.md).

## 3. Tecnologías y arquitectura

| Área | Elección |
| --- | --- |
| Motor | Phaser 4.2.1 con WebGL y mapas isométricos. |
| Desarrollo | TypeScript 7, Vite 8, Node.js 24 y pnpm. |
| Interfaz | React 19, HTML y CSS para menús, inventario y planificación. |
| Estado y validación | Zustand y Zod. |
| Guardado | IndexedDB con Dexie; exportación e importación. |
| Arte | Tiled para mapas y LibreSprite para sprites. |
| Pruebas | Vitest y Playwright. |
| Publicación | Vercel conectado a GitHub. |

Las versiones son la base investigada el 11 de octubre de 2026. Al abrir la fase de implementación se verificará compatibilidad y se fijarán versiones exactas en un lockfile; una modificación de motor o arquitectura requerirá ADR.

La arquitectura será un **monolito modular**:

- **Simulación:** recursos, desplazamiento, riesgos y evaluación; independiente del renderizado.
- **Presentación:** escenas Phaser, animaciones, cámara y sonido.
- **Interfaz:** planificación, inventario, mapa, diario y conversaciones.
- **Contenido:** misiones, regiones, objetos y diálogos en JSON validado.
- **Persistencia:** partidas versionadas, respaldo y migraciones.
- **Servidor opcional:** conversación libre; el juego se ejecuta en el navegador.

Los contratos principales serán MissionDefinition, ChapterDefinition, RegionDefinition, CollectibleDefinition, DialogueNode, ScienceSource y SaveGame, con IDs estables. Se añaden estado narrativo del compañero, pruebas validadas de Aurora, manifiesto de rescate, vuelo local y tramos de crucero separados. Recoger un registro dañado no equivale a haber confirmado su contenido.

El [pilotaje](brain/02-Diseno/VIAJE-ESPACIAL.md) usa posición y velocidad relativas. Los cruceros contabilizan tiempo físico y recursos por tramo, sin esperar días o meses reales. Ver [ADR 004](brain/03-Arquitectura/ADR/ADR-004.md).

El servicio online previsto es **POST /api/companion/chat**. Recibirá mensaje y contexto limitado; devolverá texto y referencias científicas.

Se prevén Vercel AI SDK 7 y AI Gateway, con google/gemini-3.8-flash como modelo base investigado. Solo se activará si la cuenta permite utilizarlo con créditos gratuitos. Sin acceso gratuito, con límites o ante un error, continúan los diálogos escritos. **No se comprarán créditos ni se habilitarán recargas.** La disponibilidad de un modelo y el saldo de la cuenta todavía no están verificados para la entrega.

### Computadora y herramientas

Equipo de referencia inspeccionado el 11 de octubre: Ryzen 7 4800H, 16 GB de RAM, GTX 1650 Ti de 4 GB y Windows 10. Ya existen Node, Git, GitHub CLI, Vercel CLI, Chrome y Obsidian.

Las instalaciones futuras se limitarán a Tiled, LibreSprite y dependencias. Se preferirá D: para herramientas y cachés por su mayor espacio disponible.

Objetivos medibles, todavía no resultados: **60 FPS** en el equipo de referencia, modo reducido de 30 FPS, carga inicial comprimida inferior a 15 MB y memoria de pestaña inferior a 500 MB.

## 4. Repositorio y cerebro

Repositorio público **LuisRz1/nasa-orbita**, rama principal main, documentación en español y nombres de código en inglés.

El vault Obsidian estará en la carpeta **brain**, dentro del mismo repositorio. Usará Markdown y enlaces relativos compatibles con GitHub, sin plugins obligatorios.

Contendrá producto, diseño, arquitectura y ADR, ciencia, arte, producción, calidad, investigación, bitácora y plantillas.

AGENTS.md definirá el orden de lectura. Cada agente deberá revisar estado y decisiones, elegir una tarea, respetar dependencias, verificar resultados y dejar el siguiente paso.

Cada tarea incluirá ID, responsable, prioridad, dependencias, área afectada, aceptación y comprobaciones. Cada cierre registrará commit, resultado probado, limitaciones y siguiente paso.

GitHub Issues será la fuente del backlog operativo; el cerebro aportará contexto y enlaces. El estado distinguirá lo implementado, lo verificado y lo pendiente.

Código propio bajo MIT; documentación y recursos creativos originales bajo CC BY 4.0. Material externo con su licencia y atribución.

## 5. Entregas, pruebas y supuestos

| Etapa | Entrega |
| --- | --- |
| Preparación | Repositorio documental, vault, investigación, arquitectura, especificaciones y reparto propuesto. |
| 28 de octubre | Revisar el enunciado completo y ajustar la matriz de requisitos. |
| 13 de noviembre | Revisar guías, evaluación y horario autorizado de la sede. |
| 14–15 de noviembre | Construir y publicar la demo dentro del periodo permitido. |
| Expansión posterior | Completar Luna, Marte, órbita y asteroide por hitos de calidad. |

La demo tendrá **15–25 minutos**, con presentación de unos cinco minutos: planificación, tramo acelerado, aproximación pilotada breve, región lunar compacta de seis lugares, LUMA, incidencia, regreso e informe. Recupera una parte del rastro de Aurora y cierra ese objetivo local. Es un fragmento del capítulo 2; no completa el capítulo ni revela el cierre de la campaña.

Reparto propuesto: propietario en programación e integración; segunda persona en arte, mapas e interfaz; tercera persona, si participa, en ciencia, narrativa, pruebas y presentación. Con dos personas, esta última responsabilidad se comparte.

La aceptación exigirá:

- Completar el ciclo de misión y explicar el efecto de elegir otro equipo.
- Movimiento legible, colisiones fiables y visibilidad del personaje.
- Guardar, recargar y recuperar sin duplicar hallazgos.
- Continuar sin red, APIs científicas o IA online.
- Fuentes científicas consultables y ficción identificada.
- Pruebas de recursos, misiones, guardado y progresión; recorrido completo en navegador.
- Pruebas con jugadores nuevos para medir duración, orientación, repetición y dificultad.

Valores por defecto: **un jugador; PC; teclado y ratón con mando; español e inglés; guardado local; IA sin pagos**.

La campaña de 12–20 horas es el objetivo posterior al evento, no una promesa para la demo. No se ha fijado fecha final de lanzamiento. Las cuotas de contenido y los presupuestos de rendimiento se revisarán mediante evidencia, dejando constancia de cualquier cambio.
