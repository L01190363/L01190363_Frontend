# L01190363_Frontend

# Frontend — Directorio de Materias y Docentes

Interfaz web que consume la API del directorio. Permite **listar, crear, editar y
eliminar** materias y docentes desde el navegador.

Proyecto Integrador del curso *CloudCoder: Codespaces, GitHub y Copilot en la nube*.

Este README es una guía paso a paso para montar el frontend desde cero en GitHub
Codespaces, explicando **qué hace y por qué** cada paso.

---

## ¿Cómo funciona? (la idea en una línea)

```
index.html (HTML+CSS+JS)  --fetch-->  Flask (puerto 5000)  -->  MySQL
                          <--JSON---
```

El frontend **no tiene base de datos ni lógica de negocio**: solo pide datos al
backend con `fetch` y dibuja la respuesta en pantalla. Por eso vive en un repo
aparte y es tan ligero.

---

## Los archivos del proyecto

El repositorio tiene esta estructura:

```
frontend-directorio/
├── .devcontainer/
│   └── devcontainer.json   -> Entorno del Codespace (Python para servir)
├── index.html              -> La aplicación completa (HTML + CSS + JS)
└── README.md
```

### 1. `index.html` — la aplicación

Un solo archivo con las tres capas juntas:

- **HTML** → la *estructura*: las pestañas, las tablas y los formularios.
- **CSS** (dentro de `<style>`) → la *apariencia*: colores, tipografía, espaciado.
- **JavaScript** (dentro de `<script>`) → la *lógica*: los `fetch` al backend y el
  dibujado de las tablas.

Dos piezas clave del JavaScript:

