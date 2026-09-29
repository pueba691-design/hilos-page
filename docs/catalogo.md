# Catálogo de prendas

Las prendas viven en un solo arreglo, `PRENDAS`, dentro de `index.html` (buscar `/* ---------- Datos (contenido de ejemplo, editable) ---------- */`). **No hay base de datos ni CMS** — para ~40 productos esto es suficiente; no hace falta nada más sofisticado (ver conversación sobre cuándo pasar a un framework: solo si el catálogo crece a cientos de prendas o se necesitan cuentas de usuario/inventario real).

## Estructura de cada prenda

```js
{
  id:'cardigan',              // único, sin espacios — se usa en URLs internas, localStorage, filtros
  cat:'abrigos',               // 'abrigos' | 'accesorios' | 'tops' — debe coincidir con los chips de categoría
  nombre:'Cárdigan Cafetal',
  precio:180000,                // en pesos, sin decimales
  color:'Tierra',               // color de hilo por defecto — debe existir en el arreglo HILOS
  tallas:['S','M','L'],         // el filtro de talla se arma solo a partir de esto, no hay que tocarlo aparte
  etiqueta:'Más pedido',        // opcional — badge tipo "Nuevo" en la tarjeta; se puede omitir
  desc:'...',                   // descripción corta, debajo del nombre en la tarjeta
  cuento:'...',                 // el texto de "¿Sabías que...?" en la ficha de producto
  escena:'cafetal',             // clave de la ilustración de inspiración (ver más abajo)
  pie:'Cafetales de ladera',    // pie de foto de la inspiración
  horas:'Unas 22 horas...',     // tiempo de tejido, texto libre
  material:'Algodón peinado...' // cuidado de la prenda, texto libre
}
```

## Colores de hilo (`HILOS`)

Arreglo aparte, justo antes de `PRENDAS`: `{n:'Tierra', c:'#A15A3C'}`. El campo `color` de cada prenda (y el `data-n` de los puntos de color en las tarjetas) hace referencia al `n` de este arreglo. Agregar un color nuevo aquí antes de usarlo en una prenda.

## Fotos: hoy son ilustraciones, mañana son las reales

Ahora mismo cada prenda se dibuja con SVG generado por código (función `prendaSVG`, sección "Ilustraciones") — es contenido de muestra. Cuando lleguen las fotos reales:

1. Súbelas a `assets/` (igual que se hizo con el logo).
2. Reemplaza la llamada a `prendaSVG(p, colorDe(p.color))` en la tarjeta y en la ficha de producto por un `<img>` apuntando al archivo.
3. Si hay varias fotos por prenda (recomendado — puesta, detalle, doblada), habría que agregar un campo tipo `fotos:['assets/cardigan-1.jpg', 'assets/cardigan-2.jpg']` y un pequeño carrusel/selector en la ficha — todavía no existe, es trabajo pendiente.

El campo `escena` (`cafetal`, `paramo`, `plaza`, `pacifico`, `guayacan`) también es una ilustración de muestra (función `escenaSVG`) para la foto de "inspiración" del ¿Sabías que...?" — mismo tratamiento cuando haya fotos/videos reales de cada lugar.

## Agregar una prenda nueva

Solo hay que agregar un objeto al arreglo `PRENDAS` con estos campos. Todo lo demás (tarjeta en la colección, filtros de categoría y talla, ficha de producto, mensaje de WhatsApp) se genera solo a partir de esos datos — no hay que tocar más código.
