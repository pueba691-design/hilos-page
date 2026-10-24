# Catálogo de prendas

Las prendas viven en un solo arreglo, `PRENDAS`, dentro de `index.html` (buscar `/* ---------- Datos (contenido de ejemplo, editable) ---------- */`). **No hay base de datos ni CMS** — para ~40 productos esto es suficiente; no hace falta nada más sofisticado (ver conversación sobre cuándo pasar a un framework: solo si el catálogo crece a cientos de prendas o se necesitan cuentas de usuario/inventario real).

Estado actual (2026-10-24): **30 de ~40 prendas ya son reales** (con foto propia), las otras 5 (Cárdigan Cafetal, Gorro Páramo, Bolso Plaza, Bufanda Pacífico, Top Guayacán — las originales de ejemplo) siguen con ilustración de muestra hasta que se reemplacen.

## Estructura de cada prenda

```js
{
  id:'canguro-margarita',          // único, sin espacios — se usa en el DOM, localStorage, filtros
  cat:'accesorios',                 // 'abrigos' | 'accesorios' | 'tops' — debe coincidir con los chips de categoría
  nombre:'Canguro Margarita',
  precio:89990,                     // en pesos, sin decimales
  color:'Terracota',                // color que se ve por defecto en la tarjeta — debe estar en `colores` (si lo hay) y existir en HILOS
  colores:['Terracota','Mostaza','Blanco'], // opcional — los colores EN LOS QUE REALMENTE VIENE esta prenda (ver abajo)
  tallas:['Única'],                 // el filtro de talla se arma solo a partir de esto, no hay que tocarlo aparte
  etiqueta:'Nuevo',                 // opcional — badge en la tarjeta; se puede omitir
  desc:'...',                       // descripción corta, debajo del nombre en la tarjeta
  cuento:'...',                     // el texto de "¿Sabías que...?" en la ficha de producto
  fotos:['assets/productos/canguro-margarita-1.jpg'], // opcional — ver sección de fotos
  escena:'cafetal',                 // SOLO en las 5 prendas de ejemplo — ilustración de inspiración (ver abajo)
  pie:'Cafetales de ladera',        // pie de foto de la inspiración, solo junto con `escena`
  horas:'Unas 10 horas...',         // tiempo de tejido, texto libre — ESTIMADO salvo que el cliente lo confirme
  material:'Algodón mercerizado...' // cuidado de la prenda, texto libre
}
```

## Colores: el campo `colores` es por prenda, no global

`HILOS` (justo antes de `PRENDAS`) es la paleta completa del sitio: `{n:'Tierra', c:'#A15A3C'}`. Pero **cada prenda solo viene en algunos de esos colores** — por eso existe `colores: [...]` en el objeto de la prenda, con los nombres exactos (deben existir en `HILOS`) de las variantes que esa prenda realmente tiene. La función `coloresDe(p)` es la que filtra: si la prenda no trae `colores`, cae de vuelta a los primeros 5 de `HILOS` (así siguen funcionando las 5 prendas de ejemplo sin tocarlas).

**Esto se usa en dos lugares — y hay que mantenerlos sincronizados a mano**: los puntos de color de la tarjeta (`.puntos`) y las hebras de la ficha (`#p-hebras`) ambos llaman a `coloresDe(act)`/`coloresDe(p)`, así que basta con definir `colores` una vez por prenda.

Si una prenda necesita un color que no existe todavía en `HILOS`, agrégalo ahí primero (con su `n` y su `c` en hex) antes de referenciarlo en `colores`.

## Fotos: el campo `fotos` (arreglo, no una sola)

Cuando una prenda tiene fotos reales, usa `fotos:['assets/productos/slug-1.jpg', 'assets/productos/slug-2.jpg', ...]`. Con eso:

- La tarjeta de la colección y la ficha de producto muestran la **primera** foto (`fotos[0]`) en vez de la ilustración generada (`prendaSVG`).
- Si hay **más de una foto**, aparecen puntos de navegación tipo carrusel — arriba a la derecha en la tarjeta, abajo a la izquierda en la ficha (junto al botón de zoom "Ver los puntos", sin chocar) — para pasar entre ellas. No hace falta nada más: basta con poner 2+ rutas en `fotos` y los puntos aparecen solos (`.galeria-puntos`/`.p-galeria` en el JS).
- Si una prenda **no** tiene `fotos`, sigue cayendo en la ilustración de muestra (`prendaSVG`) — así las 5 prendas de ejemplo no se rompen.

**Convención de nombres**: `assets/productos/<slug-del-producto>-1.jpg`, `-2.jpg`, etc. El slug no tiene que ser igual al `id`, pero conviene que sí para no perderse.

### Cómo se preparan las fotos (receta, no está automatizado)

No hay ningún paso de build que optimice imágenes — se hace a mano con Python/Pillow antes de subirlas:
1. `ImageOps.exif_transpose()` para corregir la orientación (fotos de celular vienen rotadas según el sensor).
2. Recorte de bordes negros si los hay (detectar filas/columnas casi negras en los bordes y cortarlas) y, si hace falta, un recorte manual puntual (sombras muy oscuras, objetos de fondo que distraen, reflejos).
3. Redimensionar al lado más largo a ~1200-1400px (más que eso no se nota en pantalla y pesa de más).
4. Guardar como JPEG `quality=78-82, optimize=True, progressive=True`. Si un archivo queda por encima de ~200KB (fotos con mucha textura/detalle, como tejidos de punto grueso), bajar a `quality≈74` y el lado largo a ~1150px — se nota poco y pesa bastante menos.

Meta aproximada: la mayoría de fotos entre 50-200KB. Con 30+ productos esto sí importa para que el sitio seteando ligero.

## El campo `escena` es legado, no lo uses en prendas nuevas

`escena` (`cafetal`, `paramo`, `plaza`, `pacifico`, `guayacan`) + `pie` son exclusivos de las 5 prendas de ejemplo originales: generan una ilustración de "inspiración" falsa (función `escenaSVG`) para la sección "¿Sabías que...?" de la ficha, con el pie "Ilustración de muestra". **Las prendas reales no deben usar este campo** — si no se define `escena`, esa sección de imagen simplemente no se muestra (se oculta `#p-insp`) y solo queda el texto de `cuento`, que es lo correcto mientras no haya una foto real del lugar/flor que inspiró la prenda. Si en el futuro se consigue esa foto, se podría agregar un campo nuevo tipo `fotoInsp` — no existe todavía, hay que construirlo.

## Agregar una prenda nueva

1. Fotos optimizadas en `assets/productos/` (ver receta arriba).
2. Un objeto nuevo en `PRENDAS` con `fotos`, `colores` (si la prenda no viene en los 5 colores por defecto, que es casi siempre), y el resto de campos.
3. Nada más — la tarjeta, los filtros de categoría y talla, la ficha, el mensaje de WhatsApp y la sección Historias se generan solos a partir de esos datos.

**Ojo con nombres repetidos**: si dos prendas reales terminan con el mismo nombre comercial (pasó con "Llavero Sunflower" y "Aretes Margarita" en este catálogo — el proveedor de fotos a veces reusa el mismo nombre para variantes), hay que diferenciarlos en el campo `nombre` (p. ej. "Llavero Margarita" vs. "Llavero Sunflower", "Aretes Margarita" vs. "Aretes Margarita Cascada") para que no se vean como un error de catálogo duplicado.
