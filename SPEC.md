# La Chusma — Especificación para Claude Code (versión 1)

## 1. Cómo usar este documento

Este documento es la especificación que se le entrega a Claude Code para construir la primera versión jugable de "La Chusma: secretos de barrio": un caso completo, en español e inglés, con anuncios simulados y la estructura lista para crecer. Va en la raíz del repositorio como `SPEC.md`, junto con el documento de diseño general como `DESIGN.md` (si todavía no existe, subí el Word original y pedile a Claude Code que lo convierta).

### Primer mensaje sugerido para Claude Code

```text
Leé SPEC.md y DESIGN.md completos. Vamos a construir la versión 1 de La Chusma siguiendo el plan por fases de la sección 10 de SPEC.md. Antes de escribir código, proponé la estructura del proyecto y la lista de escenas, y esperá mi OK. Después trabajá una fase por vez: al terminar cada una, verificá los criterios de aceptación y resumime qué funciona y qué falta. No integres anuncios reales ni premios en dinero: usá los servicios simulados que describe la spec. No tengo computadora: todo se compila con GitHub Actions como indica la sección 11, y yo pruebo el APK en mi Android.
```

### Objetivo de la versión 1

Una persona que nunca vio el juego lo instala en un Android de gama media, elige idioma, juega el caso 3 "La vecina que se muda" de principio a fin y entiende la revelación final sin ayuda.

### Definición de terminado

- [ ] Exporta un APK de Android que corre a 60 fps estables en un teléfono de gama media (objetivo mínimo 30 fps).
- [ ] Selector de idioma español o inglés en la primera apertura, cambiable desde Opciones.
- [ ] Runner de tres carriles con salto, deslizamiento, obstáculos, monedas, pistas, tres vidas y cronómetro.
- [ ] Caso 3 completo: rumor, briefing, carrera, testimonio, minijuego del flete, entrega con tres versiones y revelación.
- [ ] Oferta de anuncio al inicio de la carrera que duplica las monedas de esa corrida (simulada).
- [ ] Pantallas de resultados y derrota con reintento gratis siempre visible.
- [ ] Guardado local de progreso, monedas, idioma y opciones.
- [ ] Módulo de premios en dinero creado pero apagado por configuración.

## 2. Stack técnico y estructura

La versión 1 se construye en Godot 4 (última versión estable 4.x) con GDScript, exportando primero a Android en vertical. Godot guarda escenas y recursos como texto, lo que permite que Claude Code los cree y edite directamente, y exporta nativo a Android e iOS.

| Decisión | Valor |
| --- | --- |
| Motor | Godot 4.x estable, renderer Mobile |
| Lenguaje | GDScript con tipado estático |
| Plataforma v1 | Android (API mínima según la plantilla de exportación de Godot); iOS en una versión posterior |
| Orientación | Vertical, resolución base 1080 x 1920, escalado `canvas_items` con aspecto `expand` |
| Rendimiento | 60 fps objetivo, menos de 100 draw calls en carrera, texturas de 1024 px como máximo |
| Control de versiones | Git en GitHub, un commit por tarea terminada |
| Compilación | GitHub Actions (sección 11) |

### Estructura de carpetas

```text
la-chusma/
  project.godot
  export_presets.cfg
  SPEC.md
  DESIGN.md
  .github/workflows/android.yml
  autoload/          # singletons globales
    GameState.gd     # progreso, monedas, protagonista activa
    SaveService.gd   # guardado local en user://
    Locale.gd        # idioma y traducciones
    AdService.gd     # interfaz de anuncios (mock en v1)
    PrizeService.gd  # programa de premios (apagado en v1)
    Config.gd        # flags y valores de balance
  data/
    cases/case_03.json
    minigames/truck_03.json
    i18n/strings.csv
    balance.json
  scenes/
    boot/LanguageSelect.tscn  boot/AgeGate.tscn
    menu/Home.tscn  menu/Map.tscn  menu/Settings.tscn
    case/Briefing.tscn  case/Delivery.tscn  case/Results.tscn  case/Defeat.tscn
    run/Run.tscn  run/Player.tscn  run/Segment_*.tscn  run/obstacles/*.tscn
    minigames/TruckPuzzle.tscn
    ui/Hud.tscn  ui/AdOfferCard.tscn  ui/SpeechBubble.tscn  ui/DebugPanel.tscn
  art/
    reference/  characters/  environment/  ui/  shaders/
  audio/
    music/  sfx/
  tests/             # pruebas que corren en GitHub Actions
```

