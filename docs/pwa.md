# PWA: manifest, service worker, ícono y pantalla de bienvenida

El sitio es instalable como app en Android, escritorio (Chrome/Edge) e iOS (Safari, "Agregar a inicio").

## Archivos involucrados

- **`manifest.json`** — nombre, colores, y los íconos. `background_color` y `theme_color` son `#FFFFFF`: la app siempre arranca en blanco.
- **`sw.js`** — service worker. Estrategia *network-first*: intenta la red primero (así siempre se ve lo último si hay conexión) y solo cae al caché si falla la red (offline). Cachea `/`, `/index.html`, `/manifest.json` y los íconos en la instalación (`ASSETS`).
- **`assets/icon-192.png` / `icon-512.png`** — ícono de propósito `"any"`. También es el `apple-touch-icon`. **Fondo blanco opaco a propósito**, no transparente — iOS rellena de negro los íconos con transparencia (bug conocido de Safari), y un fondo blanco se funde con `background_color`.
- **`assets/icon-maskable-192.png` / `icon-maskable-512.png`** — ícono de propósito `"maskable"`, el que usan Android y la pantalla nativa de splash. Tiene el logo dentro de una "zona segura" (~68% del lienzo) porque el sistema operativo recorta estos íconos en distintas formas (círculo, squircle...). También con fondo blanco, para que no se vea como una caja de color sobre el fondo blanco de la app.

### Cómo se generaron los íconos

No hay un editor de imágenes en el flujo: se usa Python + Pillow para recortar el ovillo del logo original (bounding box del canal alfa) y comprimir con cuantización de paleta (`Image.quantize(colors=N, method=Image.FASTOCTREE)`), que para una ilustración plana como esta reduce el peso ~90% sin pérdida visible. Ver el historial de commits para los scripts exactos si hay que regenerar algo.

## La pantalla de bienvenida (`#splash`)

Es un `<div>` propio (primer elemento del `<body>`), **no** el splash nativo del sistema operativo — ver la sección de abajo para no confundirlos. Fondo blanco, logo grande con una animación sutil de "respiración", "HILOS" y el mensaje *"Un momento, estamos tejiendo esto para ti"*.

Se activa solo con `@media (display-mode: standalone)` en CSS — por eso nunca aparece al entrar por el navegador normal, sin ningún riesgo de parpadeo (`display:none` por defecto, sin depender de JS para ocultarlo). El JS correspondiente espera a `window.load` + un mínimo de tiempo (2.4s, ajustable en el bloque `Pantalla de inicio al abrir como app instalada`) antes de desvanecerlo, para que alcance a verse.

## ⚠️ La pantalla nativa de Android/iOS (no es nuestro código)

Antes de que `#splash` aparezca, **el sistema operativo** muestra su propia pantalla de splash generada automáticamente a partir del `manifest.json`: el ícono maskable + el campo `"name"` en texto plano, con la tipografía del sistema. **Esto no se puede quitar, saltar ni restylear** más allá de elegir qué ícono y qué `background_color` usa — es una limitación de la plataforma, no un bug del sitio.

Si alguna vez alguien reporta "veo una pantalla vieja/fea antes de la de bienvenida", casi seguro es esto, no un problema de caché. Antes de intentar diagnosticarlo como bug, pedir una captura de pantalla — el splash nativo y el `#splash` propio se confunden fácil por la descripción pero tienen causas y arreglos completamente distintos.

## Caché y actualizaciones

`_headers` (leído por Cloudflare Pages) define:

- `/assets/*` y `/manifest.json`: `Cache-Control: no-cache` — el navegador revalida en cada carga (rápido gracias a ETags de Cloudflare), así nunca queda una versión vieja de un ícono pegada. **No subir esto a un `max-age` largo mientras el logo/íconos sigan cambiando** — ya causó un bug real una vez (ver el commit "Corrige el parpadeo de vista vieja al abrir la PWA").
- `/index.html` y `/`: `Cache-Control: max-age=0, must-revalidate` — siempre se revisa si hay una versión nueva.
- `/sw.js`: `no-cache`.

Además, `index.html` registra el service worker y, si detecta que uno **nuevo** tomó el control (`controllerchange`), hace **un solo** `location.reload()` automático — así el usuario nunca queda a medio camino entre la versión vieja y la nueva de la app. Si se cambia qué archivos cachea `sw.js`, hay que subir el nombre de `CACHE` (`hilos-v2`, `hilos-v3`, …) para que el service worker viejo borre su caché anterior en el `activate`.

## Probar cambios de PWA

Los cambios de ícono/manifest/splash **no se ven en un dispositivo que ya tenía la app instalada** hasta limpiar los datos del sitio y reinstalarla (Android: ⓘ junto a la URL → Configuración del sitio → Borrar y restablecer; iOS: Ajustes → Safari → Borrar historial y datos). Avisar esto siempre que se pida probar un cambio de este tipo, para no perder tiempo pensando que el fix no funcionó.
