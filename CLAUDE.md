# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Qué es

Landing de tipo brochure para Hepta Chile (https://heptachile.cl), cuyo objetivo es invitar a comprar en la tienda web. No es una app: es **un solo archivo HTML estático** (`hepta-home-vN.html`) con CSS y JS inline, más la carpeta `Imagenes/`. No hay build, linter ni tests, ni dependencias salvo Google Fonts (Archivo y Lexend).

El número de versión del archivo cambia cuando el usuario lo renombra (v3_1 → v6 → v7). Trabaja siempre sobre el `hepta-home-v*.html` que exista en la raíz y no lo renombres por tu cuenta.

## Comandos

- Ver la página: abrir el HTML directo, o servirlo con `python -m http.server 8080` desde la raíz. Las imágenes usan rutas relativas, así que funciona de ambas formas.
- Captura headless para revisar el layout (Edge): `msedge.exe --headless --window-size=1440,6400 --virtual-time-budget=10000 --screenshot=<ruta absoluta>.png <url>`. La ruta de salida debe ser absoluta o falla con "Acceso denegado".
- Repo público: https://github.com/MatiasLaporta/hepta-landing (rama `main`). Se publica con la cuenta de `gh` **MatiasLaporta**; antes de pushear, `gh auth switch --user MatiasLaporta`. El usuario también usa `MatiasDigitals` para otros trabajos.

## Arquitectura del HTML

Orden de secciones (marcadas con comentarios `<!-- ... -->`): header sticky → hero (slider + franja de contadores) → categorías (carrusel) → obra gruesa (mosaico "bento" + sub-bloque de terciados) → cómo comprar → banda destacada (slider) → contacto → footer.

Cosas que requieren leer varias partes para entender:

- **Tokens de diseño en `:root`**: la paleta (naranja `#F7941D`, tinta, crema) la exige el cliente; no la cambies. Tipografías `--display` (Archivo 125% ancho, mayúsculas, clase `.display`) y `--body` (Lexend).
- **Dos carruseles, dos mecanismos distintos**:
  - Categorías (`#categorias`, dentro de `#catsWrap`): scroll horizontal nativo con `scroll-snap`; los botones `#catPrev`/`#catNext` hacen `scrollBy`. Se ven ~4,3 tarjetas a propósito para que la siguiente asome; `cState()` activa/desactiva botones y apaga el degradé y el texto "Desliza" (`.at-end`, `.off`) al llegar al final. Los anchos están en tres lugares (regla base y los `@media` de 1100 y 860 px) y deben cambiarse juntos.
  - Hero (`.slide`) y banda destacada (`.bslide`): fundido por opacidad con autoplay. La banda apila sus slides en una grilla de una celda (`.bslides{display:grid}` + `grid-area:1/1`) para que la altura sea la del más alto; no los pases a `position:absolute`.
- **Tarjetas con foto vs. foto de producto**: las imágenes de producto del sitio tienen fondo blanco, por eso `.ph.prod` usa `object-fit:contain`, y `.tile.prod` parte la tarjeta en imagen arriba y texto sobre fondo oscuro. Las fotos de obra usan `cover` con degradé. Elegir la clase correcta según el tipo de imagen.
- **Contadores** (`.counter b[data-to]`): animan de 0 al valor al entrar en pantalla; `data-prefix`/`data-suffix` agregan el `+` en naranja. Las cifras son datos del cliente; no las inventes ni las cambies sin que lo pida.
- **SEO/AEO/GEO**: el `<head>` lleva title, meta description, canonical y og con `https://www.heptachile.cl/`, más un JSON-LD `HardwareStore` con dirección, teléfonos, horario y catálogo. Los textos de la página están escritos con palabras clave; si cambias datos de contacto, horarios o categorías, actualiza también el JSON-LD.
- El formulario de contacto es el embed JavaScript de Hapee (`<div data-zentru-form="774691286/solicitud-de-cotizaci-n-landing-hepta" data-mode="inline">` + `https://beta.hapee.ai/static/form-embed.js`, dentro de `.form-card`, que no debe llevar fondo ni borde: el formulario ya trae su propia tarjeta y su página es transparente dentro del iframe). El script crea un iframe con alto automático y reenvía a Hapee las UTM/gclid/fbclid de la landing, por eso se usa en vez de un iframe plano. Campos, colores y opciones (p. ej. el desplegable Categoría) se editan en Hapee, no en el HTML.

## Responsive y favicon

- En móvil (`@media (max-width:860px)`) los textos grandes (H1 del hero, contadores, títulos y recuadros de la banda, botones) usan unidades `vw`/`clamp()` en vez de px fijos, para que se ajusten porcentualmente al ancho de cualquier celular. No les devuelvas tamaños en px. Las grillas del mismo bloque usan `minmax(0,1fr)` para que un hijo con `white-space:nowrap` no ensanche la columna.
- Edge headless no baja de ~500 px de ancho de layout; para probar 360 px, carga la página dentro de un `<iframe style="width:360px">` en un HTML temporal (y bórralo después).
- Favicon: `Public/favicon.svg` (entregado por Hepta; es un PNG incrustado) más `favicon-32.png`, `favicon-192.png`, `apple-touch-icon.png` y `favicon.ico` generados desde él. La carpeta se llama `Public` con P mayúscula; las rutas del `<head>` deben respetarlo.

## Imágenes

- `Imagenes/Banners/` y `Imagenes/Fotos Categoria/` (con subcarpeta `Terciados/`) contienen los **PNG/JPG originales** que entregó Hepta (2–6 MB) y, al lado, las **copias `.webp` optimizadas** con nombres sin espacios ni tildes. El HTML solo debe referenciar los `.webp`; las rutas con espacios van como `%20`.
- Al recibir fotos nuevas: generar el `.webp` (Pillow, 1920 px de ancho para banners, ~900 px para fotos de categoría, q≈80), dejar el original intacto y no referenciarlo desde el HTML.
- Lo que no tiene foto de Hepta usa imágenes del propio sitio (`https://heptachile.cl/wp-content/uploads/...`), enlazadas en remoto.

## Convenciones del contenido

- Textos en español de Chile, tuteo ("tu obra"), tono directo.
- Todos los enlaces de producto apuntan a las URLs reales de heptachile.cl y abren en pestaña nueva (`target="_blank" rel="noopener"`). WhatsApp de ventas: `https://wa.me/56987690593`.
- Datos oficiales: 2 bodegas con retiro (e-commerce en Lo Espejo y terciados en Lampa); despacho en 2 a 3 días hábiles; los terciados se venden por pallets cerrados. Si el sitio real y la landing se contradicen, manda heptachile.cl.
- El usuario fija el texto exacto de muchas secciones (descripciones de categorías, terciados, tarjeta "Obra gruesa"); no las reescribas por SEO sin pedirlo.