### Reglas de código para Claude Code

- Separar datos narrativos (JSON y CSV) de la lógica: ningún texto visible escrito en el código, siempre una clave de traducción.
- Una máquina de estados explícita para el caso: `INTRO → BRIEFING → RUN → MINIGAME → RUN → DELIVERY → REVEAL → RESULTS`, más `DEFEAT`.
- La lógica de anuncios y premios nunca vive dentro del runner: el runner emite señales (`run_finished`, `coins_collected`) y los servicios escuchan.
- Todos los valores de balance (velocidades, tiempos, multiplicadores, topes) en `data/balance.json`.
- Nombres de código en inglés; textos y comentarios de diseño pueden estar en español.
- Cada fase termina con las pruebas pasando y el APK compilado sin errores en GitHub Actions (sección 11).

## 3. Dirección de arte

El juego tiene estética 3D caricaturesca, como un largometraje animado familiar: personajes de cabeza grande y gestos exagerados, y un barrio de colores saturados con formas simples y bloques limpios. Las cuatro imágenes de referencia aprobadas se guardan en `art/reference/` y son la guía visual obligatoria.

### Las cuatro referencias

| Referencia | Qué tomar de ella |
| --- | --- |
| La protagonista | Vecina de unos 60 años, rodete gris con mechones sueltos, anteojos de marco grueso, aros de perla, camisa rosa abotonada, jean celeste arremangado, cinturón marrón. Ojos grandes, sonrisa amplia, piel con sombreado suave |
| El barrio | Calle recta vista desde atrás y arriba, casas de dos o tres pisos en rojo, amarillo, verde, rosa y celeste, tanques en los techos, faroles azules, sendas peatonales blancas, colectivos y autos redondeados |
| Los personajes secundarios | Escala más chica y cabezona, casa con techo azul, ladrillo, cerca blanca, plantas y regadera. Estilo acogedor y detallado |
| La acción | Cámara detrás del personaje, calle con carriles marcados, vereda con cerca de madera, faroles, autos que vienen de frente con buena lectura |

### Reglas visuales

- **Formas:** proporciones de 3 a 4 cabezas, manos y pies grandes; edificios de bloques con bordes biselados.
- **Materiales:** colores planos con sombreado suave tipo toon de dos o tres tonos, sin texturas fotográficas.
- **Luz:** sol cálido de tarde, sombras suaves y cortas, cielo celeste claro, niebla de distancia leve para dar profundidad.
- **Contorno:** contorno fino oscuro solo en personajes, obstáculos y pistas para separarlos del fondo.
- **Lectura:** las pistas son siempre violeta brillante con halo dorado; ningún otro objeto usa ese color.
- **Sin marcas reales:** carteles, colectivos y negocios con nombres inventados.

### Paleta base

| Uso | Hex |
| --- | --- |
| Fucsia acción (botones principales) | `#E8367C` |
| Violeta pista | `#8B4FD8` |
| Dorado moneda | `#F2B33D` |
| Azul tinta (textos e íconos) | `#1F2A5C` |
| Crema (fondos de interfaz) | `#FFF4E0` |
| Naranja peligro | `#FF8A1F` |
| Fachadas | `#D9714E`, `#E8A0A0`, `#8FD3C7`, `#F6E39B`, `#5BA4E6` |
| Cielo | `#BFE3F2` |

### Cómo lograrlo en la versión 1

Claude Code no puede modelar personajes de calidad cinematográfica, así que la v1 se arma en tres capas que se pueden reemplazar sin tocar la lógica:

1. **Prototipo:** formas primitivas de Godot (cajas, cápsulas, esferas) con los colores de la paleta y un shader toon propio. La protagonista es una cápsula con cabeza esférica, rodete y anteojos hechos con primitivas.
2. **Paquetes gratuitos:** reemplazar escenario y obstáculos por modelos low poly con licencia CC0 (por ejemplo los kits de ciudad de Kenney o los personajes de Quaternius), recoloreados con la paleta. Verificar la licencia de cada paquete y registrarla en `art/LICENSES.md`.
3. **Arte final:** modelos encargados a un artista 3D según las referencias, en formato glTF (`.glb`) con animaciones de correr, saltar, deslizarse, chocar, celebrar y sorpresa.

Cada modelo se carga a través de una escena contenedora (`Player.tscn`, `obstacles/*.tscn`) para que cambiar el arte sea reemplazar un archivo.

### Shader toon

