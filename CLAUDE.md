# Hilos, tejidos en crochet: sitio web

Contexto del proyecto para Claude Code. Léelo antes de hacer cambios.

## Qué es
Página web de Hilos (@hilos.nata en Instagram), un emprendimiento colombiano de prendas tejidas a mano en crochet, bajo pedido.
El sitio es estático (un solo `index.html`, sin servidor ni base de datos) y se publica gratis en Cloudflare Pages.
Por ahora NO hay pagos en línea: la "bolsa" arma el pedido y lo envía por WhatsApp. Más adelante se cobrará con Wompi dentro de esa misma bolsa (ver "Carrito y pagos en línea").

## Pedido del cliente
- Que no sea una página tradicional: interactiva, sin sobrecargar, con un hilo conductor.
- Cada prenda con su "¿Sabías que…?": de qué lugar o cosa se inspiró, con foto o video de la inspiración.
- Que comprar se sienta como una experiencia y no como un gasto.
- Aspecto premium, como una tienda de marca hecha por desarrolladores senior.

## Identidad visual (respetar siempre)
- Colores sacados del logo: crudo `#FAF5F2`, caramelo `#C1825F`, tierra `#A15A3C`, café `#7A4430`, lino `#F3EBE4`. Fondo principal blanco `#FFFFFF`.
- Botones: fondo negro `#000000` con letra caramelo `#C1825F` (se eligió caramelo en vez de tierra porque sobre negro tiene mejor contraste).
- Tipografías: Josefin Sans (títulos, como el "HILOS" del logo), Hanken Grotesk (texto), Parisienne (solo acentos cursivos, como "Tejidos en Crochet" del logo).
- Tiene modo oscuro y debe verse bien en celular.

## Partes del sitio
- Portada con ovillo 3D realista (Three.js r128 desde cdnjs): hebras con textura de cabos torcidos, pelusa, agujas de madera, sombra en el piso. El hilo suelto es una cuerda 3D con física que choca con el ovillo y el piso, sigue al puntero y se puede agarrar.
- Colección con filtros, colores de hilo por tarjeta y ficha de producto (color, talla, cantidad, zoom a los puntos, "¿Sabías que…?" con la inspiración).
- Sección "Historias": scroll narrativo fijo que pasa por cada prenda y su inspiración.
- Cómo pedir (4 pasos), preguntas frecuentes, sección "A tu medida" y pie de página.
- Bolsa guardada en localStorage; al finalizar abre WhatsApp con el pedido y el subtotal.

## Pendiente
- Reemplazar el contenido de ejemplo: las 5 prendas (Cárdigan Cafetal, Gorro Páramo, Bolso Plaza, Bufanda Pacífico, Top Guayacán), precios, historias, tallas y tiempos son inventados.
- Poner fotos reales de las prendas (ideal PNG/WebP recortado) y de las inspiraciones; hoy son ilustraciones SVG de muestra.
- Número real de WhatsApp: constante `WHATSAPP` en el script (hoy `573000000000`).

## Carrito y pagos en línea (DECISIÓN TOMADA: nuestra bolsa + Wompi)
Decisión del cliente: el carrito será la bolsa propia que ya tiene el sitio (ícono de la cabecera) y los pagos en línea se harán con Wompi. No proponer otras plataformas de carrito ni cambiar de pasarela salvo que el cliente lo pida.

- Etapa 1 (ahora): lanzar con la bolsa actual enviando el pedido por WhatsApp. Costo cero. No quitar la bolsa ni el ícono de la cabecera.
- Etapa 2 (cuando el cliente abra su cuenta Wompi): agregar en la misma bolsa un botón "Pagar en línea" con Wompi, sin cambiar el diseño. Mantener también la opción de pedir por WhatsApp.
  - Wompi (Bancolombia): sin mensualidad, solo comisión por venta. Acepta PSE, tarjetas, Nequi, botón Bancolombia y efectivo. Requiere que el negocio tenga cuenta de ahorros o corriente en Bancolombia.
  - Integración: usar el Widget o Web Checkout de Wompi. La firma de integridad (hash con la clave secreta) se calcula en una función de Cloudflare Pages (`/functions`), nunca en el navegador. La clave secreta va como variable de entorno en Cloudflare, no en el código.
  - El total se debe recalcular en la función a partir de los productos y precios, no confiar en el total que envía el navegador.
  - Plan B solo si el cliente no puede abrir cuenta Bancolombia: Mercado Pago, con la misma bolsa y el mismo esquema de función en Cloudflare.
- Descartado: Snipcart (usa Stripe, que no opera para negocios en Colombia, y cobra mínimo mensual) y Shopify (mensualidad y habría que rehacer el diseño).
