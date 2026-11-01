# Ayuda: realidad aumentada con MindAR y A-Frame

Guía práctica para trabajar con [MindAR](https://hiukim.github.io/mind-ar-js-doc/) (seguimiento de imágenes en el navegador) y [A-Frame](https://aframe.io/) (escenas 3D en HTML). Está basada en el documento **"Realidad aumentada - MindAR" de Pablo Gutierrez**: se actualizaron las versiones, se corrigieron algunos errores y se completó con la documentación oficial y con lo aprendido en este repositorio.

> Versiones usadas en este repo: **A-Frame 1.5.0** y **MindAR 1.2.5**. MindAR 1.2.5 es la última versión publicada en npm (enero de 2024).

---

## Índice

1. [Instalación](#1-instalación)
2. [Estructura básica de una página](#2-estructura-básica-de-una-página)
3. [Crear el archivo `.mind`](#3-crear-el-archivo-mind)
4. [Parámetros de `mindar-image`](#4-parámetros-de-mindar-image)
5. [Corregir la vibración de los contenidos](#5-corregir-la-vibración-de-los-contenidos)
6. [Eventos y control desde JavaScript](#6-eventos-y-control-desde-javascript)
7. [Mostrar un video](#7-mostrar-un-video)
8. [Mostrar un GIF](#8-mostrar-un-gif)
9. [Textos, acentos y la letra ñ](#9-textos-acentos-y-la-letra-ñ)
10. [Probar en el teléfono](#10-probar-en-el-teléfono)
11. [Notas y limitaciones](#11-notas-y-limitaciones)
12. [Dónde buscar ayuda](#12-dónde-buscar-ayuda)

---

## 1. Instalación

Agregá estos dos scripts en el `<head>`. **El orden importa**: primero A-Frame y después MindAR.

```html
<script src="https://aframe.io/releases/1.5.0/aframe.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/mind-ar@1.2.5/dist/mindar-image-aframe.prod.js"></script>
```

> **Cambio respecto del documento original:** el PDF usa A-Frame 1.2.0 y MindAR 1.1.5, con un tercer script (`mindar-image.prod.js`). Desde MindAR 1.2.x, con A-Frame alcanza con `mindar-image-aframe.prod.js`. El otro paquete es para usar MindAR con Three.js directamente, sin A-Frame.

Fijá siempre la versión exacta (`@1.2.5`) y no uses `@latest`. Una actualización puede cambiar el comportamiento sin aviso.

---

## 2. Estructura básica de una página

```html
<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Prueba MindAR</title>
  <script src="https://aframe.io/releases/1.5.0/aframe.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/mind-ar@1.2.5/dist/mindar-image-aframe.prod.js"></script>
</head>
<body>
  <!--
    <a-scene>: la escena 3D.
      mindar-image: qué archivo .mind usar (las imágenes a reconocer).
      vr-mode-ui y device-orientation-permission-ui: se desactivan porque no se usan en AR.
  -->
  <a-scene mindar-image="imageTargetSrc: ./targets.mind;"
           color-space="sRGB"
           renderer="colorManagement: true; physicallyCorrectLights: true"
           vr-mode-ui="enabled: false"
           device-orientation-permission-ui="enabled: false">

    <!-- Recursos precargados para usar dentro de las entidades -->
    <a-assets>
      <img id="angel" src="./angel.png">
      <img id="diablo" src="./diablo.png">
    </a-assets>

    <!-- Cámara fija: MindAR mueve los objetos, no la cámara -->
    <a-camera position="0 0 0" look-controls="enabled: false"></a-camera>

    <!-- Cada entidad se ancla a una imagen del .mind según su targetIndex -->
    <a-entity mindar-image-target="targetIndex: 0">
      <a-image src="#diablo" position="0 0 0" width="1" height="1"></a-image>
    </a-entity>

    <a-entity mindar-image-target="targetIndex: 1">
      <a-image src="#angel" position="0 0 0" width="1" height="1"></a-image>
    </a-entity>
  </a-scene>
</body>
</html>
```

En este ejemplo, si la cámara ve la araña (imagen 0) aparece el diablo, y si ve la paloma (imagen 1) aparece el ángel.

> **Correcciones respecto del PDF:**
> - El segundo `<a-image>` decía `id="#angel"`. Tiene que ser **`src="#angel"`**: `src` apunta al recurso y `id` es el nombre del propio elemento.
> - `imageTargetSrc: . /targets.mind` tenía un espacio de más. La ruta correcta es `./targets.mind`.
> - En `renderer`, las propiedades se separan con `;`, no con `,`.

**Unidades:** dentro de `mindar-image-target`, **1 unidad equivale al ancho de la imagen reconocida**. Un `a-plane` con `width="1"` cubre todo el ancho del cartel, sin importar su tamaño real.

---

## 3. Crear el archivo `.mind`

El archivo `.mind` guarda los "puntos característicos" de las imágenes que la cámara tiene que reconocer. Se genera con el compilador oficial:

**👉 https://hiukim.github.io/mind-ar-js-doc/tools/compile**

1. Arrastrá una o varias imágenes al recuadro.
2. Pulsá **Start** y esperá. Al terminar, el compilador muestra dónde detectó los puntos.
3. Pulsá **Download compiled** y guardá el `targets.mind` en la carpeta del proyecto.

### `targetIndex` funciona como un array

El orden en que cargás las imágenes define su índice:

| Orden en el compilador | `targetIndex` |
|---|---|
| 1.ª imagen (araña) | `0` |
| 2.ª imagen (paloma) | `1` |
| 3.ª imagen | `2` |
| … | … |

Anotá ese orden: el `.mind` no guarda los nombres de las imágenes.

### Qué imágenes funcionan bien

- **Muchos detalles bien repartidos**: texturas, bordes, texto o ilustraciones. Las zonas lisas o vacías no aportan puntos.
- **Buen contraste** y sin patrones que se repitan (azulejos, rayas, grillas uniformes).
- **Evitá superficies brillantes** o con reflejos (vidrio, plástico laminado brillante).
- **Mejor impresas que fotografiadas del lugar real**. Lo que se reconoce en la vida real depende de la luz y del ángulo. Un cartel impreso a partir de la misma imagen que compilaste es mucho más estable.
- Mirá la visualización del compilador: si los puntos se concentran en una sola zona, la detección va a ser débil.
- Para un mismo lugar podés compilar **varias fotos** (de frente, de costado, con distinta luz) en un `.mind`. Así lo hace este repo (ver [sección 11](#11-notas-y-limitaciones)).

---

## 4. Parámetros de `mindar-image`

Se escriben en el atributo de la escena separados por `;`:

```html
<a-scene mindar-image="imageTargetSrc: ./targets.mind; maxTrack: 1; filterMinCF: 0.001; filterBeta: 1000">
```

| Parámetro | Por defecto | Para qué sirve |
|---|---|---|
| `imageTargetSrc` | — | Ruta al `.mind` (obligatorio) |
| `maxTrack` | `1` | Cuántas imágenes se siguen a la vez |
| `autoStart` | `true` | Si es `false`, la cámara no arranca hasta llamar a `start()` (ver [sección 6](#6-eventos-y-control-desde-javascript)) |
| `filterMinCF` | `0.001` | Suavizado: **más bajo = menos vibración** |
| `filterBeta` | `1000` | Suavizado: **más alto = menos retraso** |
| `warmupTolerance` | `5` | Cuadros seguidos en que debe verse la imagen para disparar `targetFound` |
| `missTolerance` | `5` | Cuadros seguidos sin verla para disparar `targetLost` |
| `uiLoading` / `uiScanning` / `uiError` | `yes` | Pantallas propias de MindAR. Con `no` las ocultás para usar tu propia interfaz |
| `showStats` | `false` | Muestra un panel de rendimiento (útil para depurar) |

---

## 5. Corregir la vibración de los contenidos

MindAR suaviza la posición con un filtro (One Euro Filter) que tiene dos perillas, y **las dos compiten entre sí**:

- **Bajar `filterMinCF`** reduce el temblor.
- **Subir `filterBeta`** reduce el retraso cuando movés la cámara rápido.

El PDF propone:

```html
mindar-image="imageTargetSrc: ./prueba.mind; filterMinCF: 0.0001; filterBeta: 0.001"
```

Con esos valores el temblor casi desaparece, pero `filterBeta: 0.001` hace que el contenido **"persiga" al cartel con mucho retraso** si movés el teléfono. Probá en este orden:

1. Bajá solo `filterMinCF` (`0.0001`) y dejá `filterBeta` en `1000`.
2. Si todavía tiembla, bajá `filterBeta` de a poco (`100`, `10`, `1`…) hasta lograr un equilibrio.
3. Subí `warmupTolerance` (por ejemplo `8`) para evitar que el contenido aparezca y desaparezca por detecciones falsas.

Más allá de los números, lo que más ayuda es **una buena imagen** (ver [sección 3](#qué-imágenes-funcionan-bien)), **buena luz** y **carteles impresos planos**. Otra alternativa son los marcadores cuadrados con borde negro (estilo AR.js), que se detectan con mucha estabilidad, aunque son menos estéticos.

---

## 6. Eventos y control desde JavaScript

Documentación oficial: https://hiukim.github.io/mind-ar-js-doc/examples/events-handling

### Eventos

| Evento | Dónde se escucha | Cuándo ocurre |
|---|---|---|
| `arReady` | `<a-scene>` | La cámara y MindAR están listos |
| `arError` | `<a-scene>` | Falló la cámara o la carga del `.mind` |
| `targetFound` | `<a-entity mindar-image-target>` | La imagen entró en cámara |
| `targetLost` | `<a-entity mindar-image-target>` | La imagen dejó de verse |

```js
const sceneEl = document.querySelector('a-scene');
const target = document.querySelector('#mi-target');

sceneEl.addEventListener('arReady', () => console.log('AR listo'));
sceneEl.addEventListener('arError', (e) => console.error('Error AR', e.detail));
target.addEventListener('targetFound', () => console.log('¡Encontrado!'));
target.addEventListener('targetLost', () => console.log('Perdido'));
```

### Arrancar y detener la cámara a mano

Con `autoStart: false`, la cámara no se abre al cargar la página. Así podés mostrar primero un botón "Comenzar", que además es necesario para pedir el permiso de cámara de forma amable:

```js
const sceneEl = document.querySelector('a-scene');
// Esperá a que A-Frame cargue la escena antes de pedir el sistema
sceneEl.addEventListener('loaded', () => {
  const arSystem = sceneEl.systems['mindar-image-system'];
  document.querySelector('#start').addEventListener('click', () => arSystem.start());
  document.querySelector('#stop').addEventListener('click', () => arSystem.stop());
});
```

| Método | Qué hace |
|---|---|
| `start()` | Abre la cámara y empieza a buscar imágenes |
| `stop()` | Detiene el seguimiento y la cámara |
| `pause(keepVideo)` | Pausa el seguimiento. Con `true`, la imagen de la cámara sigue visible |
| `unpause()` | Reanuda después de `pause()` |
| `switchTarget(index)` | Limita la búsqueda a una sola imagen del `.mind`. No figura en la documentación, pero existe en el código de la 1.2.5 |

> Los valores por defecto y los métodos de esta guía se verificaron contra el código fuente de MindAR 1.2.5 (`src/image-target/aframe.js` y `controller.js`).

---

## 7. Mostrar un video

**1. Precargá el video en `<a-assets>`:**

```html
<a-assets>
  <video id="cat-video" src="./images/cat.mp4"
         crossorigin="anonymous" loop muted playsinline webkit-playsinline></video>
</a-assets>
```

**2. Ponelo en la entidad con `<a-video>`:**

```html
<a-entity mindar-image-target="targetIndex: 0" id="video-target">
  <a-video src="#cat-video" width="1" height="0.5625" position="0 0 0"></a-video>
</a-entity>
```

(`0.5625` = 9/16, para que un video 16:9 ocupe el ancho del cartel).

**3. Reproducilo y pausalo con los eventos:**

```js
document.addEventListener('DOMContentLoaded', () => {
  const video = document.getElementById('cat-video');
  const target = document.getElementById('video-target');
  target.addEventListener('targetFound', () => video.play());
  target.addEventListener('targetLost', () => video.pause());
});
```

### Reproducción automática y iPhone

El PDF dice que "en iOS los videos no funcionan". En realidad, Safari **sí** los reproduce si se cumplen estas condiciones:

- El `<video>` tiene **`muted`** y **`playsinline`**. Sin `muted`, el navegador bloquea `play()` si no hubo un toque del usuario.
- Si querés sonido, la primera reproducción tiene que salir de un **toque del usuario** (un botón). Después se puede pausar y reanudar con los eventos.
- Usá **MP4 con H.264**: algunos iPhone viejos no reproducen HEVC.
- En **modo de bajo consumo**, iOS puede bloquear la reproducción automática aunque todo esté bien configurado.

---

## 8. Mostrar un GIF

A-Frame no anima GIFs por sí solo. Se usa el componente **aframe-gif-shader**:

```html
<!-- en el <head>, después de A-Frame -->
<script src="https://cdn.jsdelivr.net/npm/aframe-gif-shader/dist/aframe-gif-shader.min.js"></script>
```

```html
<a-entity mindar-image-target="targetIndex: 0">
  <a-plane position="0 0 0" width="1" height="1"
           material="shader: gif; src: url(./images/pato.gif)"></a-plane>
</a-entity>
```

También se puede usar con `<a-image>` o con un `<a-entity>` que tenga `geometry` y `material`.

### GIF sin fondo (transparente)

El shader no borra el cuadro anterior antes de dibujar el siguiente, así que en un GIF transparente los cuadros se van superponiendo. Para corregirlo:

1. Cloná https://github.com/mayognaise/aframe-gif-shader dentro del proyecto.
2. En `dist/aframe-gif-shader.min.js`, buscá:
   ```js
   __draw:function(){this.__ctx.drawImage(this.__frames[this.__frameIdx],0,0,this.__width,this.__height),this.__texture.needsUpdate=!0;}
   ```
   y reemplazalo por:
   ```js
   __draw:function(){this.__clearCanvas();this.__ctx.drawImage(this.__frames[this.__frameIdx],0,0,this.__width,this.__height),this.__texture.needsUpdate=!0;}
   ```
3. Cargá tu copia local en lugar de la del CDN:
   ```html
   <script src="./aframe-gif-shader/dist/aframe-gif-shader.min.js"></script>
   ```
4. Agregá `transparent: true` al material: `material="shader: gif; src: url(./pato.gif); transparent: true"`.

> ⚠️ `aframe-gif-shader` es un paquete viejo y sin mantenimiento, pensado para versiones anteriores de A-Frame. **Probalo con A-Frame 1.5.0 antes de depender de él.** El PDF también indica que los GIF de unos **400 × 400 px** funcionan mejor. Alternativas más estables: un **video MP4** (sección 7), o un **WebM con canal alfa** si necesitás transparencia (no funciona en Safari/iOS).

---

## 9. Textos, acentos y la letra ñ

El PDF dice que `<a-text>` "no soporta acentos". Lo verificamos: **la fuente `roboto` que trae A-Frame tiene solo los 95 caracteres ASCII (códigos 32 a 126)**. No tiene `á é í ó ú ñ`, `¿ ¡`, `º`, `·`, `—`, etc. Esos caracteres no se dibujan.

Hay tres soluciones:

1. **Generar una fuente propia con los caracteres del español** (recomendado):
   1. Entrá a https://msdf-bmfont.donmccurdy.com/
   2. Subí una fuente `.ttf` (por ejemplo Roboto o DM Sans, de Google Fonts).
   3. En *charset*, agregá los caracteres que necesitás: `áéíóúÁÉÍÓÚñÑüÜ¿¡º·—“”`.
   4. Descargá el zip y copiá el `.json` y el `.png` al proyecto.
   5. Usalo así:
      ```html
      <a-text value="Junín · Estación" font="./fonts/DMSans-msdf.json" negate="false"></a-text>
      ```
2. **Escribir el texto dentro de una imagen** (PNG) y mostrarla con `<a-image>`. Es lo más simple si el texto no cambia.
3. **Dibujar el texto en un `<canvas>`** con las fuentes del navegador y usarlo como textura. Admite cualquier carácter, pero requiere más código.

> Las fuentes que vienen con A-Frame se descargan desde `cdn.aframe.io`. Si alojás tu propia fuente, además funciona sin conexión.

---

## 10. Probar en el teléfono

La cámara (`getUserMedia`) **solo funciona en un contexto seguro**: `https://` o `http://localhost`. Abrir el HTML con doble clic (`file://`) no sirve.

| Situación | Cómo |
|---|---|
| En la computadora | `python3 -m http.server 8000` → `http://localhost:8000`, o la extensión **Live Server** de VS Code |
| En el teléfono | Publicar con HTTPS (GitHub Pages, Netlify, etc.) o usar un túnel: `ngrok http 8000`, `cloudflared tunnel --url http://localhost:8000` |

`http://192.168.x.x:8000` desde el teléfono **no** funciona, porque no es un contexto seguro.

### Ver la consola del teléfono

- **Android + Chrome**: conectá el teléfono por USB, activá la depuración USB y abrí `chrome://inspect` en la computadora.
- **iPhone + Safari**: en el iPhone, *Ajustes → Safari → Avanzado → Inspector web*. En la Mac, *Safari → Desarrollo → [tu iPhone]*.

---

## 11. Notas y limitaciones

| Nota | Detalle / solución |
|---|---|
| **Un `targetIndex` por entidad** | No se puede escribir `targetIndex: 0, 1, 2`. Para mostrar lo mismo con varias imágenes, creá **una entidad por índice** con el mismo contenido (con JavaScript es fácil: `Array.from({ length: N }, (_, i) => …)`). Así lo hacen `archivo-vivo` (28 imágenes, clona el rig) y `recorrido-oficinas` (`buildTargets()`). |
| **Tiempo de carga** | MindAR tarda unos segundos en estar listo (el PDF habla de un mínimo de ~3 s), y los `.mind` grandes tardan más en descargarse. Mostrá un mensaje de "Preparando cámara…" y esperá a `arReady`. |
| **Peso de los `.mind`** | Aumenta con la cantidad de imágenes. En este repo: `clarolab-museo.mind` (28 imágenes) pesa unos 14 MB, mientras que cada `.mind` del recorrido de oficinas pesa entre 0,7 y 1,4 MB. |
| **Muchos targets en un `.mind`** | Cuantas más imágenes, más lenta la detección. `recorrido-oficinas` divide el recorrido en un `.mind` por estación y los carga de a uno. |
| **Acentos en `<a-text>`** | Ver [sección 9](#9-textos-acentos-y-la-letra-ñ). |
| **Videos en iOS** | Funcionan con `muted` + `playsinline` (ver [sección 7](#7-mostrar-un-video)). |
| **Dependencia de CDN** | Sin Internet no cargan A-Frame ni MindAR. Para una exhibición sin conexión, descargá los `.js` y servilos localmente. |
| **API interna** | `recorrido-oficinas` usa `arSystem._startAR()` (método privado) para cambiar de `.mind` sin cerrar la cámara. No actualices MindAR sin probar. |

---

## 12. Dónde buscar ayuda

- **La consola del navegador (F12)**: casi todos los errores aparecen ahí (404 del `.mind`, permisos de cámara, errores de JavaScript). Copiá el mensaje exacto para buscarlo.
- **Documentación de MindAR**: https://hiukim.github.io/mind-ar-js-doc/
  - Configuración del seguimiento: https://hiukim.github.io/mind-ar-js-doc/quick-start/tracking-config
  - Ejemplos (varios targets, interfaz propia, eventos): https://hiukim.github.io/mind-ar-js-doc/examples/summary
- **Documentación de A-Frame 1.5.0**: https://aframe.io/docs/1.5.0/introduction/
  - Componente de texto y fuentes: https://aframe.io/docs/1.5.0/components/text.html
- **Repositorio de MindAR (issues)**: https://github.com/hiukim/mind-ar-js/issues. Buscá en inglés: la mayoría de las preguntas y respuestas están en ese idioma.
- **Ejemplo completo del PDF original**: https://github.com/pablitogutierrez/MindAR_Documentacion
- **Asistentes de IA**: sirven para entender un error. Pegales el mensaje de la consola y el fragmento de código, y verificá la respuesta contra la documentación oficial.
- **Personas que ya usaron la librería** y, sobre todo, **probar por tu cuenta**: cambiar un parámetro a la vez y observar el resultado.

---

*Basado en "Realidad aumentada - MindAR" de Pablo Gutierrez. Ampliado y actualizado en noviembre de 2026 con la documentación oficial de MindAR y A-Frame.*