Crear `art/shaders/toon.gdshader`: iluminación por bandas (2 o 3 cortes), color de sombra cálido en lugar de gris, borde por inversión de normales en un segundo pase para el contorno, y un parámetro `rim_light` suave. Todos los materiales del juego usan este shader.

## 4. Pantallas y flujo

La v1 tiene diez pantallas; el recorrido feliz va de la selección de idioma a los resultados en menos de cinco minutos. Toda la interfaz usa botones grandes en la mitad inferior, al alcance del pulgar.

### Flujo

1. **LanguageSelect** (solo la primera vez): dos ventanas con la vecina; carteles "Español" y "English"; botón "¡Listo! / Done". Preselecciona el idioma del sistema.
2. **AgeGate** (solo la primera vez): "¿En qué año naciste?" con selector. Define el modo de anuncios y si el jugador podría participar en premios en el futuro (sección 9).
3. **Home:** fachada de la casa con la vecina en la ventana, logo, botón fucsia "¡A averiguar!", botones "Mapa", "Álbum" (deshabilitado con candado en v1) y "Ropero" (deshabilitado), engranaje de Opciones.
4. **Map:** maqueta del barrio con el alfiler del caso 3 activo y el resto bajo niebla.
5. **Briefing:** tarjeta de cartón con el rumor, el objetivo, dos objetivos opcionales, tiempo límite y botón "¡Vamos!".
6. **AdOfferCard** (sobre el Briefing, antes de correr): oferta de x2 monedas (sección 7).
7. **Run:** carrera con HUD; se interrumpe para el **TruckPuzzle** y vuelve.
8. **Delivery:** pizarrón de corcho con las pistas juntadas, medidor de certeza y tres versiones para contar.
9. **Results:** revelación, desenlace, estrellas, monedas con el multiplicador aplicado.
10. **Defeat:** cara de espanto, "Reintentar gratis" siempre visible, "Seguir corriendo" con anuncio una vez por caso, "Volver al mapa".

**Settings**, accesible desde Home y desde la pausa: idioma, volúmenes separados, vibración, efectos reducidos, tamaño de texto, modo zurdo, términos y privacidad, número de compilación.

### HUD de la carrera

| Posición | Elemento |
| --- | --- |
| Arriba izquierda | Tres mates como vidas |
| Arriba centro | Reloj con el tiempo restante; naranja en los últimos 10 s |
| Arriba derecha | Tres casilleros de pistas y contador de monedas (con insignia "x2" si el multiplicador está activo) |
| Debajo del reloj | Tira de mensajes cortos que se va sola a los 2 s |
| Abajo derecha | Botón de pausa (la habilidad de la protagonista llega en la v2) |

## 5. Mecánica del runner, caso 3 y minijuego

El runner avanza solo por tres carriles y se arma con tramos de calle diseñados a mano que se encadenan según el guion del caso, no al azar, para que las pistas siempre estén donde la historia las necesita. Todos los valores son iniciales y viven en `balance.json`.

### Valores de partida

```json
{
  "lanes_x": [-2.0, 0.0, 2.0],
  "lane_change_time": 0.12,
  "speed_start": 8.0,
  "speed_max": 14.0,
  "speed_gain_per_10s": 0.5,
  "jump_height": 1.6,
  "jump_time": 0.55,
  "slide_time": 0.6,
  "lives": 3,
  "minor_hit_time_penalty": 2.0,
  "invulnerable_after_hit": 2.0,
  "swipe_min_px": 40,
  "swipe_angle_tolerance_deg": 30,
  "coin_value": 1,
  "clue_bonus_coins": 10,
  "start_ad_coin_multiplier": 2,
  "segment_length_m": 30
}
```

### Controles

- Deslizar izquierda o derecha: cambiar de carril (con búfer de un gesto).
- Deslizar arriba: saltar. Deslizar abajo: deslizarse; en el aire, cae rápido.
- Tolerancia de ángulo de 30° y distancia mínima de 40 px.
- En escritorio, flechas y WASD para pruebas automáticas.

### Obstáculos de la v1

| Obstáculo | Cómo se esquiva | Penalización |
| --- | --- | --- |
| Bolsa de basura | Saltar o cambiar de carril | Menor: −2 s |
| Maceta | Saltar | Menor: −2 s |
| Balde | Saltar | Menor: −2 s |
| Sábana tendida | Deslizarse | Menor: −2 s |
| Cajas de mudanza | Cambiar de carril | Mayor: −1 vida |
| Puerta que se abre | Cambiar de carril (aviso de 1 s) | Mayor: −1 vida |
| Auto estacionado | Cambiar de carril | Mayor: −1 vida |
| Perro con pelota (peligro especial) | Cruza dos carriles con aviso de ladrido | Mayor: −1 vida |

