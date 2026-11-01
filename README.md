# Clarolab AR — Archivo Vivo

Recorrido interactivo de realidad aumentada para **Clarolab** (Junín, Buenos Aires). Es una página web estática que, al apuntar la cámara del teléfono a la puerta, la mascota o los paisajes de Clarolab, muestra una espiral 3D con las cinco paradas del recorrido:

| # | Capítulo | Tema |
|---|----------|------|
| 01 | Desarrollo | Productos con React, Spring Boot, microservicios, Python e IA |
| 02 | Testing | Pruebas automatizadas y manuales, CI/CD, rendimiento |
| 03 | Seguridad | Análisis estático/dinámico, detección de vulnerabilidades |
| 04 | Lab Creation | I+D, exploración de tecnologías, formación |
| 05 | Customer Journey / Equipo | Trabajo codo a codo con los aliados |

No requiere build ni backend: HTML + CSS + JavaScript, con las librerías cargadas desde CDN.

## Stack

| Librería | Versión | Uso |
|----------|---------|-----|
| [A-Frame](https://aframe.io) | 1.5.0 | Escena 3D (`<a-scene>`) y componentes |
| [MindAR](https://github.com/hiukim/mind-ar-js) (`mindar-image-aframe`) | 1.2.5 | Seguimiento de imágenes (image tracking) |
| [qrcodejs](https://github.com/davidshimjs/qrcodejs) | 1.0.0 | QR de acceso generado en la propia página |

## Estructura

```
.
├── museo.html          # Página principal (bienvenida, escena AR, colección)
├── museo.css           # Estilos, animaciones y responsive (breakpoint 760px)
├── script.js           # Lógica: espiral 3D, navegación, ciclo de vida de MindAR, QR
└── images/
    ├── clarolab-museo.mind   # Targets compilados (28 imágenes, ≈14 MB)
    └── img-utiles/           # Cuadros de la mascota (1–12.png) y tetris.gif
```

## Cómo funciona

### Página

1. **Bienvenida** (`#inicio`): presentación con la mascota guía.
2. **Recorrido** (`#recorrido`): vista previa 3D que avanza sola cada 4,8 s, botón **Iniciar** para abrir la cámara, tarjeta de detalle del capítulo y QR de acceso.
3. **Colección** (`#coleccion`): resumen en texto de las cinco paradas.

### Escena 3D (`script.js`)

- **`helix-spine`** (componente A-Frame): dibuja una hélice (`THREE.TubeGeometry`, 2,7 vueltas) y reparte sobre ella las cinco tarjetas `.artifact`. Agrega además cuatro planos con cuadros de la mascota que flotan alrededor de la espiral.
- **`selectExhibit(index)`**: cambia el capítulo activo. Actualiza los textos, resalta la tarjeta y anima la rotación y la traslación del rig para centrar la tarjeta elegida. También cambia el cuadro de la mascota guía.
- Los textos de cada capítulo están en el array `exhibits`, al comienzo de `script.js`.

### Realidad aumentada

- **`openAR()`**: verifica el contexto seguro y `getUserMedia`, configura `mindar-image` sobre la escena con `imageTargetSrc: ./images/clarolab-museo.mind` y arranca el sistema `mindar-image-system`.
- **Múltiples targets**: `prepareDoorAnchors()` clona el rig `#museumRig` en `activeMarkerCount` (28) anclas, una por imagen compilada en el `.mind`. Así **cualquier** imagen reconocida muestra la misma espiral. `maxTrack: 1` limita el seguimiento a una imagen a la vez.
- **`closeAR()`**: detiene MindAR, elimina las anclas clonadas y restaura la vista previa. Si el usuario cierra mientras la cámara arranca, el cierre se difiere con `closeRequested`.
- Los errores (permiso denegado, fallo de carga del `.mind`, origen no seguro) se muestran como mensajes en `#arStatus`.

### QR de acceso

`refreshQr()` genera un QR hacia la URL actual con `#recorrido`, **solo si la página se sirve por HTTPS**. Con otro protocolo muestra un placeholder.

## Ejecución local

La cámara solo funciona en un **contexto seguro** (HTTPS o `localhost`). Abrir `museo.html` con `file://` no alcanza.

```bash
# Desde la raíz del proyecto
python3 -m http.server 8080
# Abrir http://localhost:8080/museo.html
```

Para probar desde el teléfono hace falta HTTPS. Algunas opciones:

- Publicar en **GitHub Pages** (el repo ya está en `github.com/napolilucas/clarolab-ar`). La URL queda como `https://<usuario>.github.io/clarolab-ar/museo.html`, y el QR se genera solo.
- Usar un túnel HTTPS (`ngrok http 8080`, `cloudflared tunnel --url http://localhost:8080`, etc.).

> El `.mind` pesa unos 14 MB: en redes móviles lentas, la primera carga del AR puede demorar.

## Cambiar las imágenes de tracking

1. Compilar las imágenes con el [MindAR Image Targets Compiler](https://hiukim.github.io/mind-ar-js-doc/tools/compile).
2. Reemplazar `images/clarolab-museo.mind`, o cambiar `mindSource` en `script.js`.
3. Ajustar **`activeMarkerCount`** en `script.js` a la **cantidad exacta** de imágenes del `.mind`. Si queda por debajo, algunas imágenes no se detectan. Si queda por encima, se crean anclas sin target.
4. Si cambia el tipo de imágenes, actualizar también `activeMarkerName` (el texto que ve el usuario).

## Personalización rápida

| Qué | Dónde |
|-----|-------|
| Textos de los capítulos (vista AR) | Array `exhibits` en `script.js` |
| Etiquetas de las tarjetas 3D | `<a-text value="…">` en `museo.html` |
| Textos de la sección Colección | `.collection-grid` en `museo.html` |
| Colores | Variables en `:root` de `museo.css` (`--green: #00e6a0`, etc.) |
| Forma de la espiral | `turns`, `radius`, `height` en `helix-spine` |
| Intervalo de la vista previa | `setInterval(…, 4800)` en `script.js` |
| Escala de la espiral sobre el target | `'.36 .36 .36'` en `openAR()` |

## Observaciones y pendientes

Detectados al revisar el código:

- **Gesto de deslizar sin implementar:** el HUD dice *"Deslizá ↑↓ para avanzar · ↔ para girar"* y `onTargetFound` dice *"deslizá hacia arriba o abajo"*, pero no hay manejadores táctiles: en AR solo se navega con los botones ← →.
- **Nombre del capítulo 05:** la tarjeta 3D dice *EQUIPO* y `exhibits`/Colección dicen *CUSTOMER JOURNEY*.
- **`previewTimer`** sigue corriendo durante la sesión AR. No cambia de capítulo porque comprueba `arActive`, pero nunca se limpia.
- **`.vscode/launch.json`** apunta a una ruta absoluta de otra máquina (`/home/clarolab/Documents/GitHub/clarolabAr/museo.html`).
- **Dependencia de CDN:** sin conexión a internet no carga A-Frame ni MindAR. Para una exhibición offline convendría alojar las librerías localmente.
- Hay un comentario con un typo en `script.js` (`targn  et`).
