# Comparación de escenas de scroll

Comparación entre la escena del **plato de las estrellas** (`#mi-stage`, namespace `mi_`)
y la escena de **estrellas verdes / postre** (`#postres`, namespaces `pc_`, `pi_`, `pRev`).

## Estructura: plato de las estrellas (`#mi-stage`)

HTML estático en el archivo, más un spacer que el propio script inserta antes de renderizar:

```
<div id="mi-pad" style="height:460vh"></div>   ← insertado por JS (mi_), no está en el HTML fuente
<div id="mi-root">
  <div id="mi-grain"></div>                     ← velo de grano, opacidad atada a p
  <div id="mi-stage">                           ← position:fixed;inset:0 (el "escenario")
    <canvas id="mi-plate"></canvas>             ← se redibuja entero en cada frame donde cambia p
    <div class="cap" id="mi-cap"></div>         ← texto/caption, HTML inyectado por mi_captionHTML(p)
    <div class="hint" id="mi-hint"></div>       ← "scrollear" cue, se apaga apenas arranca el scroll
    <div class="panel" id="mi-cover">           ← portada ("El plato de las estrellas")
      <div class="box">…</div>
    </div>
    <div class="panel" id="mi-outro">           ← cierre ("La cima es una franja angosta")
      <div class="box">…<div class="obar">…</div></div>
    </div>
  </div>
</div>
```

Puntos clave de esta estructura:
- Todo el contenido visual real (los puntos/"migas") **no existe como DOM**: se genera y
  dibuja en el `<canvas>` desde un array `mi_dots` (posición, forma, categoría de estrella)
  creado una sola vez con un PRNG semillado.
- `#mi-cover` y `#mi-outro` son los únicos textos "de verdad" en el HTML; `#mi-cap` se llena
  dinámicamente según el tramo de `p` en el que está el scroll.
- El único spacer de toda la escena es `#mi-pad`; no hay spacers intermedios por sub-tramo.

## Estructura: estrellas verdes / postre (`#postres`)

```
<div id="footpin">                    ← (escena anterior: cierre de "las cocinas del prestigio")
  <div class="foot" data-stage="7">   ← position:sticky; se desvanece cuando entra #postres
    …
  </div>
</div>                                ← su ::after agrega 100vh de spacer para el pin

<section id="postres">
  <div class="pcard">
    <div class="pcover-spacer"></div>          ← spacer de 100vh, reserva scroll para el cartel
    <div class="pcover">                       ← position:fixed;inset:0; display:none hasta entrar en rango
      <div class="cbox">
        <div class="course-tag">Postre</div>
        <h1 class="pcover-title">La estrella que no se sirve en el plato</h1>
        <p class="pcover-sub">…</p>
      </div>
    </div>
    <div class="pbody">                        ← contenido normal en flujo, NO pineado
      <div class="menu-head reveal">…</div>     ← "Tres estrellas & verdes"
      <div class="menu">
        <div class="course reveal">…</div>      ← un bloque por país (Francia, España, EE.UU. …)
        <div class="course reveal">…</div>
        …
      </div>
      <div class="pfoot reveal">…</div>         ← cierre: "El postre perfecto no deja huella."
    </div>
  </div>
</section>
```

Puntos clave de esta estructura:
- Es DOM real y completo: cada país, cada plato, cada nombre está escrito en el HTML.
  No hay canvas ni generación procedural.
- Tiene **dos spacers distintos** con roles distintos: el de `#footpin` (transición
  de entrada desde la escena anterior) y `.pcover-spacer` (el cartel de portada del postre).
- `.pbody` no está pineado ni tiene spacer propio: fluye normal, y sus hijos `.reveal`
  simplemente esperan a entrar en viewport.

## Comparación

| | 🍽️ Plato de las estrellas (`mi_`) | 🍃 Estrellas verdes (`pc_`/`pi_`/`pRev`) |
|---|---|---|
| **Cómo se fija en pantalla** | `#mi-stage` es `position:fixed;inset:0` todo el tiempo que está activo | Mezcla: `.foot` usa `position:sticky`, `.pcover` usa `position:fixed` (evitaron sticky ahí a propósito porque no andaba confiable) |
| **Cómo reserva el espacio de scroll** | Un solo spacer gigante creado por JS: `#mi-pad` de `460vh` | Dos spacers chicos: el `::after` de `#footpin` (`100vh`) y `.pcover-spacer` (`100vh`) — cada tramo de transición tiene el suyo |
| **Qué maneja el progreso** | Una sola variable continua `p` (0→1) que se recalcula en *cada* frame con `requestAnimationFrame`, sin importar si algo cambió | Igual usa rAF + `scrollY`, pero calcula **dos progresos independientes** (`ta` para foot→postres, `p` para el cover), cada uno con su propio rango chico |
| **Qué se re-renderiza con el scroll** | El canvas entero se **redibuja a mano** (`mi_draw`) — recalcula gradientes, posiciones y colores de cada miga en cada cambio de `p` | Nunca redibuja nada: solo pisa `style.opacity` / `style.transform` de elementos DOM ya existentes |
| **Tecnología de render** | Canvas 2D generativo (formas random con PRNG semillado) | HTML/CSS puro con `transition` |
| **Granularidad del efecto** | Continuo y fino: decenas de "ventanas" de aparición (`mi_smooth(a,b,p)`) por cada punto/texto, como una animación de video scrubbeable | Grueso: nada más que un fundido + deslizamiento en 2-3 tramos; el menú de países ni siquiera es continuo |
| **El menú de países (`.course`)** | — (no aplica) | Ni sigue `p` ni usa rAF: es un `IntersectionObserver` con `threshold:.16` que agrega la clase `.in` **una sola vez** por elemento y lo deja de observar. Es "aparece y listo", no scroll-linked |
| **Coordinación con la escena vecina** | Chequea en vivo si `.scroller` (copas) ya empezó a aparecer (`mi_wineAlreadyIn`) para auto-desactivarse y no pisarse | El propio `pi_` script mide la distancia entre `#footpin` y `#postres` para armar el crossfade — coordinación explícita por posición, no por estado de opacidad de la otra escena |
| **Costo/optimización** | Evita redibujar el canvas si `p` no cambió (`mi_lastDrawn`) | No hay nada costoso que evitar: solo dos números por frame y un par de `style.opacity` |

## La diferencia de fondo

El plato es una *simulación continua*: un solo reloj (`p`) maneja un dibujo procedural
completo, como un video controlado por el scroll.

El postre es *coreografía discreta*: varios efectos cortos e independientes (dos
crossfades cortos + un reveal-on-enter) que se disparan por tramos, sin necesidad de
redibujar nada porque todo el contenido ya es DOM estático.
