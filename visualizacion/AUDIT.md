# AUDIT — visualizacion/index.html

Auditoría de relevamiento puro. **No se aplicó ningún fix en este pase.** Todos los
rangos de línea están verificados contra el archivo real (leído con `Read`/`Grep`,
no de memoria de conversaciones previas).

## 0. Identidad del archivo auditado

```
sha256(visualizacion/index.html)              = a973a8dc611d28ace6f7c75a438523d726deee34b2523cdb664a782ef5e93639
sha256(respuesta HTTP de localhost:8123/index.html) = a973a8dc611d28ace6f7c75a438523d726deee34b2523cdb664a782ef5e93639
diff (archivo local vs. respuesta HTTP)        = sin diferencias (bit a bit)
líneas totales                                 = 2159
```

El archivo que audité abajo es exactamente el que sirve `localhost:8123` en este momento.

Estructura general del `<head>`/`<body>`:
- Un único bloque `<style>`: líneas 7–496.
- Cinco bloques `<script>` inline + 1 `<script src="d3.min.js">`:
  - Script 1: 958–1436
  - Script 2: 1437–1529
  - `<script src="d3.min.js">`: 1530
  - Script 3: 1531–1735
  - Script 4: 1736–2077
  - Script 5: 2086–2159

---

## 1. Inventario de escenas

Ordenadas por **orden real de scroll** (no por orden de declaración en el HTML — varias
escenas insertan su propio "pad" de scroll con `insertBefore` en tiempo de ejecución, lo
que reordena el flujo real respecto del HTML estático). Se indica al final de cada fila
si el orden HTML difiere del orden de scroll.