### Datos del caso 3

```json
{
  "id": "case_03",
  "title_key": "CASE03_TITLE",
  "rumor_key": "CASE03_RUMOR",
  "protagonist": "marta",
  "receiver": "rosa",
  "zone": "block",
  "time_of_day": "siesta",
  "time_limit_s": 120,
  "segments": ["intro", "easy_a", "clue_boxes", "easy_b", "witness_hose", "truck_block", "medium_a", "arrival_plaza"],
  "clues": [
    {"id": "box_office", "segment": "clue_boxes", "type": "object", "text_key": "CASE03_CLUE_BOX", "core": true},
    {"id": "wardrobe_fight", "segment": "witness_hose", "type": "witness", "text_key": "CASE03_CLUE_WARDROBE", "core": true},
    {"id": "truck_booking", "source": "minigame", "type": "document", "text_key": "CASE03_CLUE_TRUCK", "core": true}
  ],
  "minigame": {"id": "truck_03", "segment": "truck_block"},
  "optional_goals": ["talk_to_driver", "max_2_hits"],
  "endings": [
    {"id": "full", "requires": ["box_office", "wardrobe_fight", "truck_booking"], "text_key": "CASE03_END_FULL"},
    {"id": "alt", "choice": true, "text_key": "CASE03_END_ALT"},
    {"id": "partial", "fallback": true, "text_key": "CASE03_END_PARTIAL"}
  ],
  "rewards": {"coins": 50, "unlock": "rosa"}
}
```

En la entrega, la versión correcta solo se puede elegir si se juntaron las tres pistas centrales; si falta alguna, aparece con candado y el texto "Te faltó averiguar algo".

### Minijuego del camión de flete

- Grilla de 6 x 6 vista desde arriba. El camión ocupa 2 casillas horizontales en la fila 3 y debe salir por la derecha.
- Los bloques se arrastran solo en su eje; no se superponen.
- Límite de 25 segundos; el reloj de la carrera queda en pausa.
- Éxito: el camión sale con bocina y el fletero da la pista `truck_booking`.
- Fallo por tiempo: el fletero mueve el camión solo, se pierden 10 s de carrera y la pista no se entrega.
- Incluir un validador que resuelva el nivel por búsqueda en anchura para confirmar que tiene solución, y guardar la cantidad mínima de movimientos.

```json
{
  "id": "truck_03",
  "size": 6,
  "exit_row": 2,
  "blocks": [
    {"id": "truck", "x": 0, "y": 2, "len": 2, "dir": "h", "target": true},
    {"id": "car_a", "x": 2, "y": 1, "len": 2, "dir": "v"},
    {"id": "car_b", "x": 4, "y": 2, "len": 3, "dir": "v"},
    {"id": "cart", "x": 3, "y": 0, "len": 2, "dir": "h"}
  ]
}
```

## 6. Idiomas

El juego sale en español e inglés desde la v1, con la elección en la primera apertura y un cambio en caliente desde Opciones. Se usa el sistema de traducciones nativo de Godot con un CSV.

### Implementación

- `data/i18n/strings.csv` con columnas `keys,es,en`, importado como traducción de Godot.
- `Locale.gd` guarda el idioma elegido en el archivo de guardado y llama a `TranslationServer.set_locale()`.
- Toda etiqueta de interfaz usa `tr("KEY")`; al cambiar de idioma se emite la señal `locale_changed` y las pantallas abiertas se refrescan.
- La marca "La Chusma" no se traduce; el subtítulo sí.
- Formato de hora: 24 h en español, 12 h con AM y PM en inglés (afecta pistas con horarios).
- Botones y globos con 30% de espacio extra para textos más largos.

### Claves mínimas de la v1

