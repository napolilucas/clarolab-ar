# Clarolab AR

Experiencias de realidad aumentada en la web para **Clarolab** (Junín, Argentina). Son sitios estáticos, sin build ni backend, que usan **A-Frame 1.5.0** y **MindAR 1.2.5** (seguimiento de imágenes) desde CDN.

| Carpeta | Experiencia | Qué hace | Marcadores |
|---------|-------------|----------|------------|
| [`archivo-vivo/`](archivo-vivo/README.md) | **Archivo Vivo** | Al reconocer la puerta, la mascota o los paisajes, muestra una espiral 3D con 5 capítulos sobre lo que hace Clarolab (desarrollo, testing, seguridad, lab, equipo). | 1 `.mind` con 28 imágenes |
| [`recorrido-oficinas/`](recorrido-oficinas/README.md) | **Recorrido por las oficinas** | Recorre las oficinas estación por estación y muestra una tarjeta AR sobre el cartel de cada lugar. | 5 `.mind`, uno por estación, cargados de a uno |

## Estructura

```
.
├── index.html            Portada: elige entre las dos experiencias
├── museo.html            Redirección a archivo-vivo/ (enlaces y QR anteriores)
├── archivo-vivo/         Museo "Archivo Vivo"
│   ├── index.html · museo.css · script.js
│   └── images/
└── recorrido-oficinas/   Recorrido por las oficinas
    ├── index.html · workplace.html · *.css · *.js
    └── assets/
```

Cada carpeta funciona sola: todas sus rutas son relativas y no comparte archivos con la otra. Los detalles técnicos de cada experiencia están en su propio `README.md`.

## Ejecución local

La cámara solo funciona en un contexto seguro (HTTPS o `localhost`):

```bash
python3 -m http.server 8000
```

| URL | Página |
|-----|--------|
| `http://localhost:8000/` | Portada |
| `http://localhost:8000/archivo-vivo/` | Archivo Vivo |
| `http://localhost:8000/recorrido-oficinas/` | Recorrido por las oficinas |

Para probar desde un teléfono hace falta HTTPS: publicar el sitio o usar un túnel (`ngrok`, `cloudflared`).

## Publicación (GitHub Pages)

*Settings → Pages → Deploy from branch → `main` / root*. Las URLs quedan así:

- `https://napolilucas.github.io/clarolab-ar/` (portada)
- `https://napolilucas.github.io/clarolab-ar/archivo-vivo/`
- `https://napolilucas.github.io/clarolab-ar/recorrido-oficinas/`

`museo.html` se conserva en la raíz como redirección, para que sigan funcionando los enlaces o QR que apuntaban a `…/clarolab-ar/museo.html`.

## Historial

Este repositorio une dos proyectos que antes estaban separados:

- `archivo-vivo/` es el contenido original de este repo (`napolilucas/clarolab-ar`), movido a una subcarpeta.
- `recorrido-oficinas/` viene de `napolilucas/ofi-clarolab-ar` y se incorporó con `git subtree`, así que conserva su historial. Esos commits tienen los archivos en la raíz, porque así estaban en el repo original. Para verlos: `git log --oneline "$(git log --format=%H --grep='Incorporar recorrido de oficinas' -1)^2"`.

## Posibles mejoras

- Las dos experiencias repiten estilos (paleta, header, HUD) y la lógica de arranque y cierre de MindAR. Se podrían extraer a una carpeta `shared/` común.
- Alojar A-Frame y MindAR localmente para poder usarlas sin conexión.
