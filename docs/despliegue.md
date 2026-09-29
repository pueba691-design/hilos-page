# Despliegue

## El flujo

```
editar index.html / assets / etc.
        ↓
git add -A && git commit -m "..." && git push
        ↓
GitHub: pueba691-design/hilos-page (rama main)
        ↓
Cloudflare Pages redespliega solo (1-2 min)
        ↓
https://hilosnata.com  (y https://hilos-page.pages.dev, el dominio gratis original)
```

No hay paso manual de "publicar" — cualquier `git push` a `main` dispara un despliegue nuevo en Cloudflare Pages.

## Cloudflare Pages, no Workers

Cloudflare unificó su dashboard bajo "Workers & Pages", y el flujo por defecto para conectar un repo ahora asume un `Worker` (con Wrangler, necesita `wrangler.toml`). Este proyecto usa el flujo **clásico de Pages** (sin build, sin Wrangler): si hay que tocar la configuración de nuevo en el dashboard, hay un link **"Continue to Pages"** (legacy workflow) al fondo de la pantalla de creación — sin eso, el asistente por defecto lleva al camino equivocado.

Configuración del proyecto en Cloudflare:
- **Framework preset**: `None`
- **Build command**: (vacío)
- **Build output directory**: `/` (raíz — ahí está `index.html`)
- **Production branch**: `main`

## Dominio

`hilosnata.com` se compró y se gestiona directo en Cloudflare (Cloudflare Registrar), en la misma cuenta que el proyecto de Pages — por eso conectar el dominio fue automático (Cloudflare ya tenía el DNS). Sigue existiendo también el dominio gratis `hilos-page.pages.dev`, apunta al mismo sitio.

## Cuentas de GitHub

El repo `hilos-page` es de la cuenta **pueba691-design**. Las credenciales de git guardadas en la máquina de desarrollo son de otra cuenta del mismo dueño, **gestionsigasoftware-netizen**, agregada como colaboradora con permiso de escritura sobre el repo. Si algún día el `git push` falla con un 403 de permisos, es probablemente esto — revisar los colaboradores del repo antes de asumir que hay que cambiar credenciales en la máquina.

## Caché (`_headers`)

Ver [`pwa.md`](pwa.md#caché-y-actualizaciones) — resumen: los archivos en `/assets/*` y `manifest.json` usan `no-cache` (siempre revalida) a propósito, porque sus nombres de archivo no cambian cuando se edita una imagen. Si en algún momento se empieza a usar nombres de archivo con hash (`logo.abc123.png`), ahí sí tendría sentido volver a un `max-age` largo.

## Pendiente de infraestructura

- **Wompi** (pagos en línea, Etapa 2 del plan — ver `CLAUDE.md`): cuando el cliente abra su cuenta Wompi, hay que crear una carpeta `/functions` en la raíz (Cloudflare Pages Functions) para calcular la firma de integridad del lado del servidor. La clave secreta de Wompi va como variable de entorno en el dashboard de Cloudflare, nunca en el código ni en el repo.