```csv
keys,es,en
LANG_TITLE,Elegí tu idioma,Choose your language
APP_SUBTITLE,secretos de barrio,Neighborhood Secrets
BTN_PLAY,¡A averiguar!,Let's snoop!
BTN_MAP,Mapa,Map
BTN_ALBUM,Álbum,Album
BTN_CLOSET,Ropero,Closet
BTN_GO,¡Vamos!,Let's go!
BTN_RETRY_FREE,Reintentar gratis,Try again for free
BTN_BACK_MAP,Volver al mapa,Back to map
BTN_TELL,¡Contalo!,Spill it!
AD_X2_TITLE,¿Duplicás tus monedas?,Double your coins?
AD_X2_BODY,Mirá un video y las monedas de esta corrida valen x2,Watch a video and this run's coins count x2
AD_WATCH,Ver video,Watch video
AD_NO_THANKS,"No, gracias",No thanks
AD_UNAVAILABLE,"Ahora no hay videos, probá más tarde","No videos right now, try later"
AD_CONTINUE,Seguir corriendo,Keep running
DEFEAT_TITLE,¡No puede ser!,No way!
DEFEAT_BODY,¡Y justo cuando estaba por enterarme de todo!,Just when I was about to find out everything!
MISSING_CLUE,Te faltó averiguar algo,You missed a clue
CASE03_TITLE,La vecina que se muda,The Neighbor Who's Moving
CASE03_RUMOR,¡Dicen que Graciela se va porque se peleó con todo el mundo!,They say Graciela is leaving because she fought with everyone!
CASE03_CLUE_BOX,Una caja dice FRÁGIL – OFICINA NUEVA,A box says FRAGILE – NEW OFFICE
CASE03_CLUE_WARDROBE,Gritaban por un ropero que no entraba,They were yelling about a wardrobe that wouldn't fit
CASE03_CLUE_TRUCK,"El flete es el sábado a Córdoba, cerca de la hermana","The truck leaves Saturday for Córdoba, near her sister"
CASE03_END_FULL,Se muda por trabajo y para estar cerca de su hermana,She's moving for work and to be near her sister
CASE03_END_ALT,"¡Se peleó con todos, yo te dije!","She fought with everyone, I told you!"
CASE03_END_PARTIAL,"Algo raro hay, pero no sé bien qué","Something's up, but I'm not sure what"
```

Los textos en inglés son una primera versión; antes del lanzamiento los revisa una persona nativa para adaptar el humor.

## 7. Publicidad: solo anuncios rewarded

El juego usa exclusivamente anuncios rewarded: videos que el jugador elige ver a cambio de una recompensa dentro del juego, anunciada antes de tocar el botón. No hay banners ni intersticiales. En la v1 los anuncios son simulados; la red real (AdMob) se conecta en una fase posterior.

### Ubicaciones

| Ubicación | Momento | Recompensa | Tope |
| --- | --- | --- | --- |
| `run_start_x2` | En el Briefing, antes de correr | Las monedas juntadas en esa corrida valen x2 | 1 por corrida, 10 por día |
| `defeat_continue` | Pantalla de derrota | Seguir con una vida desde el último punto de control | 1 por caso |
| `minigame_hint` | Dentro del minijuego | Resalta el próximo movimiento correcto | 1 por minijuego |

Todos los topes son configurables en `balance.json`.

### La oferta de x2 al inicio

1. Al tocar "¡Vamos!" en el Briefing, sube la tarjeta `AdOfferCard` con un televisor animado, el título "¿Duplicás tus monedas?" y dos botones del mismo tamaño: "Ver video" y "No, gracias".
2. Si el jugador mira el video completo y la red confirma la recompensa, `GameState.run_coin_multiplier = 2` y el HUD muestra la insignia dorada "x2" junto al contador.
3. Si cierra el video antes de tiempo o el anuncio falla, la carrera arranca normal con x1 y un mensaje amable; nunca se bloquea el juego.
4. Si no hay anuncio disponible, el botón se ve gris con "Ahora no hay videos, probá más tarde".
5. El multiplicador se aplica al final, en la pantalla de resultados, y se reinicia al terminar o abandonar la corrida.
6. El multiplicador afecta solo a las monedas, no al puntaje de habilidad del ranking (sección 8).

### Interfaz `AdService`

```gdscript
# autoload/AdService.gd
extends Node
signal reward_earned(placement: String)
signal ad_closed(placement: String, rewarded: bool)
signal ad_failed(placement: String, reason: String)

func is_rewarded_ready(placement: String) -> bool: ...
func can_show(placement: String) -> bool: ...   # revisa topes diarios y por caso
func show_rewarded(placement: String) -> void: ...
```

- **`MockAdService` (v1):** muestra una pantalla de "video" falsa de 5 segundos con botón de cerrar; el panel de depuración permite simular éxito, cierre anticipado y falla.
- **`AdMobService` (fase posterior):** integra AdMob mediante un plugin para Godot 4 compatible con la versión del motor; usa los IDs de prueba oficiales durante el desarrollo y los reales solo en la versión de tienda.
- La recompensa se entrega solo con la señal `reward_earned`, nunca con el simple cierre del video.
- Registrar en un log local cada oferta mostrada, aceptada, completada y fallada, para medir después.

