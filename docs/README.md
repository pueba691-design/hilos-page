# Documentación técnica de Hilos

Esta carpeta documenta **cómo está construido el sitio y cómo trabajar con él**. Para el contexto de marca, identidad visual y decisiones de negocio (colores, tipografías, carrito, Wompi), lee primero [`CLAUDE.md`](../CLAUDE.md) en la raíz del proyecto — esa es la fuente de verdad para todo lo de negocio/diseño y no se repite aquí.

## Índice

- [`arquitectura.md`](arquitectura.md) — cómo está organizado `index.html`: secciones, sistema de colores, el ovillo 3D, la colección y sus filtros, la bolsa, la ficha de producto.
- [`pwa.md`](pwa.md) — manifest, service worker, ícono, pantalla de bienvenida, y las particularidades de Android/iOS que hay que conocer.
- [`despliegue.md`](despliegue.md) — el flujo GitHub → Cloudflare Pages, el dominio, la caché, y cómo publicar un cambio.
- [`catalogo.md`](catalogo.md) — la estructura de datos de las prendas (`PRENDAS`) y cómo agregar o editar una.

## Lo esencial en una frase

Es un sitio estático de una sola página (`index.html`, sin build ni framework): todo el CSS y el JavaScript viven dentro de ese archivo. Se edita directamente, se sube a GitHub (`pueba691-design/hilos-page`) y Cloudflare Pages lo despliega solo en cada `git push` a `main`.
