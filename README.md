# L01190363_Frontend
# Frontend — Directorio de Materias y Docentes

Interfaz web que consume la API del directorio. Permite **listar, crear, editar y
eliminar** materias y docentes desde el navegador.

Proyecto Integrador del curso *CloudCoder: Codespaces, GitHub y Copilot en la nube*.

> **README preliminar.** Documenta el arranque básico del repositorio.
> Se ampliará con la guía paso a paso una vez integrado el frontend.

---

## ¿Qué es este repositorio?

Es la **capa de presentación** del proyecto. No tiene base de datos ni lógica de
negocio: solo pide datos al backend y los muestra. La separación es intencional:

```
Este repo (frontend)             Repo backend
┌───────────────────┐  fetch   ┌──────────────────────┐
│  index.html       │ ───────► │  Flask (puerto 5000) │ ──► MySQL
│ (HTML + CSS + JS) │ ◄─────── │  responde en JSON    │
└───────────────────┘  JSON    └──────────────────────┘
```

## Opción elegida: HTML + JavaScript puro

De las dos opciones del proyecto, este equipo eligió **HTML + JS puro** (Opción A)
por ser la más ligera: no instala dependencias, no requiere build y se sirve con el
servidor estático que ya trae Python.

## Stack

- **HTML + CSS + JavaScript** (sin frameworks)
- **`fetch`** para las llamadas HTTP al backend
- **Servidor estático:** `python -m http.server`
- **Entorno:** GitHub Codespaces + `devcontainer.json`

## Estructura del repositorio

```
frontend-directorio/
├── .devcontainer/
│   └── devcontainer.json   -> Entorno del Codespace (Python para el servidor)
├── index.html              -> La aplicación completa (HTML + CSS + JS)
└── README.md
```

---

## Puesta en marcha rápida

### 1. Configurar la URL del backend

Abre `index.html` y localiza la constante `URL_BASE` al inicio del `<script>`.
Pega ahí la **URL pública del puerto 5000** de tu backend:

```js
const URL_BASE = "https://tu-codespace-5000.app.github.dev";
```

> **Importante:** el puerto 5000 del backend debe estar en **Public**. Si está en
> Private, el navegador no podrá leer las respuestas aunque el código sea correcto.

### 2. Servir el frontend

```bash
python -m http.server 8080
```

### 3. Abrir la aplicación

En la pestaña **Ports**, abre el **puerto 8080**. La página carga las materias y
los docentes automáticamente.

---

## Pendientes (se completarán en los siguientes pasos)

- [ ] `index.html` integrado y probado contra el backend
- [ ] `.devcontainer/devcontainer.json` del frontend
- [ ] Guía paso a paso ampliada en este README