### Reglas que el código debe respetar

- Ver un anuncio siempre es voluntario; perder nunca obliga a ver uno.
- La recompensa es siempre algo del juego (monedas, una vida, una ayuda), nunca dinero.
- No mostrar anuncios durante la carrera, un diálogo o un minijuego en curso.
- Antes de cargar anuncios reales: pedir consentimiento con la herramienta de consentimiento de la red (necesaria para usuarios de Europa) y marcar como contenido infantil los anuncios para menores de 13 según el control de edad (sección 9).

## 8. Monedas, ranking y premios en dinero

La v1 no reparte dinero: las monedas son solo del juego y el programa de premios queda construido pero apagado (`PRIZES_ENABLED = false`) hasta tener aprobación legal y de las tiendas. Repartir dinero real entre jugadores convierte la app en un concurso con premios, algo que las tiendas y las leyes regulan aparte.

### Por qué se separa del anuncio de x2

- **AdMob** permite que los anuncios rewarded den objetos del juego, como monedas o una vida extra, pero sus políticas prohíben compensar a los usuarios por ver anuncios fuera de ese esquema. Si mirar un video aumenta la chance de cobrar dinero, la recompensa deja de ser un objeto del juego.
- **Google Play** trata las apps con premios en efectivo como concursos de dinero real: exigen una solicitud aprobada, clasificación solo para adultos y controles para que no participen menores.
- **Apple** pide reglas oficiales dentro de la app, aclarar que Apple no patrocina el concurso y clasificación 17+, y puede tratar estos rankings con premios como juego con dinero real.
- **Azar o habilidad:** si las monedas duplicadas suben la probabilidad de ganar, el premio depende en parte de algo distinto a la habilidad, y eso lo acerca a una lotería, que está mucho más regulada en Argentina y en EE. UU.

Por eso el diseño separa dos números: las **monedas** (se pueden duplicar con el anuncio y sirven para cosméticos) y el **puntaje de habilidad** (decide el ranking y nunca lo toca un anuncio).

### Modelo propuesto para una fase posterior: torneo semanal de habilidad

| Parámetro | Valor de partida | Configurable |
| --- | --- | --- |
| Período | Semana de lunes a domingo | Sí |
| Pozo | Un porcentaje de la ganancia neta de anuncios del período | Sí, `prize_pool_share` (por ejemplo 0.30 para el pozo y 0.70 para el desarrollador) |
| Ranking | Puntaje de habilidad calculado en el servidor | Fórmula en `balance.json` |
| Ganadores | Los mejores N puntajes elegibles | Sí, `prize_winners` |
| Participación | Gratis para todo el que juega, sin compra ni anuncio obligatorio | No |
| Elegibilidad | 18 años o más, país o estado habilitado, identidad verificada, reglas aceptadas | Lista de países en configuración |
| Pago | Proveedor de pagos externo, con los datos fiscales que corresponda | Pendiente |

El porcentaje lo define el dueño del proyecto; la regla importante es publicarlo en las reglas oficiales y respetarlo. La ganancia por usuario de los anuncios suele ser baja, así que el pozo crece solo con mucha audiencia.

### Puntaje de habilidad

```text
skill_score = casos_completos * 1000
            + pistas_centrales * 150
            + segundos_restantes * 5
            - choques_mayores * 100
# no incluye monedas ni ningún efecto de anuncios
```

### Qué construye Claude Code en la v1

- `PrizeService.gd` con la interfaz completa (`is_enabled()`, `is_player_eligible()`, `submit_score()`, `get_leaderboard()`) que devuelve "no disponible" mientras el flag esté apagado.
- Cálculo y guardado local del `skill_score` de cada corrida, listo para enviarse a un servidor más adelante.
- Ninguna pantalla, texto, ícono ni descripción de tienda que mencione dinero, premios en efectivo o "ganar plata" mientras el flag esté apagado.

### Qué se puede decir cuando se active

Solo después de la aprobación: "Torneo semanal: los mejores puntajes de habilidad comparten un pozo de premios. Ver reglas oficiales." Evitar promesas como "ganá plata jugando" o montos garantizados, porque las tiendas los consideran engañosos.

## 9. Guardado, privacidad y edad

El progreso se guarda solo en el teléfono, el juego no pide permisos sensibles y un control de edad en la primera apertura decide qué anuncios se pueden mostrar y quién podría entrar a un torneo en el futuro.

### Guardado

