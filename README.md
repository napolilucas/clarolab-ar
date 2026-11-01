# Recorrido AR de Clarolab

Sitio web estático que guía a los visitantes por las oficinas de Clarolab (Junín, Argentina) usando **realidad aumentada basada en imágenes**. El visitante abre la página en el teléfono, activa la cámara y apunta a los carteles de cada estación. Sobre cada cartel aparece una tarjeta 3D con información de ese lugar.

No hay backend, build ni dependencias que instalar. Son archivos HTML, CSS y JavaScript que cargan **A-Frame 1.5.0** y **MindAR 1.2.5** desde CDN.

---

## Índice

- [Inicio rápido](#inicio-rápido)
- [Cómo funciona la experiencia](#cómo-funciona-la-experiencia)
- [Estructura del proyecto](#estructura-del-proyecto)
- [Configuración de estaciones](#configuración-de-estaciones)
- [Arquitectura de `workplace.js`](#arquitectura-de-workplacejs)
- [Interfaz y estilos](#interfaz-y-estilos)
- [Agregar o modificar una estación](#agregar-o-modificar-una-estación)
- [Publicación](#publicación)
- [Limitaciones conocidas](#limitaciones-conocidas)

---

## Inicio rápido

La cámara (`getUserMedia`) solo funciona en un **contexto seguro**: `https://` o `http://localhost`. Si abrís el HTML con doble clic (`file://`), el navegador bloquea la cámara y la carga de los `.mind`.

```bash
# desde la raíz del proyecto
python3 -m http.server 8000
```

Abrí `http://localhost:8000/`. Necesitás conexión a Internet para descargar A-Frame y MindAR.

Para probar desde un teléfono, el sitio tiene que estar servido por HTTPS. Las opciones son publicarlo (ver [Publicación](#publicación)) o usar un túnel HTTPS hacia tu servidor local. Una IP de la red local con `http://` no alcanza.

---

## Cómo funciona la experiencia

```
index.html ──(toque en el GIF)──► workplace.html
                                     │
                         "Comenzar el recorrido"
                                     │
                                     ▼
                        permiso de cámara + MindAR
                                     │
              ┌──────────────────────┴───────────────────────┐
              ▼                                              │
   Estación N: se carga su .mind                             │
              │                                              │
   ¿se detecta el cartel? ──sí──► se muestra la tarjeta 3D   │
              │                         │                    │
              no (30 s)          se pierde el cartel (2,5 s) │
              │                         │                    │
              └──────────► estación N+1 ┴────────────────────┘
                     (después de la última vuelve a la primera)
```

1. **Portada (`index.html`)**: muestra el título "Recorrido Clarolab" y un GIF animado de escáner. El GIF **no lee QR**: es un enlace a `workplace.html`. El QR impreso en la entrada tiene que apuntar a la URL pública de `workplace.html` (o de la portada).
2. **Bienvenida (`workplace.html`)**: el botón **Comenzar el recorrido** oculta la bienvenida, muestra el visor y pide permiso para la cámara.
3. **Recorrido por estaciones**: se analiza **un solo archivo `.mind` a la vez**. Cada estación tiene su propio `.mind`, que puede contener varias fotos del mismo lugar.
   - Cuando MindAR reconoce cualquiera de esas fotos, se muestra la tarjeta de la estación y el estado pasa a `Punto N de 5 · TÍTULO`.
   - Cuando el cartel deja de verse, el recorrido avanza a la siguiente estación después de `lostAdvanceMs` (2,5 s).
   - Si no se detecta nada, avanza después de `idleAdvanceMs` (30 s).
   - La cámara sigue encendida durante los cambios de estación. Después de la última estación, el recorrido vuelve a la primera.
4. **Salir**: el botón **×** o la tecla **Escape** detienen la cámara y vuelven a la bienvenida. El logo `C/CLAROLAB` del visor lleva a la portada.

---

## Estructura del proyecto

```
.
├── index.html             Portada con enlace al recorrido
├── tour.css               Estilos de la portada
├── workplace.html         Bienvenida + escena A-Frame + HUD del escáner
├── workplace.css          Estilos de la bienvenida y del HUD sobre la cámara
├── workplace-content.js   Datos: estaciones, archivos .mind, textos y tiempos
├── workplace.js           Lógica: cámara, MindAR, tarjetas 3D y avance del recorrido
└── assets/
    ├── targets_entrada.mind    Estación 1 · Entrada (4 imágenes)
    ├── targets_ofi2.mind       Estación 2 · Oficina (3 imágenes)
    ├── targets_ofiFran.mind    Estación 3 · Oficina de Fran (4 imágenes)
    ├── termica.mind            Estación 4 · Tablero eléctrico (1 imagen)
    ├── cuadros_color.mind      Estación 5 · Cuadros / equipo (1 imagen)
    ├── termica.jpg             Foto original del tablero (referencia, no se carga)
    ├── cuadrados_color.jpg     Foto original de los cuadros (referencia, no se carga)
    ├── panelpruea.png          Fondo de las tarjetas AR
    ├── qr-scan.gif             Animación de escáner (portada y visor)
    ├── ars_scan.svg            Ilustración de la bienvenida
    └── Clarolab.svg            Logo (no se usa actualmente en las páginas)
```

### Orden de carga de scripts (`workplace.html`)

```html
<script src="https://aframe.io/releases/1.5.0/aframe.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/mind-ar@1.2.5/dist/mindar-image-aframe.prod.js"></script>
<script src="./workplace-content.js"></script>   <!-- define window.WORKPLACE_CONTENT -->
<script src="./workplace.js" defer></script>      <!-- se ejecuta con el DOM ya leído -->
```

MindAR se registra como componente de A-Frame, así que tiene que cargarse después de A-Frame. `workplace-content.js` define los datos antes de que `workplace.js` los lea.

---

## Configuración de estaciones

Todo el contenido está en `workplace-content.js`, en el objeto global `window.WORKPLACE_CONTENT`:

```js
window.WORKPLACE_CONTENT = {
  groups: [
    {
      id: 'entrada',                          // prefijo para los id de las entidades
      file: './assets/targets_entrada.mind',  // marcador de la estación
      targetCount: 4,                         // cantidad de imágenes dentro del .mind
      title: 'BIENVENIDOS',
      lines: ['…', '…', '…', '…'],            // exactamente 4 líneas
      footer: 'DE JUNÍN AL MUNDO',
      accent: '#00e6a0', background: '#07140f', visual: 'welcome' // ver Limitaciones
    },
    // …
  ],
  idleAdvanceMs: 30000,  // avanzar si no se detecta nada
  lostAdvanceMs: 2500    // avanzar después de perder el cartel
};
```

| # | `id` | Archivo | Imágenes | Título |
|---|------|---------|----------|--------|
| 1 | `entrada` | `targets_entrada.mind` | 4 | BIENVENIDOS |
| 2 | `oficina2` | `targets_ofi2.mind` | 3 | OFICINA N.º 1 |
| 3 | `fran` | `targets_ofiFran.mind` | 4 | FRAN |
| 4 | `termica` | `termica.mind` | 1 | SEGURIDAD |
| 5 | `cuadros` | `cuadros_color.mind` | 1 | UN GRAN EQUIPO |

El orden del array define el orden del recorrido. Todas las imágenes de un mismo `.mind` muestran la misma tarjeta.

---

## Arquitectura de `workplace.js`

El archivo está envuelto en una IIFE (`(() => { … })()`) para no contaminar el ámbito global.

### Estado

| Variable | Propósito |
|----------|-----------|
| `arSystem` | Instancia de `mindar-image-system` |
| `active` | La cámara está corriendo (se activa con `arReady`) |
| `starting` | `openAR()` está en curso; evita aperturas dobles |
| `closeRequested` | El usuario cerró durante el arranque; la limpieza se completa en `finally` |
| `currentGroupIndex` | Estación actual |
| `targetEntities` | Entidades `<a-entity mindar-image-target>` de la estación actual |
| `detectedTargets` | `Set` de objetivos visibles en este momento |
| `routeAdvanceTimer` | Timer de avance (inactividad o pérdida del cartel) |
| `switchingGroup` | Hay un cambio de estación en curso |

### Funciones

| Función | Qué hace |
|---------|----------|
| `addPanelBackground(card)` | Agrega un `a-plane` con `panelpruea.png` como fondo de la tarjeta |
| `createText(parent, id, value, …)` | Crea un `a-text` (Roboto, shader MSDF) |
| `buildTargets(group, index)` | Elimina las entidades anteriores y crea `targetCount` entidades `mindar-image-target`, cada una con su tarjeta (marca "CLAROLAB · ESTACIÓN 0N", título, divisor, 4 líneas y pie). Registra `targetFound` / `targetLost` |
| `setStatus(msg)` | Actualiza el mensaje del HUD (`role="status"`, `aria-live="polite"`) |
| `scheduleIdleAdvance()` | Programa el avance por inactividad |
| `advanceRouteGroup()` | Detiene y libera el controlador actual, pasa a la siguiente estación (con módulo), arma sus objetivos y reinicia el tracking con `arSystem._startAR()` sin cerrar la cámara. Si falla, reintenta a los 5 s |
| `showScanner(msg)` / `restoreWelcome()` | Alternan entre la bienvenida y el visor y manejan `aria-hidden`, la clase `ar-active` y el foco |
| `waitForScene()` | Espera el evento `loaded` de A-Frame, con un límite de 10 s |
| `openAR()` | Verifica el contexto seguro, espera la escena, arma los objetivos, configura `mindar-image` y llama a `arSystem.start()`. Traduce los errores a mensajes para el usuario (archivo/red, permisos, genérico) |
| `closeAR()` | Cancela los timers, quita el componente `mindar-image` (esto detiene la cámara) y vuelve a la bienvenida. Si el cierre ocurre durante el arranque, delega la limpieza al `finally` de `openAR()` |

### Parámetros de MindAR

```js
scene.setAttribute('mindar-image', {
  imageTargetSrc: group.file,
  autoStart: false,              // la cámara arranca al pulsar el botón
  uiLoading: 'no', uiScanning: 'no', // se usa el HUD propio
  filterMinCF: 0.001, filterBeta: 1000, // suavizado (filtro One-Euro)
  warmupTolerance: 8,            // frames necesarios para confirmar una detección
  missTolerance: 15,             // frames sin ver el cartel antes de dar targetLost
  maxTrack: 1                    // un cartel a la vez
});
```

La tarjeta se posiciona en coordenadas del objetivo, donde el ancho de la imagen equivale a 1. El fondo mide `0.72 × 1.067` y está desplazado `(-.02, .2, .08)`.

---

## Interfaz y estilos

**`tour.css` (portada)**
- `.welcome-art` es un lienzo vertical que pasa a horizontal en pantallas ≥ 760 px o en orientación apaisada.
- `.scan-entry` posiciona el GIF. `mix-blend-mode: screen` hace que el negro del GIF se funda con el fondo.

**`workplace.css` (recorrido)**
- `.welcome-panel`: bienvenida en dos columnas en escritorio y una en móvil.
- `.scan-panel`: siempre está en el DOM, pero invisible hasta tener `.is-visible`. Así A-Frame puede inicializarse antes de abrir la cámara.
- `.scan-hud`: capa superpuesta con `pointer-events: none`, excepto el botón de cierre.
- `.finder`: las esquinas verdes y la línea de escaneo, hechas solo con CSS.
- `env(safe-area-inset-*)` respeta los notch y las barras del sistema.
- Tiene ajustes para ≤ 760 px, ≤ 380 px, alturas bajas y `prefers-reduced-motion`.

Paleta: `--ink #050806`, `--green #00e6a0`, `--paper #f2f8f5`.

---

## Agregar o modificar una estación

1. **Compilá el marcador** con el [MindAR Image Targets Compiler](https://hiukim.github.io/mind-ar-js-doc/tools/compile). Podés subir varias fotos del mismo cartel o lugar para mejorar la detección y descargar el `.mind`.
2. Copiá el `.mind` en `assets/`. Conviene guardar también las fotos originales como referencia.
3. Agregá una entrada en `groups` dentro de `workplace-content.js`, con `file`, el `targetCount` exacto (la cantidad de fotos compiladas), `title`, **4** `lines` y `footer`.
4. Ubicá la entrada en la posición del array que corresponda al orden del recorrido.

Para actualizar una estación existente, alcanza con reemplazar su `.mind` (y `targetCount` si cambió la cantidad de fotos) o editar sus textos. No hace falta tocar `workplace.js`.

> Si `targetCount` es mayor que la cantidad real de imágenes del `.mind`, se crean entidades que nunca se detectan. Si es menor, algunas fotos se reconocen pero no muestran tarjeta.

---

## Publicación

Cualquier hosting estático con HTTPS sirve. Como el repositorio está en GitHub, **GitHub Pages** es la opción más directa: *Settings → Pages → Deploy from branch → `main` / root*.

Después de publicar, generá el QR físico con la URL pública.

---

## Limitaciones conocidas

- **API privada de MindAR**: el cambio de estación sin cerrar la cámara usa `arSystem._startAR()`, un método interno de MindAR 1.2.5. Actualizar MindAR puede romperlo, y en ese caso el HUD muestra un error. Mantené la versión fijada.
- **Campos sin uso**: `accent`, `background` y `visual` de cada estación todavía **no se aplican**. Los colores de la tarjeta están fijos en `#00e6a0` dentro de `workplace.js`.
- **Recorrido cíclico y automático**: las estaciones avanzan por tiempo, no por posición. Si el visitante apunta a un cartel que no es el de la estación actual, ese cartel no se reconoce hasta que llegue su turno.
- **Textos fijos de 4 líneas**: las posiciones Y de las líneas están fijas, así que con más de 4 líneas las que sobran no se muestran.
- **Dependencia de CDN**: sin Internet no cargan A-Frame ni MindAR.
- **Peso de los recursos**: los `.mind` suman unos 5 MB y `qr-scan.gif` pesa unos 3 MB. Conviene tenerlo en cuenta en redes móviles.
- **Nombres a revisar**: `panelpruea.png` (parece "panelprueba") y la estación `oficina2`, cuyo título es "OFICINA N.º 1".
