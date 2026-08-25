# Documentación técnica — jcrf-landing

Este documento explica cómo está armado el sitio, dónde vive cada cosa, y cómo
operarlo sin depender de esta conversación. Pensado para: (a) que Juan Carlos
lo use como referencia propia, (b) que sirva de contexto si en el futuro se
retoma este proyecto con Claude (u otra IA) sin memoria de esta conversación,
y (c) como respaldo ante una emergencia (perder acceso, migrar de proveedor, etc).

Última actualización: 2026-08-25.

---

## 1. Qué es esto, en una frase

Un sitio estático hecho con **Jekyll** (generador de sitios estáticos en Ruby),
usando el theme **AcademicPages** (fork de Minimal Mistakes), publicado con el
build nativo de **GitHub Pages** (sin Cloudflare, sin Netlify, sin nada
instalado en la Mac), y editable desde un panel web (**Sveltia CMS**) para no
tener que tocar código a mano.

No hay base de datos, no hay servidor propio, no hay backend. Todo el
contenido son archivos de texto (Markdown/YAML) dentro de un repositorio de
GitHub. Cada cambio en esos archivos dispara una reconstrucción automática del
sitio.

## 2. Dónde vive todo

| Cosa | Dónde |
|---|---|
| Código + contenido (repo) | https://github.com/JCRAMIREZFIGUEROA/jcrf-landing |
| Copia local en la Mac | `~/jcrf-landing` |
| Sitio publicado (URL de prueba) | https://jcramirezfigueroa.github.io/jcrf-landing/ |
| Panel de edición (CMS) | https://jcramirezfigueroa.github.io/jcrf-landing/admin/ |
| Theme original (upstream, MIT license) | https://github.com/academicpages/academicpages.github.io |

El repo es un **fork real** de GitHub (no una copia manual): existe una
relación con el repo original de AcademicPages, y localmente hay dos remotes
git — `origin` (tu copia) y `upstream` (el theme original), por si algún día
quieres traer actualizaciones del theme con `git pull upstream master`.

**El dominio real `jcramirezfigueroa.com` NO está conectado a este sitio.**
Sigue mostrando tu WordPress de siempre. Ver sección 8.

## 3. Cómo se publica (sin instalar nada)

GitHub Pages tiene un modo "legacy" (build nativo) que, apenas detecta un push
a la rama `master`, corre Jekyll en sus propios servidores y publica el
resultado. Flujo:

```
editas un archivo (via /admin/ o directo en GitHub o local)
        ↓
se hace commit + push a la rama master
        ↓
GitHub Pages reconstruye el sitio automáticamente (~30-60 seg)
        ↓
https://jcramirezfigueroa.github.io/jcrf-landing/ se actualiza solo
```

No hay que ejecutar `jekyll build`, no hay que subir archivos por FTP, no hay
"deploy button". Empujar a `master` **es** publicar.

## 4. Cómo editar — 3 formas, de más simple a más técnica

### A) Panel de edición (`/admin/`) — recomendado para el día a día

Sveltia CMS, un editor tipo formulario que hace commits al repo por ti. Login
con un **Personal Access Token** de GitHub (Settings → Developer settings →
Personal access tokens, con permiso de escritura sobre el repo) — no hace
falta OAuth ni ningún servicio intermedio porque eres el único usuario del
CMS (así lo recomienda la propia documentación de Sveltia).

Colecciones disponibles ahí mismo:
- **Blog** — crear/editar posts (título, fecha, tags, cuerpo).
- **Páginas → Portada** — el texto e imagen de la home.
- **Título del sitio** — nombre del sitio y descripción corta.
- **Autor / Contacto** — nombre, bio, y links (email, LinkedIn, ResearchGate,
  GitHub, Twitter/X, Bluesky, Google Scholar, ORCID).
- **Secciones del menú** — lista arrastrable de qué aparece en el menú de
  arriba, en qué orden.

**"Privado" en el Blog** = borrador. Un post nuevo/editado se guarda en una
rama/PR aparte y no aparece en el sitio hasta apretar "Publish" en el panel.

### B) Editar directo en github.com

Sin instalar nada: ir al archivo en el repo, click en el lápiz (editar),
guardar ("commit"). Útil para cosas que el CMS no cubre (ver sección 6).
Atajo: cambiar `/blob/` por `/edit/` en la URL del archivo salta directo al
editor.

### C) Editor local + git

El repo está clonado en `~/jcrf-landing`. Se puede abrir con cualquier editor
(VS Code, etc.) y subir cambios con:
```bash
cd ~/jcrf-landing
git add -A && git commit -m "cambio" && git push
```

## 5. Mapa de archivos — qué controla qué

```
jcrf-landing/
├── admin/
│   ├── index.html        # el panel de Sveltia CMS en si
│   └── config.yml        # que colecciones/campos existen en el panel
├── _config.yml           # configuracion tecnica de Jekyll (plugins, permalinks,
│                          # colecciones). A PROPOSITO no es editable desde el CMS:
│                          # si un CMS reescribe este archivo completo se puede
│                          # romper el sitio entero. Solo se toca a mano.
├── _data/
│   ├── site.yml           # titulo + descripcion del sitio (editable via CMS)
│   ├── authors.yml        # nombre, bio, links de contacto (editable via CMS)
│   └── navigation.yml     # que secciones aparecen en el menu de arriba (editable via CMS)
├── _pages/
│   ├── about.md            # la Portada (home, permalink: /)
│   ├── publications.html   # oculta del menu, no borrada
│   ├── talks.html          # oculta del menu, no borrada
│   ├── teaching.html       # oculta del menu, no borrada
│   ├── portfolio.html      # oculta del menu, no borrada
│   ├── cv.md                # oculta del menu, no borrada
│   └── year-archive.html   # la pagina "Blog" (listado de posts)
├── _posts/                # cada post del blog es un archivo .md aca
├── images/
│   └── profile.png         # la foto de Juan Carlos (real, tomada de jcramirezfigueroa.com)
└── _includes / _layouts / _sass/   # el theme en si (HTML/CSS). No suele hacer
                                      # falta tocarlo salvo ajustes de diseño.
```

