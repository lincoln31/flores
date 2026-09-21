# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proyecto

Página web de una sola pieza, `index.html` ("Flores para ti"): animación romántica en español. Un girasol se seca, cae al suelo, brota un árbol cuyas ramas y girasoles dibujan un corazón, y aparece una carta con un contador de días/horas desde una fecha. No hay build, dependencias, gestor de paquetes, linter ni tests; HTML, CSS y JS (vanilla, en una IIFE) viven en el mismo archivo. Solo se carga externamente la tipografía de Google Fonts (Figtree, Instrument Serif).

## Desarrollo

- Abrir `index.html` directamente en el navegador (o con cualquier servidor estático). Recargar para reiniciar la animación (el botón "Ver de nuevo" hace `location.reload()`).
- Personalización: objeto `CONFIG` al inicio del `<script>` (`fecha`, `mensaje`, `leyenda`). La fecha también se puede sobrescribir con `?fecha=AAAA-MM-DD` (o `AAAA-MM-DDTHH:MM`) en la URL.

## Arquitectura

**Guion secuencial** (`start()`, al final del script): tras el clic en el girasol, encadena con `await` las fases: barra de carga → `wilt()` → `fallHero()` → `landing()` → `growTree()` → `startSway()` → `bloomAll()` → `story()`. Añadir o reordenar fases se hace en esa función.

**Estado de escena por CSS**: `#app[data-scene]` (`intro` → `loading` → `grow` → `story`) controla transiciones de cielo (`.sky--a/.sky--b`); `#ground.up`, `#treeWrap.shift`, `#letter.on`, `#legend.on`, `#replay.on` activan el resto. La geometría se ajusta con variables `:root` (`--gh`, `--th`, `--shift`) y un `@media (max-aspect-ratio:4/5)` para pantallas verticales.

**Generación procedural determinista** (semillas fijas con `mulberry32`, por lo que el árbol es idéntico en cada carga):
- `genNodes`/`buildTree`: ramas por "colonización del espacio" (space colonization) con puntos atractores dentro del corazón paramétrico (`heartPt`, `OUTLINE`, `inside`). Los segmentos se guardan en `segs` y `growTree()` los revela con un bucle `requestAnimationFrame` según distancia desde la raíz.
- `buildFlowers`: girasoles pequeños sobre el contorno del corazón y relleno por mayor hueco; reutilizan el `<symbol>`-like `#sf` (`buildSymbol`) vía `<use>`, con paleta por variables CSS (`--p1`, `--p2`, `--d`).
- `buildBigFlower`: girasol grande de la portada; los pétalos son `.p` con variables `--c`, `--dry`, `--rot`, `--sy` que la clase `.wilt` usa para secarlos.

**Animación**: mezcla de Web Animations API (`el.animate`) para caídas, chispas, viento y floraciones, y transiciones CSS. Las tres piezas del viento: `gust()` (inclina `#lean` y desprende flores con `detach()`), `startSway()` (balanceo continuo en `#sway`) y `windLoop()`. Las flores desprendidas se recrean en `#fallen` y vuelven a florecer con `bloom()`.

**Tiempo y accesibilidad**: todas las duraciones pasan por `D()`/`sleep()`, que aplican `TS` (0.35 con `prefers-reduced-motion`); el viento y el balanceo se omiten en ese modo. Mantener este patrón al añadir animaciones nuevas.

**SVG**: se construye con el helper `el(name, attrs, parent)` (namespace SVG). `#tree` usa `viewBox="0 0 600 720"` con la base en `BASE = {x:300, y:690}`; `#groundPt` marca el punto de aterrizaje que usa `fallHero()`.
