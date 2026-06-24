# CLAUDE.md

## Qué es este proyecto
Scrollytelling para un trabajo de facultad sobre restaurantes con estrellas
Michelin en el mundo (dataset: 3.842 restaurantes, CSV en este repo).
Es una sola página HTML que combina varias "escenas" activadas por scroll,
con estética de menú de restaurante (entrada / plato principal / postre)
y simbología visual de Michelin (libro rojo, estrellas).

Archivo principal actual: `carta-completa.html` (~5200 líneas, todo inline).

## Stack
- HTML + CSS + JS vanilla (sin build step, sin framework)
- D3.js para mapas/geografía
- Canvas 2D para la visualización de puntos tipo "constelación"
- Google Fonts: Playfair Display (títulos), Inter (texto)
- Paleta: negro carbón #222222, rojo Michelin #BD2333, dorado #F7B036

## Estructura de escenas conocidas (por id, ya existentes en el archivo)
- `#book-scene` — apertura tipo libro/guía Michelin con SVGs generativos
- `#mi-stage` — mapa de puntos tipo constelación (canvas), namespace `mi_`
- `#ti-stage` — timeline de expansión de la guía por país/año, namespace `ti_`
- `#stage` — mapa coroplético por país (D3 + GeoJSON), sin namespace claro
- `#bookoutro` / `#cm-tickets-section` — cierre, namespace `bo`/`cm_`

## Problemas conocidos (ver diagnóstico para detalle)
- GeoJSON mundial completo embebido inline en un `<script>` (~370KB)
- 10 bloques `<script>` y 4 bloques `<style>` sin consolidar
- Namespaces inconsistentes por escena (`mi_`, `ti_`, `bo`, `cm_`, sin prefijo)
- SVGs decorativos grandes mezclados con la estructura HTML

## Reglas para trabajar en este repo
- NO tocar la lógica visual/funcional de ninguna escena al reorganizar archivos.
  Reorganizar primero, verificar que se ve igual, recién después tocar diseño.
- NO instalar dependencias ni frameworks nuevos sin preguntar (sigue siendo
  HTML/CSS/JS vanilla, sin build step).
- Conservar la paleta de colores y tipografías existentes a menos que se
  pida explícitamente cambiarlas.
- Cuando extraigas datos (GeoJSON, arrays de eventos, etc.) a archivos
  separados, cargalos con `fetch()` relativo — no asumas un servidor con
  rutas especiales.
- Después de cada cambio estructural, decime cómo verificarlo en el navegador
  (qué abrir, qué mirar, qué no debería haber cambiado visualmente).
- Si algo requiere romper la convención anterior (ej: un namespace nuevo),
  preguntame antes de aplicarlo a todo el archivo.

## Comandos útiles
- No hay build step. Para previsualizar: abrir el HTML directo en el navegador,
  o servir la carpeta con `python3 -m http.server 8000` si hay fetch() a
  archivos locales (necesario una vez que se externalice el GeoJSON).