## 6. Qué es editable desde `/admin/` y qué no

Por diseño, **no todo pasa por el CMS**. Ciertas cosas (plugins, permalinks,
estructura de colecciones de Jekyll) viven en `_config.yml` y se dejaron
fuera del CMS a propósito: un editor tipo formulario que gestiona un archivo
YAML completo puede sobrescribirlo entero y borrar configuración que no
aparece en el formulario. Para evitar ese riesgo, lo editable vive en
archivos chicos y dedicados (`_data/site.yml`, `_data/authors.yml`,
`_data/navigation.yml`) que **solo** contienen lo que se edita — nada que
pueda romper el sitio si se sobrescribe.

Si en el futuro quieres que algo más sea editable desde el panel (ej. los
campos de una publicación académica, o la sección CV), se agrega como una
colección nueva en `admin/config.yml`, apuntando idealmente a su propio
archivo de datos, no a `_config.yml`.

## 7. Secciones activas vs. ocultas

Activo en el menú ahora mismo: Periodismo, Academia, La Siderúrgica, Jardín
digital, Archivo, Blog, Contacto (todas apuntando a URLs reales, algunas
externas al sitio).

Ocultas (archivos siguen ahí, con contenido de ejemplo del theme, no
borrados — se pueden reactivar agregando una entrada en "Secciones del
menú" en el CMS, o descomentando en `_data/navigation.yml`):

| Sección | URL |
|---|---|
| Publications | `/publications/` |
| Talks | `/talks/` |
| Teaching | `/teaching/` |
| Portfolio | `/portfolio/` |
| CV | `/cv/` |
| Guide | `/markdown/` |

## 8. Relación con WordPress (`jcramirezfigueroa.com`)

**Son dos sistemas 100% independientes hoy.** No comparten base de datos,
hosting, ni login. Lo único que se hizo fue copiar manualmente, una vez,
contenido real desde el WordPress hacia este sitio nuevo (nombre, foto,
links).

- El dominio `jcramirezfigueroa.com` está registrado y alojado en
  **WordPress.com** (sitio `jcrfblog.wordpress.com`, activo desde 2016,
  dominio propio conectado).
- Tiene un **blog real y activo** en `jcramirezfigueroa.com/blog/`, con
  decenas de posts (2023 en adelante al menos). Es la fuente más probable
  para una futura migración de contenido real al Blog de este sitio nuevo —
  no se ha tocado ni copiado nada de ahí todavía, eso es una fase aparte.
- Puedes seguir usando WordPress exactamente igual que siempre —
  administrar, publicar, todo — sin que afecte en nada a `jcrf-landing`.
- El día que se decida conectar `jcramirezfigueroa.com` a este sitio nuevo
  (cambiar DNS), el WordPress dejaría de verse en ese dominio (seguiría
  existiendo en `jcrfblog.wordpress.com`, solo que ya no en el dominio
  personalizado). **Eso no se ha hecho.** Es una decisión aparte, que
  requiere confirmación explícita cada vez que se hable de ella — no basta
  con haberlo mencionado una vez.

## 9. Portabilidad / plan de emergencia

Este sitio no depende de GitHub más allá de ser donde vive el repo y donde
corre el build. Si algún día hay que migrarlo:

- El repo se puede clonar/mover a cualquier otro lugar (GitLab, Bitbucket, un
  disco duro) sin perder nada — es git estándar.
- El sitio (Jekyll + este theme) se puede desplegar en cualquier hosting
  estático: Netlify, Cloudflare Pages, Vercel, un servidor propio con
  `jekyll build` + cualquier servidor web. No hay lock-in con GitHub.
- Sveltia CMS tampoco es exclusivo de GitHub Pages — solo necesita que el
  backend sea un repo de git (GitHub, GitLab, etc.).
- Para desarrollar localmente hace falta Ruby + Bundler (`bundle install` y
  `bundle exec jekyll serve`). La Mac tiene Ruby 2.6 (del sistema), que es
  antiguo para las gemas modernas de Jekyll — si se quiere previsualizar
  local en algún momento, probablemente haga falta instalar una versión más
  nueva de Ruby (via rbenv/asdf). Por ahora todo el trabajo se verifica
  contra el build real de GitHub Pages, sin instalar nada local.

## 10. Pendientes conocidos (a la fecha de este documento)

- Falta la línea de bio/descripción de la Portada — la escribe Juan Carlos
  directamente cuando quiera (en `/admin/` o pidiéndomelo).
- El Blog solo tiene 2 posts de prueba, marcados como tales. La migración de
  posts reales desde WordPress es una fase aparte, sin definir aún.
- El menú "Explorar por año / tema" del Blog (columnas de años y tags) ya
  está implementado y funcionando en `/year-archive/` — ver
  `_includes/blog-filter-menu.html`.
- DNS del dominio real: sin tocar, a la espera de decisión explícita.