- Archivo `user://save.json` escrito por `SaveService.gd`, con copia `save.bak` antes de cada escritura para evitar corrupción.
- Se guarda: idioma, año de nacimiento, opciones, monedas, casos completados, pistas encontradas, desenlaces vistos, `skill_score` por corrida y contadores diarios de anuncios.
- Guardado automático al terminar un caso, un minijuego y al pasar la app a segundo plano.
- Campo `save_version` para migrar el formato en versiones futuras.

### Control de edad

| Edad declarada | Anuncios | Torneo futuro |
| --- | --- | --- |
| Menos de 13 | Anuncios marcados como contenido infantil, sin personalización | No |
| 13 a 17 | Anuncios sin personalización | No |
| 18 o más | Anuncios según el consentimiento del jugador | Sí, si se cumplen los demás requisitos |

La pregunta es neutral (año de nacimiento, sin sugerir una respuesta) y no se puede cambiar libremente después; si el jugador quiere corregirla, se reinicia el control.

### Privacidad

- Sin permisos de contactos, ubicación, cámara ni micrófono.
- Sin cuentas en la v1. Cuando llegue el torneo, la verificación de identidad la hace un proveedor externo y el juego guarda solo un identificador.
- Pantalla de política de privacidad y términos accesible desde Opciones, en ambos idiomas.
- Completar la sección de seguridad de datos de Google Play declarando el uso del SDK de anuncios.

## 10. Plan por fases

Claude Code trabaja en siete fases, una por vez, y no pasa a la siguiente hasta que se cumplen los criterios de aceptación de la actual. Al terminar cada fase, hace un commit y resume qué funciona.

| Fase | Qué se construye | Criterio de aceptación |
| --- | --- | --- |
| 1. Base | Proyecto Godot, carpetas, autoloads vacíos, `balance.json`, exportación Android y workflow de GitHub Actions | GitHub Actions compila el APK, lo publica en `build-latest` y al instalarlo en el teléfono muestra el logo y el número de compilación |
| 2. Runner gris | Tres carriles, controles táctiles y de teclado, cámara, tramos con primitivas, obstáculos, monedas, vidas, cronómetro, HUD | Se puede correr 120 s, esquivar los 8 obstáculos, perder vidas y ver la derrota |
| 3. Caso 3 | Máquina de estados del caso, carga de `case_03.json`, pistas, testimonio, entrega con tres versiones, resultados | El caso se completa con los tres desenlaces según las pistas y la elección |
| 4. Minijuego | `TruckPuzzle` con grilla, arrastre, tiempo límite, validador por búsqueda en anchura | El nivel `truck_03` se resuelve, entrega la pista y el fallo aplica la penalización |
| 5. Pantallas e idiomas | LanguageSelect, AgeGate, Home, Map, Briefing, Defeat, Settings; CSV de traducciones; cambio de idioma en caliente | Todo el recorrido funciona en español y en inglés, sin textos sueltos en el código |
| 6. Anuncios simulados y guardado | `AdService` con `MockAdService`, las tres ubicaciones rewarded, x2 de monedas, topes; `SaveService`; `PrizeService` apagado | El x2 duplica las monedas solo si el video simulado se completa; el progreso sobrevive a cerrar la app |
| 7. Estética | Shader toon, paleta, niebla, iluminación de siesta, protagonista de primitivas con rodete y anteojos, efectos de pista y moneda | Las capturas se parecen en color y tono a las referencias de `art/reference/` |

### Fuera del alcance de la v1

- Anuncios reales y conexión con AdMob.
- Torneo, servidor, cuentas y pagos.
- Los otros 14 casos, el álbum, el ropero y las habilidades de las protagonistas.
- Arte final encargado; la v1 usa primitivas o paquetes CC0.
- iOS.

### Siguiente versión (v2)

Conectar AdMob con IDs de prueba y consentimiento, sumar a Rosa como segunda protagonista con cambio de protagonista, el álbum de secretos, dos casos más y el primer arte final de la protagonista.

## 11. Compilar y probar sin computadora

El dueño del proyecto trabaja solo desde el celular, así que todo se compila en la nube: cada cambio subido a GitHub genera automáticamente un APK que se descarga e instala en el teléfono. Claude Code no ve el juego funcionando; el dueño es quien prueba y le cuenta qué ve.

### Flujo de trabajo

1. Claude Code escribe el código en el repositorio de GitHub y sube los cambios a la rama `main`.
2. GitHub Actions compila el proyecto con Godot en modo sin pantalla y genera el APK.
3. El APK se publica en una versión de GitHub llamada `build-latest`, que se reemplaza en cada compilación.
4. El dueño abre esa versión desde el navegador o la app de GitHub en su Android, descarga el APK y lo instala.
5. Prueba el juego y le manda a Claude Code capturas, grabaciones de pantalla o una descripción de lo que falla.

