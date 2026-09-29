# Arquitectura

Todo el sitio vive en **un solo archivo, `index.html`**: HTML, `<style>` inline en el `<head>` y `<script>` inline al final del `<body>`. No hay build, no hay `npm install`, no hay framework — se edita el archivo y ya. La única dependencia externa es **Three.js r128**, cargado desde cdnjs para el ovillo 3D.

## CSS: variables primero

Todo el sistema de color y tipografía está en variables CSS al inicio del `<style>` (`:root{...}`):

```css
--blanco --crudo --lino --arena --caramelo --tierra --cafe --negro   /* paleta cruda del logo */
--bg --bg-2 --card --ink --ink-2 --head --accent                    /* roles semánticos */
--btn-bg --btn-ink --btn-ink-h --btn-line                           /* SIEMPRE negro, ver regla abajo */
```

Hay un bloque `@media (prefers-color-scheme: dark)` y un `:root[data-theme="dark"]` que redefinen `--bg`, `--ink`, `--head`, etc. para modo oscuro. **Regla de la marca**: `--btn-bg` es negro en los dos modos — el negro es *solo* para botones, nunca para fondos de tarjetas o secciones. Si una sección necesita un fondo "elevado", se usa `var(--card)` (crema en claro, café oscuro en oscuro), no negro.

## Estructura del `<body>`

En orden: `#splash` (pantalla de bienvenida de la PWA, ver [`pwa.md`](pwa.md)) → encabezado con menú → `<main>` con las secciones (`portada`, `#coleccion`, `#historias`, `#proceso`, `#cuidado`, `#encargo`) → `<footer>` → diálogos nativos `<dialog>` (ficha de producto `#producto`, bolsa `#bolsa`) → toast `#tostada` → aviso de instalar `#pwa-aviso` → botón `#ir-arriba`.

## JavaScript: un IIFE grande + un IIFE aparte para el ovillo

Casi todo el script está dentro de un único `(function(){ 'use strict'; ... })()`, con `$`/`$$` como atajos de `querySelector`/`querySelectorAll` definidos al inicio. Los bloques principales, en el orden en que aparecen:

| Sección (comentario en el código) | Qué hace |
|---|---|
| `Datos (contenido de ejemplo, editable)` | El arreglo `PRENDAS` — ver [`catalogo.md`](catalogo.md) |
| `Ilustraciones` | Genera los SVG de muestra de cada prenda (`prendaSVG`) — se reemplaza por fotos reales cuando lleguen |
| `Cinta` | La cinta animada de frases que se repite arriba de la colección |
| `Bolsa` | Carrito en `localStorage` (`hilos-bolsa`) y armado del mensaje de WhatsApp |
| `Colección` | Pinta las tarjetas, y los filtros por categoría + talla (sticky bar) |
| `Ficha de producto` | El `<dialog>` de detalle: color, talla, cantidad, zoom, "¿Sabías que…?". Incluye el **borrador de selección** (`sessionStorage`, clave `hilos-borrador`) que restaura la prenda/color/talla/cantidad si el navegador recarga la pestaña a medio elegir (p. ej. al volver de WhatsApp) |
| `Historias con scroll` | El scroll narrativo fijo que pasa por cada prenda |
| `Revelado, cabecera y detalles` | Animaciones de aparición al hacer scroll, y comportamiento del header |
| `Ovillo 3D realista con hilo físico` | Todo Three.js: la bola de hilo, la física de la cuerda suelta, la interacción con el mouse |
| `Instalar como app` | El aviso `#pwa-aviso` — ver [`pwa.md`](pwa.md) |
| `Volver arriba` | El botón `#ir-arriba` con el anillo de progreso |

Al final, fuera del IIFE grande: el registro del service worker.

### El ovillo 3D, en corto

Es una malla generada por código (no un modelo `.glb`), tejida a partir de cientos de tubos curvos (`THREE.TubeGeometry`) enrollados alrededor de una esfera, más una "pelusa" de líneas sueltas y dos agujas hechas con `LatheGeometry`. El hilo suelto es una cuerda con física simple (Verlet) que choca con la esfera y el piso, y sigue al mouse **solo dentro de un radio** alrededor del ovillo (`RADIO_ACCION`, en el bloque de "Interacción") — fuera de ese radio no lo sigue, para no sentirse invasivo. La densidad de la malla se reduce en móvil (`movil = !finoPuntero`) para rendimiento.

**Para probar cambios en esta escena con Playwright**: el canvas WebGL repinta constantemente, lo que cuelga `page.click()`/`page.screenshot({fullPage:true})` normales. Lanza Chromium con `args:['--use-gl=swiftshader','--enable-webgl','--ignore-gpu-blocklist','--disable-gpu-sandbox']`, usa `fullPage:false`, y dispara clics con `page.evaluate(() => el.click())` en vez de `page.click()`.

## Colección: cómo funcionan los filtros

Los chips de categoría son fijos en el HTML (Todo/Abrigos/Accesorios/Tops). Los chips de **talla** se generan en JS a partir de `PRENDAS` (`[...new Set(PRENDAS.flatMap(p=>p.tallas))]`), así que **no hay que tocar código** al agregar prendas con tallas nuevas — aparecen solas. Los dos filtros se combinan con AND (`aplicarFiltros()`). La barra completa (`.filtros-barra`) es `position:sticky` bajo el header.