| # | Escena | Selector raíz | Mecanismo de pin | Script (líneas) | CSS (líneas, aprox.) |
|---|---|---|---|---|---|
| 1 | Apertura libro | `#bookintro` | `position:fixed` (siempre presente) + rAF manual leyendo `scrollY` contra spacer propio `#bookpad` (130vh, insertado por JS antes de `document.body.firstChild`) | 1739–1819 | 263–301 (+ `.bi-*`, `.book-*`, `.bib-man` dispersos en ese rango) |
| 2 | Mapa mundial (coroplético D3) | `#graphic` | `position:fixed` + rAF manual (`frame()`, dentro del IIFE async) leyendo `scrollY` contra `#mscroller` (6×`.mstep`, insertado por JS) + `#mapgap` (30vh) | 1531–1734 (`frame()` en ~1680–1734) | 153–158 (+ `#map`, `#cover`, `#panel`, `#closing`, `#legend`, `#nav` dispersos 159–244) |
| 3 | Expansión de la guía (timeline) | `#ti-root` | `position:fixed` (`#ti-stage`, `#ti-grain`) + rAF manual (`expan_loop`) contra spacer propio `#ti-pad` (620vh, insertado antes de `#ti-root`) | 1176–1353 (`expan_loop` en 1322–1352) | 346–379 |
| 4 | El plato de las estrellas (constelación) | `#mi-root` | `position:fixed` (`#mi-stage`, `#mi-grain`) + rAF manual (`mi_loop`) contra spacer propio `#mi-pad` (460vh, insertado antes de `#mi-root`) | 959–1174 (`mi_loop` en 1134–1173) | 302–343 |
| 5 | Estrellas & precio (tickets) | `#cm-tickets-stage` | `position:fixed` + rAF manual (`render()`/`measure()`/`setActive()`) contra spacer dedicado `#cm-tickets-scroll` (250vh, estático en el HTML) | 2003–2076 (datos de las tarjetas en 1969–2002, sin scroll) | 127–473 (rango ancho: comparte prefijo `.cm-*` con la escena #10 del juego/calculadora) |
| 6 | Las cocinas del prestigio (copas/shelf) | `#stage` / `#shelfwrap` / `#shelf` / `.scroller .intro/.step` | **Híbrido**: `position:fixed` (`#stage`, `#scrim`, `#matte`, `#frame`, `.legend`) + rAF continuo (`frame()`, mismo IIFE que la escena #2, comparten script) para opacity/zoom, **más** `IntersectionObserver` (`io`, threshold 0.55) sobre `[data-stage]` para el cambio discreto de foco entre copas (`applyStage`) | rAF: 1531–1734 (compartido con #2) · IO: 1355–1435 (`io` en 1429, `applyStage` en 1414–1427) | 11–13 (`#stage`), 14–15 (`#frame`), 16 (`#matte`), 20 (`#scrim`), 21–41 (`#shelfwrap`/`#shelf`/`.glass`), 63–75 (`.legend`) |
| 7 | Pasos de texto de "cocinas" | `.scroller .step[data-stage]`, `.card` | Sin pin propio — son marcadores de flujo normal, consumidos por el `IntersectionObserver` y por la matemática de `frame()` de la escena #6 | mismo que #6 | 42–45 (`.step`), 46 (`.card`), 48–60 (`.intro`) |
| 8 | Cierre de las cocinas | `#footpin` / `.foot` | `position:sticky` (`.foot{position:sticky;top:0}`) + rAF manual (`pi_frame`) que solo maneja el **fade de opacity** de `.foot`/`#postres` — el pin en sí lo hace CSS nativo, no JS | 1444–1479 | 71 (`.foot`), 80 (`#footpin`) |
| 9 | Postre — cartel intro ("La estrella que no se sirve en el plato") | `#postres .pcover` | `position:fixed` + rAF manual (`pc_render`/`pc_measure`/`pc_setActive`) contra spacer dedicado `.pcover-spacer` (100vh) — **migrado desde `position:sticky` en este mismo hilo de trabajo** | 1485–1528 | 130–138 |
| 10 | Postre — menú ("Tres estrellas & verdes") | `#postres .pbody` | Sin pin propio — contenido en flujo normal que queda expuesto cuando `#postres .pcover` se desactiva; el fade-in de sus `.reveal` internos usa `IntersectionObserver` (`pRev`) | IO `pRev`: 1432–1433 (dentro del IIFE de la escena #6, línea 1355–1435) · reorganización de acordeón de países: 1822–1868 (sin scroll) | 81–152 (`#postres` completo, incluye #9) |
| 11 | El juego — precio & bolsillo (calculadora de salario) | `#cm-game-section` | **Ninguno** — sección estática en flujo normal, puramente interactiva (`input`/`select`), sin listener de scroll ni rAF | cálculo de conversión: 1870–1967 (sin scroll) | 388–424 |
| 12 | Cierre (círculo rojo + muñeco Michelin) | `#bookoutro` | `position:fixed` + rAF manual (script 5) contra spacers estáticos `#bo-spacer` (100vh) + un `<div>` anónimo adicional (100vh) | 2090–2158 | 477–495 |

Notas sobre el orden real de scroll (por inserción dinámica de spacers):
`bookpad(130vh) → mscroller(6×.mstep) → mapgap(30vh) → #ti-pad(620vh) → #mi-pad(460vh) → #cm-tickets-scroll(250vh) → .scroller (intro/steps + footpin + #postres) → #cm-game-section → #bo-spacer + div(100vh)`.
El orden de **declaración en el HTML** es distinto: `#bookintro, #graphic, #ti-root, #mi-root, #cm-tickets-stage, #stage/#shelf, .legend, .scroller(...)`, porque las escenas 1–4 son overlays `position:fixed` que no ocupan espacio de flujo — su "lugar" real en el scroll lo define el spacer que cada una inserta por JS, no su posición en el HTML estático.

---

## 2. Tabla comparativa de mecanismos de pin

| Mecanismo | Escenas | Cantidad |
|---|---|---|
| `position:fixed` + rAF manual (spacer dedicado) | bookintro, mapa (#graphic), timeline (#ti-root), constelación (#mi-root), tickets (#cm-tickets-stage), **postres/pcover (#postres .pcover, post-migración)**, bookoutro | 7 |
| `position:fixed` + rAF manual + `IntersectionObserver` (híbrido) | cocinas/copas (#stage/#shelf) | 1 |
| `position:sticky` + rAF manual (solo opacity) | footpin/.foot | 1 |
| `IntersectionObserver` puro (sin rAF, sin pin propio) | reveal de `#postres` (`pRev`) — no pinea nada, solo dispara una clase `.in` | (auxiliar, no es una escena con pin) |
| Sin mecanismo de pin (contenido estático) | `#cm-game-section` (calculadora) | 1 |

**Por qué `#postres .pcover` fue la única que tuvo que migrar de mecanismo, y no fue "esta vez sí funcionó":**

Antes de esta migración, `position:sticky` se usaba en exactamente **dos** lugares de
todo el archivo: `.foot` (escena 8) y `#postres .pcover` (escena 9) — ambas agregadas
en el mismo diff sin commitear (confirmado en la sesión anterior vía `git diff` contra
`HEAD`). Ninguna otra escena del archivo — ni bookintro, ni el mapa, ni la timeline,
ni la constelación, ni los tickets, ni el bookoutro — usa `position:sticky` en ningún
punto: **todas fueron construidas desde el origen con el patrón `position:fixed` +
spacer + rAF manual**. Es decir, `#postres` y `.foot` no son "una escena más que
también pinea distinto"; son las **únicas dos** que se apartan del idioma establecido
en el resto del archivo, y lo hacen porque se escribieron después, con un mecanismo
distinto al que ya predominaba.

Con la migración de esta sesión, `#postres .pcover` pasó a usar el mecanismo
mayoritario (fixed+rAF, igual que 6 de las otras 7 escenas pineadas). **`.foot`
queda ahora como el único punto de todo el archivo que sigue dependiendo de
`position:sticky`** — es decir, el desbalance no desapareció, se invirtió: antes
había dos escenas "distintas al resto" (sticky), ahora hay una sola (`.foot`).

No encontré nada estructural (ancestro compartido, handler global, orden de scripts)
que differencie a `#postres` de `.foot` en cuanto a por qué una "funcionaba" y la otra
no — ambas están en la misma cadena de ancestros (`.scroller`), ambas se miden con el
mismo patrón `absTop()`, y la auditoría de ancestros de la sesión anterior no encontró
overflow/transform/filter/contain/will-change en ningún punto de la cadena de ninguna
de las dos. La única diferencia estructural real entre ambas es el mecanismo de pin
en sí (sticky vs. fixed) — que es justamente la variable que se cambió.

---

## 3. Código muerto / huérfano relacionado a los intentos anteriores sobre `#postres`

Búsqueda dirigida de restos de las versiones previas (`.pcover-wrap`, `pi_wrap`,
`pi_cover`, `pi_bStart`, `pi_bRange`, `margin-top:-100vh` fuera de lugar):

| Patrón buscado | Resultado |
|---|---|
| `.pcover-wrap` (clase CSS o referencia JS) | **0 ocurrencias** — completamente eliminada en la migración |
| `pi_wrap`, `pi_cover`, `pi_bStart`, `pi_bRange` (variables JS de la versión sticky de `.pcover`) | **0 ocurrencias** |
| `margin-top:-100vh` | **1 ocurrencia**, línea 81 (`#postres{...margin-top:-100vh...}`) — es la de la escena 8→9 (footpin→postres), **sigue en uso**, no es un resto huérfano |
| Comentario que menciona el mecanismo viejo | Línea 1498, comentario explicativo dentro del script nuevo (`pc_measure`) que referencia por qué el rango es `spacer.offsetHeight` completo y no `spacerHeight - innerHeight` — es documentación del fix, no código muerto |
| Reglas CSS duplicadas para `.pcover` (una sticky + una fixed conviviendo) | No encontradas — solo existe la regla `fixed` actual (línea 131) |
| Variables `pc_*` sin usar | No encontradas — `pc_start`, `pc_range`, `pc_active`, `pc_last`, `pc_clamp`, `pc_smooth`, `pc_absTop`, `pc_measure`, `pc_setActive`, `pc_render` están todas referenciadas |

**Conclusión de esta sección**: no quedan restos de los intentos anteriores sobre
`#postres` específicamente. La migración a fixed+rAF dejó el árbol de código limpio
en lo que respecta a esa escena puntual.

No relevé código muerto **fuera** de `#postres` (footer de calculadora, mapa, etc.)
porque quedó fuera del alcance pedido ("restos de los intentos anteriores sobre
`#postres`") — si querés ese barrido más amplio, lo puedo hacer en un pase aparte.

---

## 4. Listeners de scroll/wheel y loops de `requestAnimationFrame`

**No hay ningún `addEventListener('scroll', ...)` ni `addEventListener('wheel', ...)`
en todo el archivo.** Las 9 escenas con pin leen `window.scrollY` **polleando dentro de
un loop de `requestAnimationFrame`**, no mediante listeners de evento — es el idioma
consistente de toda la página.

Los 8 puntos donde se lee `window.scrollY` (línea del `var/const sy = window.scrollY...`):

| Línea | Loop | Escena | Rango activo (definido por su propio spacer) |
|---|---|---|---|
| 1136 | `mi_loop` | Constelación (#mi-root) | `mi_start` (= `offsetTop` de `#mi-pad`) hasta `mi_start + mi_padH(460vh) + mi_OUT`, con margen `mi_FADE` en el borde de entrada |
| 1324 | `expan_loop` | Timeline (#ti-root) | `expan_start` (= `offsetTop` de `#ti-pad` + `EXPAN_LEAD`) hasta `expan_start + expan_padH(≈620vh) + expan_FADE` |
| 1462 | `pi_frame` | Footpin→postres (escena 8→9, fase A) | `pi_aStart` (`offsetTop` de `#footpin`) hasta `pi_aStart + pi_aRange` (`pi_aRange` = distancia hasta el `offsetTop` de `#postres`) |
| 1509 | `pc_render` | Postres `.pcover` (escena 9, fase B) | `pc_start` (`offsetTop` de `.pcover-spacer`) hasta `pc_start + pc_range` (`pc_range` = alto del spacer, 100vh) |
| 1682 | `frame` | Mapa (#graphic) + cocinas/copas (#stage) | Desde `0`/`window.__bookPad` (arranca el mapa) hasta `_footTop` (offsetTop de `.foot` dentro de `#footpin`) — cubre TODO el tramo del mapa + `.scroller` (intro/steps) |
| 1772 | `loop` (bookintro) | Apertura del libro (#bookintro) | `0` hasta `window.__bookPad` (altura de `#bookpad`, 130vh) |
| 2034 | `render` | Tickets (#cm-tickets-stage) | `start` (`offsetTop` de `#cm-tickets-scroll`) hasta `start + range` (`range = scrollHeight(250vh) - innerHeight`), con `outExt`/`outFade` extendiendo la cola |
| 2129 | `loop` (bookoutro) | Cierre (#bookoutro) | rango definido contra `#bo-spacer` (no relevado en detalle en este pase — ver script 5, 2090–2158) |

### Superposición de rangos (dos handlers activos al mismo tiempo)

Encontré **acoplamiento explícito y deliberado** entre tres pares de loops, mediante
lectura cruzada del estado (`style.opacity`/`style.display`) de elementos de la OTRA
escena — no son rangos completamente aislados:

1. **`expan_loop` (timeline) ↔ `mi_loop` (constelación)** — línea 1326:
   `var miAlreadyIn = miEl && miEl.style.display==='block' && parseFloat(miEl.style.opacity||'0') > 0.05;`
   `expan_loop` se desactiva (`inRange=false`) si la escena de constelación ya está
   activa, y a la inversa `mi_loop` (línea 1140) chequea `mi_wineAlreadyIn` sobre el
   scroller de "cocinas". Esto es un **handoff intencional** para la frontera entre
   dos escenas consecutivas (ver orden real de scroll en la sección 1) — está
   diseñado para que no compitan, pero **sí corren ambos loops en paralelo todo el
   tiempo** (los dos `requestAnimationFrame` están siempre vivos, solo que uno se
   auto-inhibe con un `return` temprano cuando detecta que el otro ya tomó control).

2. **`frame()` (mapa/cocinas) lee flags globales escritos por `mi_loop`/`bookintro`**:
   línea 1686 (`window.__bookPad`, escrito por bookintro en 1752) y no hay lectura
   directa de mi_/expan_ desde `frame()` — pero sí lee `window.__postresIn` (escrito
   por el propio script de `frame()` vía un `IntersectionObserver` en 1733) y
   `window.__legendWantedByStage` (escrito por el `applyStage` de la escena 6, dentro
   del mismo IIFE). Estos son globales de `window`, no locals — cualquier otro script
   futuro que declare las mismas variables las pisaría sin aviso.

3. **`pi_frame` (footpin→postres) y `pc_render` (postres `.pcover`) corren en paralelo
   sobre el mismo `#postres`**: `pi_frame` está siempre vivo (su `requestAnimationFrame`
   nunca se detiene) y solo dejó de tocar `pi_cover`/`.pcover` tras la migración de esta
   sesión — ahora escribe únicamente `pi_foot.style.opacity` y `pi_postres.style.opacity`
   (el `#postres` completo). `pc_render` escribe `pc_cover.style.opacity`/`.transform`
   sobre `.pcover`, hijo de `#postres`. **No compiten por la misma propiedad del mismo
   elemento** (uno toca `#postres` como sección completa, el otro toca `.pcover` adentro),
   pero ambos modifican `opacity` en una cadena de ancestro-descendiente al mismo tiempo
   durante el solape de sus rangos — el `opacity` efectivo en pantalla de `.pcover` es el
   producto de ambos (opacity de `#postres` × opacity de `.pcover`), lo cual es el
   comportamiento buscado (crossfade compuesto) pero vale dejarlo explícito porque no es
   obvio leyendo un solo script sin el otro.

No encontré superposición de rango entre `pc_render` (postres) y ningún otro loop más
allá de `pi_frame` (ambos comparten el mismo elemento raíz `#postres`, ya cubierto en el
punto 3). Los rangos de bookintro/mapa/timeline/constelación/tickets/bookoutro son
secuenciales por diseño (cada uno con su propio spacer dedicado) salvo los handoffs
explícitos ya descriptos en los puntos 1 y 2.

---

## Alcance de este pase

Esto es únicamente mapa — no evalué si algo "funciona" ni prioricé nada para arreglar.
Quedan a tu criterio los próximos pasos (por ejemplo: si migrar también `.foot` para
que no quede como el único sticky remanente, o si dejarlo así porque nunca se reportó
como roto).