- **`URL_BASE`**: la única línea que debes configurar. Apunta al backend.
- **`api()`**: un helper que centraliza *todas* las llamadas HTTP. Agrega el header
  JSON, maneja el `204 No Content` de los DELETE y muestra los errores del backend.

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Directorio de Materias y Docentes</title>
  <style>
    :root {
      --navy: #15315c;
      --navy-700: #1b3a6b;
      --bg: #eef1f6;
      --card: #ffffff;
      --border: #d8dee8;
      --text: #1f2733;
      --muted: #5b6675;
      --accent: #c2410c;            /* ámbar quemado, usado con moderación */
      --ok-bg: #e7f4ec; --ok-fg: #1c6b3f;
      --err-bg: #fdeaea; --err-fg: #a01c1c;
      --mono: ui-monospace, SFMono-Regular, "SF Mono", Menlo, Consolas, monospace;
      --sans: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    }
    * { box-sizing: border-box; }
    body {
      margin: 0; font-family: var(--sans); color: var(--text);
      background: var(--bg); line-height: 1.5;
    }
    header.top { background: var(--navy); color: #fff; padding: 20px 24px; }
    header.top h1 { margin: 0; font-size: 1.3rem; letter-spacing: .2px; }
    header.top p { margin: 4px 0 0; color: #b9c6dd; font-size: .85rem; }
    .container { max-width: 920px; margin: 0 auto; padding: 24px; }

    /* Pestañas */
    nav.tabs { display: flex; gap: 4px; border-bottom: 2px solid var(--border); margin-bottom: 24px; }
    nav.tabs button {
      background: none; border: none; padding: 12px 18px; font-size: .95rem;
      font-family: inherit; color: var(--muted); cursor: pointer;
      border-bottom: 2px solid transparent; margin-bottom: -2px;
    }
    nav.tabs button.active { color: var(--navy); border-bottom-color: var(--accent); font-weight: 600; }

    /* Tarjetas */
    .card { background: var(--card); border: 1px solid var(--border); border-radius: 10px; padding: 20px; margin-bottom: 20px; }
    .card h2 { margin: 0 0 14px; font-size: 1.05rem; color: var(--navy); }

    /* Formularios */
    form.fila { display: flex; flex-wrap: wrap; gap: 10px; align-items: flex-end; }
    .campo { display: flex; flex-direction: column; gap: 4px; flex: 1; min-width: 130px; }
    .campo label { font-size: .74rem; color: var(--muted); text-transform: uppercase; letter-spacing: .4px; }
    input, select {
      font-family: inherit; font-size: .95rem; padding: 9px 11px;
      border: 1px solid var(--border); border-radius: 7px; background: #fff; color: var(--text);
    }
    input:focus, select:focus { outline: 2px solid var(--navy-700); outline-offset: 1px; border-color: var(--navy-700); }
    button.btn {
      font-family: inherit; font-size: .9rem; padding: 9px 16px; border-radius: 7px;
      border: 1px solid var(--navy); background: var(--navy); color: #fff; cursor: pointer;
    }
    button.btn:hover { background: var(--navy-700); }
    button.btn.ghost { background: #fff; color: var(--navy); }
    button.btn.ghost:hover { background: #f0f3f8; }

    /* Tablas */
    .tabla-wrap { overflow-x: auto; }
    table { width: 100%; border-collapse: collapse; font-size: .92rem; }
    th, td { text-align: left; padding: 10px 12px; border-bottom: 1px solid var(--border); }
    th { font-size: .72rem; text-transform: uppercase; letter-spacing: .4px; color: var(--muted); }
    td.code { font-family: var(--mono); }
    .acciones { display: flex; gap: 6px; }
    button.mini {
      font-family: inherit; font-size: .8rem; padding: 5px 10px; border-radius: 6px; cursor: pointer;
      border: 1px solid var(--border); background: #fff; color: var(--text);
    }
    button.mini:hover { background: #f0f3f8; }
    button.mini.danger { color: var(--err-fg); border-color: #e7b8b8; }
    button.mini.danger:hover { background: var(--err-bg); }
    .vacio { color: var(--muted); font-style: italic; padding: 14px 12px; }

    /* Mensajes */
    #mensaje { padding: 11px 14px; border-radius: 8px; margin-bottom: 16px; font-size: .9rem; display: none; }
    #mensaje.ok { background: var(--ok-bg); color: var(--ok-fg); display: block; }
    #mensaje.error { background: var(--err-bg); color: var(--err-fg); display: block; }

    button, input, select { transition: background .15s, border-color .15s; }
    @media (prefers-reduced-motion: reduce) { * { transition: none !important; } }
  </style>
</head>
<body>
  <header class="top">
    <h1>Directorio de Materias y Docentes</h1>
    <p>Proyecto Integrador · CloudCoder</p>
  </header>

  <div class="container">
    <div id="mensaje"></div>

    <nav class="tabs">
      <button id="tab-materias" class="active" onclick="mostrarSeccion('materias')">Materias</button>
      <button id="tab-docentes" onclick="mostrarSeccion('docentes')">Docentes</button>
    </nav>

    <!-- ===================== SECCIÓN MATERIAS ===================== -->
    <section id="seccion-materias">
      <div class="card">
        <h2 id="titulo-form-materia">Agregar materia</h2>
        <form class="fila" onsubmit="guardarMateria(event)">
          <div class="campo">
            <label for="m-clave">Clave</label>
            <input id="m-clave" required placeholder="TC1028">
          </div>
          <div class="campo" style="flex:2">
            <label for="m-nombre">Nombre</label>
            <input id="m-nombre" required placeholder="Programación en Python">
          </div>
          <div class="campo" style="flex:.6">
            <label for="m-creditos">Créditos</label>
            <input id="m-creditos" type="number" min="0" placeholder="5">
          </div>
          <button class="btn" type="submit" id="btn-materia">Agregar</button>
          <button class="btn ghost" type="button" id="btn-cancelar-materia"
                  style="display:none" onclick="cancelarEdicionMateria()">Cancelar</button>
        </form>
      </div>

      <div class="card">
        <h2>Materias registradas</h2>
        <div class="tabla-wrap">
          <table>
            <thead>
              <tr><th>ID</th><th>Clave</th><th>Nombre</th><th>Créditos</th><th>Acciones</th></tr>
            </thead>
            <tbody id="tbody-materias"></tbody>
          </table>
        </div>
      </div>
    </section>

    <!-- ===================== SECCIÓN DOCENTES ===================== -->
    <section id="seccion-docentes" style="display:none">
      <div class="card">
        <h2>Agregar docente</h2>
        <form class="fila" onsubmit="crearDocente(event)">
          <div class="campo">
            <label for="d-nombre">Nombre</label>
            <input id="d-nombre" required placeholder="Ana Torres">
          </div>
          <div class="campo" style="flex:1.4">
            <label for="d-email">Email</label>
            <input id="d-email" type="email" required placeholder="ana.torres@tec.mx">
          </div>
          <div class="campo">
            <label for="d-materia">Materia</label>
            <select id="d-materia"></select>
          </div>
          <button class="btn" type="submit">Agregar</button>
        </form>
      </div>

      <div class="card">
        <h2>Docentes registrados</h2>
        <div class="tabla-wrap">
          <table>
            <thead>
              <tr><th>ID</th><th>Nombre</th><th>Email</th><th>Materia</th><th>Acciones</th></tr>
            </thead>
            <tbody id="tbody-docentes"></tbody>
          </table>
        </div>
      </div>
    </section>
  </div>

  <script>
  /* ============================================================
     CONFIGURACIÓN — lo ÚNICO que debes cambiar.
     Pega aquí la URL pública del puerto 5000 de tu Codespace.
     Ejemplo: https://mi-codespace-5000.app.github.dev
     (el .replace quita la diagonal final por si la pegas con "/")
     ============================================================ */
  const URL_BASE = "https://TU-URL-PUBLICA-DEL-PUERTO-5000".replace(/\/$/, "");

  /* ------------------------------------------------------------
     Helper único para todas las llamadas HTTP.
       - Agrega el header JSON.
       - Maneja el 204 No Content (DELETE) sin leer cuerpo.
       - Si el backend responde error, lanza su mensaje.
     ------------------------------------------------------------ */
  async function api(ruta, opciones = {}) {
    const resp = await fetch(URL_BASE + ruta, {
      headers: { "Content-Type": "application/json" },
      ...opciones,
    });
    if (resp.status === 204) return null;          // DELETE: sin cuerpo
    const data = await resp.json();
    if (!resp.ok) throw new Error(data.error || "Error en la petición");
    return data;
  }

  /* Muestra un aviso de éxito o error arriba de la página. */
  function mostrarMensaje(texto, tipo = "ok") {
    const el = document.getElementById("mensaje");
    el.textContent = texto;
    el.className = tipo;
    setTimeout(() => { el.className = ""; el.textContent = ""; }, 4000);
  }

  /* Alterna entre las secciones Materias y Docentes. */
  function mostrarSeccion(nombre) {
    document.getElementById("seccion-materias").style.display = nombre === "materias" ? "" : "none";
    document.getElementById("seccion-docentes").style.display = nombre === "docentes" ? "" : "none";
    document.getElementById("tab-materias").classList.toggle("active", nombre === "materias");
    document.getElementById("tab-docentes").classList.toggle("active", nombre === "docentes");
  }

  /* ============================================================
     MATERIAS
     ============================================================ */
  let materiasCache = [];     // se reusa para el <select> de docentes
  let materiaEditId = null;   // null = creando ; un id = editando

  async function cargarMaterias() {
    try {
      materiasCache = await api("/materias");
      const filas = materiasCache.map(m => `
        <tr>
          <td class="code">${m.id}</td>
          <td class="code">${m.clave}</td>
          <td>${m.nombre}</td>
          <td>${m.creditos ?? "—"}</td>
          <td><div class="acciones">
            <button class="mini" onclick="editarMateria(${m.id})">Editar</button>
            <button class="mini danger" onclick="eliminarMateria(${m.id})">Eliminar</button>
          </div></td>
        </tr>`).join("");
      document.getElementById("tbody-materias").innerHTML =
        filas || `<tr><td colspan="5" class="vacio">No hay materias todavía.</td></tr>`;
      actualizarSelectMaterias();
    } catch (e) {
      mostrarMensaje(e.message, "error");
    }
  }

  /* Crea (POST) o actualiza (PUT) según materiaEditId. */
  async function guardarMateria(evento) {
    evento.preventDefault();
    const cuerpo = {
      clave:    document.getElementById("m-clave").value.trim(),
      nombre:   document.getElementById("m-nombre").value.trim(),
      creditos: parseInt(document.getElementById("m-creditos").value) || null,
    };
    try {
      if (materiaEditId === null) {
        await api("/materias", { method: "POST", body: JSON.stringify(cuerpo) });
        mostrarMensaje("Materia agregada.");
      } else {
        await api(`/materias/${materiaEditId}`, { method: "PUT", body: JSON.stringify(cuerpo) });
        mostrarMensaje("Materia actualizada.");
        cancelarEdicionMateria();
      }
      evento.target.reset();
      cargarMaterias();
    } catch (e) {
      mostrarMensaje(e.message, "error");
    }
  }

  /* Pasa la materia al formulario para editarla. */
  function editarMateria(id) {
    const m = materiasCache.find(x => x.id === id);
    if (!m) return;
    document.getElementById("m-clave").value    = m.clave;
    document.getElementById("m-nombre").value   = m.nombre;
    document.getElementById("m-creditos").value = m.creditos ?? "";
    materiaEditId = id;
    document.getElementById("titulo-form-materia").textContent = `Editando materia #${id}`;
    document.getElementById("btn-materia").textContent = "Guardar cambios";
    document.getElementById("btn-cancelar-materia").style.display = "";
    window.scrollTo({ top: 0, behavior: "smooth" });
  }

  function cancelarEdicionMateria() {
    materiaEditId = null;
    document.querySelector("#seccion-materias form").reset();
    document.getElementById("titulo-form-materia").textContent = "Agregar materia";
    document.getElementById("btn-materia").textContent = "Agregar";
    document.getElementById("btn-cancelar-materia").style.display = "none";
  }

  async function eliminarMateria(id) {
    if (!confirm(`¿Eliminar la materia #${id}?`)) return;
    try {
      await api(`/materias/${id}`, { method: "DELETE" });
      mostrarMensaje("Materia eliminada.");
      cargarMaterias();
    } catch (e) {
      mostrarMensaje(e.message, "error");   // aquí cae el 409 si tiene docentes
    }
  }

  /* ============================================================
     DOCENTES
     ============================================================ */
  async function cargarDocentes() {
    try {
      const docentes = await api("/docentes");
      const filas = docentes.map(d => `
        <tr>
          <td class="code">${d.id}</td>
          <td>${d.nombre}</td>
          <td class="code">${d.email}</td>
          <td>${d.materia_nombre ?? "—"}</td>
          <td><div class="acciones">
            <button class="mini danger" onclick="eliminarDocente(${d.id})">Eliminar</button>
          </div></td>
        </tr>`).join("");
      document.getElementById("tbody-docentes").innerHTML =
        filas || `<tr><td colspan="5" class="vacio">No hay docentes todavía.</td></tr>`;
    } catch (e) {
      mostrarMensaje(e.message, "error");
    }
  }

  /* Rellena el <select> de materias del formulario de docentes. */
  function actualizarSelectMaterias() {
    const select = document.getElementById("d-materia");
    select.innerHTML = `<option value="">— Sin materia —</option>` +
      materiasCache.map(m => `<option value="${m.id}">${m.clave} · ${m.nombre}</option>`).join("");
  }

  async function crearDocente(evento) {
    evento.preventDefault();
    const materiaSel = document.getElementById("d-materia").value;
    const cuerpo = {
      nombre:     document.getElementById("d-nombre").value.trim(),
      email:      document.getElementById("d-email").value.trim(),
      materia_id: materiaSel ? parseInt(materiaSel) : null,
    };
    try {
      await api("/docentes", { method: "POST", body: JSON.stringify(cuerpo) });
      mostrarMensaje("Docente agregado.");
      evento.target.reset();
      cargarDocentes();
    } catch (e) {
      mostrarMensaje(e.message, "error");
    }
  }

  async function eliminarDocente(id) {
    if (!confirm(`¿Eliminar al docente #${id}?`)) return;
    try {
      await api(`/docentes/${id}`, { method: "DELETE" });
      mostrarMensaje("Docente eliminado.");
      cargarDocentes();
    } catch (e) {
      mostrarMensaje(e.message, "error");
    }
  }

  /* ============================================================
     ARRANQUE — cargamos ambas tablas al abrir la página.
     ============================================================ */
  cargarMaterias();
  cargarDocentes();
  </script>
</body>
</html>
```

### 2. `.devcontainer/devcontainer.json` — el entorno

El más sencillo del curso. Solo carga una imagen con Python (para el servidor
estático) y reenvía el puerto 8080. **No tiene `postCreateCommand`** porque el
frontend no instala dependencias — esa es la diferencia con el del backend.

```jsonc
{
  // Configuración del Codespace para el frontend HTML + JS.
  // Es el más sencillo del curso: solo necesita Python para servir
  // los archivos estáticos con "python -m http.server".
  "name": "Frontend HTML+JS",
  "image": "mcr.microsoft.com/devcontainers/python:3.12",

  // Reenvía el puerto del servidor estático.
  "forwardPorts": [8080],
  "portsAttributes": {
    "8080": { "label": "Frontend estático" }
  },

  "customizations": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "ritwickdey.LiveServer"
      ]
    }
  }
}
```

---

## Puesta en marcha — paso a paso

### Paso 1 · Sube los archivos al repositorio

Sube `index.html`, `README.md` y `.devcontainer/devcontainer.json` a tu repo
`frontend-directorio` en GitHub.

> **Qué hace:** deja el repo listo para abrir un Codespace con todo lo necesario.

### Paso 2 · Crea el Codespace

En el repositorio: **Code -> Codespaces -> Create codespace on main**.

> **Qué pasa aquí:** el `devcontainer.json` construye un entorno con Python listo
> para servir la página. Como no hay dependencias, la construcción es muy rápida.

### Paso 3 · Configura la URL del backend

Abre `index.html` y localiza la constante `URL_BASE` al inicio del `<script>`.
Reemplaza el placeholder por la **URL pública del puerto 5000** de tu backend:

```js
const URL_BASE = "https://tu-codespace-5000.app.github.dev";
```

> **Por qué es el paso crítico:** el frontend no sabe dónde está el backend hasta
> que se lo dices aquí. Es el puente entre los dos repos. Si esta URL está mal, las
> tablas saldrán vacías o con error.

### Paso 4 · Sirve el frontend

```bash
python -m http.server 8080
```

> **Qué hace:** levanta un servidor web estático que entrega el `index.html` en el
> puerto 8080. La terminal queda ocupada con el servidor (es normal).

### Paso 5 · Abre el puerto 8080

En la pestaña **Ports**, abre el **puerto 8080** (clic en el ícono del globo o en
la URL).

> **Qué hace:** te da el enlace para ver la aplicación en el navegador.

### Paso 6 · Prueba la aplicación

Al cargar la página deberías ver las materias y los docentes que ya están en la
base de datos.

> **Qué confirma:** si las tablas se llenan, el frontend, el backend y la base de
> datos están conectados de extremo a extremo.

---

## Requisito clave: el backend debe estar vivo y público

El frontend (8080) y el backend (5000) corren en **dos Codespaces distintos** al
mismo tiempo. Para que la app funcione:

1. El backend debe estar **corriendo** (`python ws_directorio.py`).
2. El puerto **5000 del backend** debe estar en **Public**.

> Si el puerto 5000 está en *Private*, el navegador bloquea las respuestas aunque
> el código sea correcto. Es la causa #1 de "no carga nada".

---

## Cómo usar la aplicación

| Pestaña      | Acciones disponibles                                             |
|--------------|------------------------------------------------------------------|
| **Materias** | Listar · Agregar · **Editar** (carga la fila al formulario) · Eliminar |
| **Docentes** | Listar (con su materia) · Agregar (con menú de materias) · Eliminar    |

- Los avisos de éxito o error aparecen en la franja superior.
- Al intentar borrar una materia con docentes asignados, verás el mensaje de error
  `409` del backend — la integridad referencial en acción.

---

## Detalles técnicos

- **Helper `api()`:** un solo punto para todas las llamadas HTTP. Centralizar aquí
  el manejo de errores y del `204` evita repetir código en cada operación.
- **`URL_BASE` como único punto de configuración:** cambiar de backend es una sola
  línea.
- **Tipografía monoespaciada** para claves, IDs y correos: hace que los códigos de
  materia (como `TC1028`) se lean como lo que son, códigos.
