# Contexto del Proyecto: POS Abarrotes

## Objetivo
Sistema de punto de venta local-first para una tiendita de abarrotes familiar en México.
Reemplaza la dependencia de la memoria de 1-2 vendedores para conocer precios,
permitiendo que cualquier persona (familiar o empleado nuevo) pueda cobrar correctamente.

## Contexto del negocio
- Tiendita de abarrotes tradicional (no tipo OXXO/Seven Eleven), sin sistema actual.
- Usuarios principales: personas de 60-70+ años, poco familiarizadas con tecnología.
- Sin internet fijo contratado en el local; el sistema debe operar 100% offline.
- La computadora estará encendida ~8 horas continuas al día.
- Presupuesto ajustado: se busca hardware y desarrollo de bajo costo.

## Alcance del MVP
- Catálogo de productos con precio y código de barras/QR (cuando exista).
- Catálogo manual para productos sin código (venta a granel: huevo, etc.), con
  precio por kg/unidad y teclado numérico para cantidad.
- Pantalla de venta única: escaneo o búsqueda manual → acumula cuenta → calcula cambio.
- Actualización de precios desde un panel de administración.
- Registro de abastecimiento/pedidos (fecha, producto, cantidad recibida).
- Descuento automático de existencias al vender.
- Reportes básicos de ventas.
- Respaldo local automático (con rotación) + sincronización a la nube cuando
  se detecte conexión a internet (no depende de internet para operar).

## Fuera de alcance (por ahora)
- Facturación fiscal.
- Manejo de stock multi-sucursal.
- Impresora de tickets.
- Cajón de dinero.

## Roles
- **Admin** (dueño/familiar responsable): requiere autenticación. Acceso a catálogo,
  precios, reportes, configuración de backup/sync.
- **Cajero**: sin autenticación, acceso directo a la pantalla de venta.

## Decisiones de arquitectura
- **Monorepo** con separación clara backend/frontend (no repos separados).
- **Backend**: Django + Django REST Framework + SQLite, corriendo localmente
  (no en servidor remoto). Se elige API REST aunque sea local para mantener
  la puerta abierta a un futuro escenario multi-terminal en red local (LAN).
- **Frontend**: React + TypeScript (SPA), diseñado para pantalla táctil,
  con botones grandes y flujos simples para usuarios de edad avanzada.
- **Empaquetado de escritorio**: Electron envolviendo el frontend React,
  levantando el backend Django como proceso hijo al iniciar la app.
  Modo kiosko (pantalla completa) para la pantalla de venta.
  Alternativa más ligera a evaluar si el hardware es limitado: `pywebview`.
- **Backup/sync**: backup local automático con rotación (independiente de
  internet) + servicio de sincronización a la nube que solo actúa cuando
  detecta conexión disponible.

## Códigos de barras
Los productos embolsados de fábrica (papitas, refrescos, galletas, jabones,
cereales) ya traen código EAN-13/UPC único por presentación, sin importar el
productor — se pueden usar tal cual, vinculándolos al catálogo por código.

## Hardware estimado (sin impresora ni cajón)
| Componente | Rango (MXN) |
|---|---|
| Mini PC | $2,500 – $5,000 |
| Monitor táctil 13-15" | $2,500 – $4,500 |
| Lector de código de barras USB | $300 – $900 |
| Regulador/no-break | $400 – $800 |

**Total estimado: ~$5,700 – $11,200 MXN**

## Plan por fases
1. **Núcleo de venta** (3-5 semanas): catálogo (por código y manual), pantalla
   de venta con cálculo de cambio, roles admin/cajero.
2. **Respaldo** (1-2 semanas): backup local con rotación + sync a la nube.
3. **Inventario y pedidos** (2-3 semanas): descuento de stock, registro de
   abastecimiento, reportes básicos de ventas.

## Estructura de proyecto propuesta (a definir en detalle)
```
pos-abarrotes/
  backend/     # Django + DRF + SQLite
  frontend/    # React + TypeScript (Vite)
  desktop/     # Electron (o pywebview) — empaquetado de escritorio
  docs/        # Documentación del proyecto, CLAUDE.md
```

## Perfil del desarrollador
- Ingeniero de sistemas / desarrollador junior.
- Stack conocido: Django/DRF, React/TypeScript, Docker, Git, REST APIs, PHP.
- Prefiere entender el "por qué" de las decisiones técnicas, no solo el código.
- Valora soluciones mantenibles y alineadas a la estructura del proyecto,
  con explicaciones claras del razonamiento.
