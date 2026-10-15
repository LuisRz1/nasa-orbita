# Órbita: La última luz

Juego de exploración espacial para navegador, con pixel art 2.5D, pilotaje, exploración planetaria y un animal extraterrestre acompañante llamado **LUMA**.

Una cápsula de la expedición desaparecida Aurora regresa con LUMA y registros dañados. Tu hermana viajaba en esa expedición. Debes reconstruir su ruta, encontrar a los supervivientes y traerlos a la Tierra.

**Estado: preparación documental. Todavía no existe una versión jugable.** El objetivo es construir una demo durante NASA Space Apps 2026 y ampliar después el juego hasta una campaña de 12–20 horas.

## Empieza aquí

| Necesitas… | Abre… |
| --- | --- |
| Compartir la propuesta completa | [Plan del proyecto](PLAN.md) |
| Ver capítulos, misiones y requisitos | [Campaña](brain/02-Diseno/MUNDO-Y-CAMPANA.md) |
| Leer la historia completa y el desenlace | [Historia principal — SPOILERS](brain/02-Diseno/HISTORIA-PRINCIPAL.md) |
| Conocer el vínculo y el final de LUMA | [Arco de LUMA — SPOILERS](brain/02-Diseno/ARCO-LUMA-SPOILERS.md) |
| Conocer el pilotaje y los trayectos | [Viaje espacial](brain/02-Diseno/VIAJE-ESPACIAL.md) |
| Recorrer el cerebro del proyecto | [Inicio del vault](brain/00-Inicio/INICIO.md) |
| Saber qué está hecho y qué sigue | [Estado actual](brain/00-Inicio/ESTADO-ACTUAL.md) |
| Trabajar como agente o colaborador | [Instrucciones de trabajo](AGENTS.md) |
| Consultar tareas y dependencias | [Backlog](brain/06-Produccion/BACKLOG.md) |
| Revisar investigación de juegos | [Comparativa de referentes](brain/01-Producto/INVESTIGACION-JUEGOS.md) |

## El juego

Preparar una misión → viajar → explorar → tomar decisiones → regresar → entender los resultados → desbloquear nuevas expediciones.

La campaña recorrerá órbita terrestre, Luna, Marte y un asteroide con **cuatro capítulos y 24 misiones principales**. Cada misión revela una pista, abre una ruta o permite rescatar a Aurora. Las fichas especifican acciones, pruebas de cierre y desbloqueos. El astronauta se verá completo en vista isométrica. Habrá muestras, observaciones, registros históricos, mejoras y conversaciones reactivas.

Controlarás maniobras, aproximaciones y acoplamientos en 2.5D. Los cruceros largos tendrán planificación y tiempo acelerado. Cada capítulo muestra qué objetivos hacen falta para avanzar.

LUMA es una criatura viva con traductor ficticio. Su color evoluciona con los capítulos inspirado en una estrella similar al Sol. La documentación del cierre está marcada con spoilers.

La ambientación será un futuro cercano hipotético. Los datos científicos tendrán fuentes; bases, misiones humanas futuras, especie de LUMA y traductor estarán identificados como ficción.

## Tecnología acordada

Phaser 4.2.1, TypeScript, Vite, React, Zustand, Zod e IndexedDB con Dexie. Tiled para mapas y LibreSprite para sprites. Vercel para la publicación y un servicio opcional de conversación online. El juego completo deberá poder jugarse sin pagar IA.

Las versiones investigadas, sus límites y la instalación futura están en [Entorno](brain/06-Produccion/ENTORNO.md). No hay dependencias que instalar ni comandos de arranque de juego en esta fase.

## Calendario y elegibilidad

- **28 de octubre de 2026:** publicación del enunciado completo.
- **13 de noviembre:** revisión de guías de entrega, evaluación y horario de la sede.
- **14–15 de noviembre:** desarrollo de la demo durante el periodo autorizado.
- **Después del evento:** expansión por hitos hasta la campaña completa.

La FAQ oficial indica que los equipos no pueden empezar a trabajar en los retos antes de la hackathon. Se acordó conservar la elegibilidad y limitar esta etapa a investigación, planificación y documentación. La preparación documental no constituye una certificación de elegibilidad: se revisarán las condiciones vigentes y las instrucciones de la sede. [FAQ oficial](https://www.spaceappschallenge.org/resources/participant-faqs/).

## Abrir el cerebro en Obsidian

1. Clona o descarga este repositorio.
2. En Obsidian, selecciona **Abrir carpeta como bóveda**.
3. Elige la carpeta **brain**, no una bóveda personal existente.
4. Abre [INICIO](brain/00-Inicio/INICIO.md).

No requiere plugins. Los enlaces también se pueden recorrer desde GitHub. Las preferencias personales de Obsidian quedan fuera de Git.

## Colaborar y compartir

Repositorio público: [LuisRz1/nasa-orbita](https://github.com/LuisRz1/nasa-orbita).

El trabajo se organizará con Issues, ramas y pull requests. Consulta [CONTRIBUTING](CONTRIBUTING.md) antes de empezar. Los responsables humanos de las tareas todavía están por asignar.

Proyecto independiente preparado para un reto de NASA Space Apps; no es un producto oficial ni implica aprobación de NASA.

## Licencias

El código original que se incorpore se publicará bajo [MIT](LICENSE). La documentación y los recursos creativos originales se publican bajo [CC BY 4.0](LICENSE-CONTENT.md). El material de terceros conserva sus condiciones: [atribuciones y procedencia](THIRD-PARTY-NOTICES.md).
