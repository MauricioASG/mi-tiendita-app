# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md — POS Abarrotes

Este archivo da contexto a Claude Code sobre el proyecto. Léelo antes de proponer
diseños, modelos o código nuevo.

## Qué es este proyecto
Sistema de punto de venta (POS) local-first para una tiendita de abarrotes
familiar en México. Sin internet fijo en el local. Usuarios principales:
personas de 60-70+ años, poco familiarizadas con tecnología.

Contexto completo del negocio y decisiones: ver `docs/contexto-proyecto-pos-abarrotes.md`.

## Stack
- **Backend**: Django 6.1 + Django REST Framework 3.18 + SQLite, Python 3.12 (`backend/`)
- **Frontend**: React + TypeScript, Vite (`frontend/`) — carpeta vacía todavía
- **Desktop**: Electron, envolviendo el frontend y levantando el backend como
  proceso hijo (`desktop/`) — carpeta vacía todavía; se configura al final,
  después de tener el flujo de venta funcionando en modo desarrollo normal.

## Estructura de carpetas
```
backend/     Django + DRF + SQLite
  config/    Proyecto Django (settings, urls, wsgi/asgi)
  catalogo/  App de catálogo de productos
  venv/      Entorno virtual (ignorado por git)
frontend/    React + TypeScript (Vite)
desktop/     Empaquetado Electron (se aborda al final)
docs/        Documentación del proyecto
```

## Comandos de desarrollo

Todos los comandos del backend se corren desde `backend/`, con el venv activado.
El shell principal de este entorno es PowerShell (Windows).

```powershell
# Activar el entorno virtual (PowerShell)
.\venv\Scripts\Activate.ps1

# Instalar/actualizar dependencias
pip install -r requirements.txt

# Congelar dependencias después de instalar algo nuevo
pip freeze > requirements.txt

# Levantar el servidor de desarrollo (http://127.0.0.1:8000)
python manage.py runserver

# Migraciones
python manage.py makemigrations
python manage.py migrate

# Crear usuario admin (rol Admin del POS)
python manage.py createsuperuser

# Tests: toda la suite
python manage.py test

# Tests: solo una app / una clase / un método
python manage.py test catalogo
python manage.py test catalogo.tests.NombreDelTestCase
python manage.py test catalogo.tests.NombreDelTestCase.test_nombre_del_metodo

# Shell de Django (útil para inspeccionar el catálogo)
python manage.py shell
```

Con Git Bash en vez de PowerShell, el activate es `source venv/Scripts/activate`.

## Arquitectura

**Por qué una API REST para una app local.** El backend Django corre en la misma
máquina que el frontend, no en un servidor remoto. Aun así se expone como API REST
en lugar de renderizar plantillas de Django, para dejar la puerta abierta a un
escenario futuro de varias terminales en red local (LAN) sin reescribir la capa de
datos. Todo lo que agregues al backend debe ser accesible vía HTTP/JSON, no solo
desde plantillas.

**Cómo se conectan las piezas.** Electron arranca el proceso de Django como hijo y
sirve el bundle de React; React habla con Django por HTTP a localhost. Esto implica
que el backend nunca asume internet y que los errores de red se tratan como fallas
locales, no como "sin conexión".

**Estado real del backend.** El proyecto Django y la app `catalogo` ya existen y
`catalogo` está registrada en `INSTALLED_APPS`, pero `models.py`, `views.py` y
`admin.py` siguen siendo los stubs vacíos que generó `startapp`. `config/urls.py`
solo enruta `/admin/`; todavía no hay router de DRF ni endpoints. El siguiente paso
del backend es el modelo de Producto.

**Settings pendientes de ajustar.** `config/settings.py` sigue con los valores por
defecto de `startproject`: `LANGUAGE_CODE = 'en-us'`, `TIME_ZONE = 'UTC'`,
`SECRET_KEY` embebida en el archivo y `DEBUG = True`. Para un POS en México hay que
cambiar idioma y zona horaria (`es-mx` / `America/Mexico_City`) antes de guardar
fechas de venta, porque los reportes por día dependen de eso. La `SECRET_KEY` y
`DEBUG` deben salir del código antes de empaquetar con Electron.

**Base de datos.** `backend/db.sqlite3` está en `.gitignore` (por el patrón
`*.sqlite3`) — nunca se versiona. El esquema viaja en las migraciones.

## Reglas de negocio clave
- **Roles**: Admin requiere autenticación (catálogo, precios, reportes,
  configuración). Cajero NO requiere autenticación — acceso directo a la
  pantalla de venta.
- **Catálogo por código de barras**: productos de fábrica ya traen EAN-13/UPC
  único por presentación — se guarda tal cual, sin lógica especial de marca.
- **Catálogo manual (granel)**: productos sin código (ej. huevo por kilo)
  necesitan búsqueda manual con botones grandes + teclado numérico para
  cantidad, no dependen del escáner.
- **Cálculo de cambio**: la pantalla de venta debe pedir "¿con cuánto pagó?"
  y mostrar el cambio en números grandes.
- **Sin facturación fiscal, sin impresora de tickets, sin cajón de dinero.**
- **Backup**: respaldo local automático con rotación, independiente de
  internet. Sincronización a la nube solo cuando se detecta conexión — nunca
  bloqueante para operar.

## Prioridades de UX
- Pantalla táctil, botones grandes, textos mínimos.
- Flujos de una sola pantalla siempre que sea posible.
- Nada de menús anidados ni configuraciones visibles en el flujo de venta
  diario — eso vive aparte, en el panel de admin.

## Convenciones de código
- El código y la documentación del proyecto están en español (nombres de apps,
  modelos y carpetas incluidos: `catalogo`, no `catalog`). Mantén esa convención.
- Explica el razonamiento detrás de cada decisión no trivial (el desarrollador
  es junior y está aprendiendo activamente).
- Al proponer cambios de código, indica ubicación exacta ("antes de esta
  línea", "reemplaza este bloque").
- Si una regla de negocio no está clara en este documento o en
  `docs/contexto-proyecto-pos-abarrotes.md`, pregunta antes de asumir.
- Prioriza soluciones simples y mantenibles sobre soluciones "inteligentes"
  difíciles de seguir — el código lo va a mantener una sola persona.

## Estado actual
- [x] Repo creado, estructura base de carpetas
- [x] Backend Django + DRF inicializado (proyecto `config`, app `catalogo` registrada)
- [ ] Frontend React + TS inicializado
- [ ] Modelo de Producto (catálogo por código + catálogo manual)
- [ ] Endpoints DRF del catálogo (router en `config/urls.py`)
- [ ] Pantalla de venta con cálculo de cambio
- [ ] Roles admin/cajero
- [ ] Backup local + sync a la nube
- [ ] Empaquetado con Electron

## Fases del proyecto
1. Núcleo de venta (catálogo, pantalla de venta, roles)
2. Respaldo (backup local + sync)
3. Inventario y pedidos (stock, abastecimiento, reportes)
