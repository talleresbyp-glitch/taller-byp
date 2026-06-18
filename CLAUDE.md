# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a single-file, zero-dependency web application for vehicle repair shop productivity tracking ("Taller Ballesteros y Pazmiño", Quito, Ecuador). The entire application lives in `productividad_byp_v2_firebase.html`. There is no build system, package manager, or compilation step.

**To run:** Open the HTML file directly in a browser. No server required.

## Architecture

The app is a single HTML file (~664 lines) structured in three layers:

1. **CSS** (lines 10–138): CSS custom properties for theming, component classes, responsive breakpoint at 780px.
2. **Configuration** (lines 159–186): Firebase config, `DEFAULTS` object with business rules (rates, commissions, goals, PINs), initial technician list, and pause reasons. All credentials and PINs are hardcoded here.
3. **JavaScript** (lines 188–662): Vanilla ES modules loaded from CDN (Firebase, SheetJS).

### Data Layer (lines 188–233)

Two interchangeable DB backends with the same interface (`get`, `set`, `del`, `sub` for pub-sub):

- `makeLocalDB()` — localStorage, keys prefixed `byp2_`
- `makeCloudDB()` — Firebase Firestore with `onSnapshot` for real-time updates; falls back to local DB on failure

Collections: `settings`, `techs`, `orders`, `jobs`

### Global State (lines 235–237)

Four mutable module-level objects: `settings`, `techs`, `orders`, `jobs` — populated on bootstrap and updated via DB subscriptions.

### Rendering (lines 293–613)

No framework. All UI is generated via `innerHTML`. The `render()` function dispatches based on `session.role`:

- `renderGate()` — role/PIN selection screen
- `renderTech()` — technician interface: start/pause/finish jobs, view active orders
- `renderCoord()` — coordinator interface: create orders, assign jobs to technicians
- `renderDash()` — KPI dashboard: goals, trends, per-technician metrics

A single reusable `modal(html)` function injects content into a shared modal container.

### Key Business Logic (lines 239–276)

- `jobSecs(job)` — computes real elapsed seconds from `job.segments[]` (each segment has `{start, end}`)
- `jobReal(job)` — real hours worked
- `billed(job)` — billable hours (`horasFacturadas`)
- `mou(jobs)` — Machine Occupancy Unit % (real hours / available hours)
- `efficiency(jobs)` — billed / real hours ratio
- `productivity(jobs)` — composite KPI

### Utilities

- `$('selector')` — `document.querySelector` alias
- `uid()` — generates unique IDs
- `num(v)` — safe `parseFloat`
- `money(n)` — formats to `es-EC` currency
- `hrs(secs)` — formats seconds as `Xh Ym`
- `fmtClock(secs)` / `fmtDT(ts)` — clock and datetime formatters

## Data Models

```
settings: { rate, commission, metaIndividual, horasDisponibles,
            metaFacturacionMensual, metaMixMO, metaMarcasExpertas,
            pinCoord, pinDash, logoUrl }

techs:    { id, nombre, pin, activo }

orders:   { id, ot, placa, marca, modelo, anio, cliente,
            ingresoAt, salidaAt, createdAt }

jobs:     { id, orderId, moDesc, tipo, tecnicoId, horasAsignadas,
            horasFacturadas, valorRepuestos, valorServicioExterno,
            estado, segments[], pauseReason, createdAt, finishedAt }
```

`job.tipo` values: `Express` | `Mantenimiento` | `Especializado`
`job.estado` values: `asignado` | `en_curso` | `pausado` | `terminado`

## Conventions

- All UI text is in Spanish; number formatting uses `es-EC` locale.
- Variable and function names are camelCase; UI labels use Spanish.
- DOM manipulation is done exclusively via `innerHTML` and `querySelector`.
- Real-time clock updates run via `setInterval(tick, 1000)`.
- PIN authentication uses an on-screen numeric keypad — no hashing.

## Export (lines 616–640)

Uses SheetJS (CDN) to generate `.xlsx` files with three sheets: detailed jobs, orders summary, and technician performance. CSV export is also available.