### Qué tiene que crear Claude Code

- `.github/workflows/android.yml`, que se ejecuta con cada push a `main` y también a mano desde la pestaña Actions.
- Usar una acción o imagen de Docker mantenida para exportar Godot 4 a Android (por ejemplo `dulvui/godot-android-export` o la imagen `barichello/godot-ci`), fijada en la misma versión de Godot que el proyecto.
- Pasos del workflow: descargar el código, importar el proyecto en modo sin pantalla, correr las pruebas, exportar el APK de depuración, subirlo a la versión `build-latest` y también como artefacto.
- `export_presets.cfg` incluido en el repositorio, sin contraseñas adentro.
- Una clave de firma de depuración fija guardada en el repositorio (solo para pruebas), para que cada APK nuevo se instale encima del anterior sin desinstalar. La clave de firma para publicar en la tienda va más adelante como secreto del repositorio, nunca en el código.
- Pruebas automáticas que corran en el workflow: carga de todos los JSON, validador del minijuego, traducciones completas en ambos idiomas y lógica del multiplicador x2.
- Si la compilación falla, Claude Code revisa el registro de GitHub Actions y lo corrige antes de dar la fase por terminada.

### Ayudas para probar desde el teléfono

- Número de compilación y fecha visibles en la pantalla de Opciones, para confirmar que se instaló la última versión.
- Panel de depuración oculto (se abre tocando cinco veces el logo) con fps, estado actual del caso, botones para saltar a cada pantalla y para simular éxito o falla de anuncios.
- Botón en el panel para borrar el guardado y empezar de cero.

### Notas para el dueño

- Para instalar el APK, Android pide permitir la instalación de apps de origen desconocido para el navegador o la app de GitHub; se habilita una sola vez.
- Los repositorios privados tienen una cuota mensual gratuita de minutos de GitHub Actions; si se agota, se puede hacer el repositorio público o esperar al mes siguiente.
- Para publicar en Google Play más adelante hace falta una cuenta de desarrollador de Google Play, que se gestiona desde el navegador del celular.

## 12. Antes de activar premios en dinero

El programa de premios se enciende solo cuando todos estos puntos están resueltos. Esta lista no reemplaza el consejo de un abogado especializado en juegos y concursos, que conviene consultar en Argentina y en EE. UU.

- [ ] Opinión legal sobre si el torneo es un concurso de habilidad y en qué países y estados se puede ofrecer.
- [ ] Solicitud aprobada en Google Play para concursos con premios en dinero real y clasificación de edad correspondiente.
- [ ] Revisión de Apple con reglas oficiales dentro de la app, aclaración de que Apple no patrocina y clasificación 17+.
- [ ] Confirmación de que ningún anuncio afecta el ranking ni la elegibilidad.
- [ ] Reglas oficiales publicadas en ambos idiomas: porcentaje del pozo, fórmula del puntaje, período, cantidad de ganadores, elegibilidad, forma de pago y aclaración de que la participación es gratuita.
- [ ] Servidor que valide puntajes y detecte trampas antes de pagar.
- [ ] Proveedor de verificación de identidad y de pagos, con tratamiento fiscal resuelto.
- [ ] Geobloqueo de los lugares no habilitados y bloqueo total para menores de 18.

### Fuentes

- [Políticas del programa AdMob y AdSense](https://support.google.com/admob/answer/48182) — prohíbe compensar a usuarios por ver anuncios fuera del inventario rewarded.
- [Resumen de los anuncios rewarded de AdMob](https://support.google.com/admob/answer/7372450?hl=en-GB) — las recompensas son objetos dentro de la app, como vidas o moneda del juego.
- [Política de Google Play sobre juegos, apuestas y concursos con dinero real](https://support.google.com/googleplay/android-developer/answer/9877032)
- [Requisitos de contests en Google Play (Klein Moynihan Turco)](https://kleinmoynihan.com/?p=14462) — proceso de solicitud y clasificación solo adultos.
- [Rechazo por guía 5.3.2 de Apple (foro de desarrolladores)](https://developer.apple.com/forums/thread/671722) — reglas oficiales en la app, Apple no patrocina, 17+.
- [App con premios clasificada como juego con dinero real, guía 5.3.4 (foro de Apple)](https://developer.apple.com/forums/thread/825845)
