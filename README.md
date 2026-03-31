# Confidential* Route Planner

> **Internal logistics tool built for [Confidential*](https://github.com/dreasy) — a last-mile FMCG distribution startup operating in the Casablanca region.**

[![Live Demo](https://img.shields.io/badge/Live%20Demo-GitHub%20Pages-blue?style=flat-square&logo=github)]([https://dreasy.github.io/delivery-route-planner](https://github.com/dreasya/Delivery-Route-Optimizer/blob/main/delivery-route-planner.html))

---

## The Problem

Confidential* delivers 20–30 FMCG orders per day across Casablanca. Every morning, the ops team had to:

1. Manually scan yesterday's Odoo export to decide which orders to deliver
2. Copy–paste addresses into Google Maps one by one
3. Mentally sequence stops with no optimization
4. Share a rough plan with the delivery agent via WhatsApp

This was taking **45–60 minutes each morning** and producing routes that were often inefficient.

## The Solution

A single-file offline web app that absorbs the entire morning workflow:

- Upload the Odoo Excel export → orders appear on the map
- Select delivery zones (corridors) to include
- Click **Optimize** → NN + 2-opt algorithm generates the best route
- Export a **Route Card** with segmented Google Maps links, ready to share

Planning time dropped from ~50 minutes to **under 5 minutes**.

---

## Features

### Data Import
- Drag-and-drop `.xlsx` upload or paste directly from Excel
- **Custom XLSX parser** — no SheetJS dependency. Implements zip decompression (`inflate`), XML parsing, and shared string resolution from scratch
- **RFC 4180 CSV parser** — handles quoted commas, Arabic text, BOM headers, TSV/CSV auto-detection
- Smart column aliasing: maps `"Matrix Order ID"`, `"Customer/City"`, `"X"`, `"Y"` etc. automatically

### Interactive Map
- **Offline vector basemap** — rendered on HTML5 Canvas from embedded geographic data. Works with no internet connection, no tile server, no API keys
- 15 delivery zones covering Greater Casablanca, all defined as real GPS polygons from order density clustering
- Point-in-polygon sector detection (ray-casting algorithm) assigns each order to its zone automatically
- Pan, zoom, hover tooltips, click-to-focus on any stop

### Route Optimization
- **Nearest Neighbor** seeding — greedy algorithm builds an initial route in O(n²)
- **2-opt improvement** — iteratively reverses route segments until no improvement is found, typically reducing distance by 5–10% over NN alone
- Home-bias on final stops: weights the last 3 stops toward the agent's home address to minimize return trip
- Supports up to 35 stops (van capacity guardrail at 30)

### Order Confirmation Workflow
- Every order has a ✓/✗ toggle — confirmed orders route, deferred ones are skipped
- Corridor cards show confirmed vs deferred counts in real time
- Deferred orders remain visible (struck through) so nothing gets lost

### Route Card Export
- Downloads a standalone `.html` file — printable, shareable via WhatsApp
- **Segmented Google Maps links**: Google Maps allows max 10 waypoints per URL, so routes are automatically split into Part 1 / Part 2 / Part 3, each ending where the next begins
- Each stop row shows its segment badge (P1/P2/P3), individual Maps navigation link, order numbers, and value
- Supports up to 35 stops across 4 links

### Session Metrics
- Every planned route is recorded: date, stops, km, duration estimate, MAD value, planning time
- KPI dashboard: avg km/day, avg stops, avg planning time
- CSV export for analysis

---

## Technical Architecture

```
delivery-route-planner.html          ← entire app in one file (~90KB)
│
├── CSS (dark UI, canvas map, responsive)
├── HTML (splash, sidebar, canvas, panels)
└── JavaScript
    ├── CONFIG
    │   ├── CORRIDORS        15 GPS polygon zones
    │   ├── COL_MAP          Column header aliases
    │   └── HUB / AGENT_HOME Warehouse & agent coords
    │
    ├── PARSERS
    │   ├── parseXLSX()      Zip → inflate → XML → rows
    │   ├── parseCSV()       RFC 4180, TSV/CSV auto-detect
    │   ├── parseDate()      Excel serials, ISO, regional formats
    │   └── mapRow()         Field extraction + coordinate validation
    │
    ├── MAP ENGINE
    │   ├── drawOfflineBasemap()   Canvas vector basemap
    │   ├── drawSectorPolygons()   Corridor overlays + labels
    │   ├── drawRoute()            Optimized path with arrows
    │   └── drawDots()             Order stop markers
    │
    ├── OPTIMIZER
    │   ├── nnWithHome()     Nearest Neighbor with home bias
    │   └── twoOpt()         2-opt improvement, O(n²)/pass
    │
    ├── WORKFLOW
    │   ├── Order confirmation toggles
    │   ├── Corridor selection & filtering
    │   └── Date range filtering
    │
    └── EXPORT
        ├── exportCard()     Route card HTML with Maps links
        ├── buildMapsUrl()   Segmented 10-stop Google Maps URLs
        └── metricsRecord()  localStorage session history
```

---

## Delivery Zones

15 zones covering Greater Casablanca, defined from real order density clustering:

| Zone | Area | Color |
|------|------|-------|
| Zone 1 | Maarif / Zerktouni | 🔴 |
| Zone 2 | Racine / Bourgogne / Anfa | 🔵 |
| Zone 3 | Hay Hassani / Oulfa | 🟣 |
| Zone 4 | Ain Sebaa / Bernoussi | 🟠 |
| Zone 5 | Gauthier / Palmier | 🟢 |
| Zone 6 | Mers Sultan / Habous | 🟡 |
| Zone 7 | Derb Sultan / Sbata | 💚 |
| Zone 8 | Sidi Maarouf | 🔵 |
| Zone 9 | Ain Diab / West Coast | 🟣 |
| Zone 10 | Central South Corridor | 🔴 |
| — | Mohammedia | 🩵 |
| — | Bouskoura | 🟤 |
| — | Nouaceur | ⚫ |
| — | Dar Bouazza | ⚫ |
| — | Settat | 🟡 |

---

## How to Use

### Option A — Live Demo
→ [dreasy.github.io/delivery-route-planner](https://dreasy.github.io/delivery-route-planner)

### Option B — Run Locally
```bash
# No install, no build step, no server needed
git clone https://github.com/yourusername/Confidential*-route-planner
open Confidential*_planner.html   # or just double-click it
```

### Daily Workflow
```
1. Open Confidential*_planner.html
2. Upload Odoo export (.xlsx or .csv)
3. Set date filter → Apply
4. Toggle ✓/✗ on orders (confirm / defer)
5. Select zones to deliver today
6. Click ⚡ Optimize Selected
7. Review route on map
8. Click ↓ Route Card → share with agent
```

---

## Design Decisions

**Why a single HTML file?**
The delivery agent and ops team use different devices. A single `.html` file requires no installation, no app store, no server — it opens in any browser on any device. The route card export is also a self-contained `.html` so it's readable offline.

**Why no dependencies?**
External CDNs (SheetJS, Leaflet, etc.) require internet and introduce version fragility. The XLSX parser and canvas map were written from scratch specifically for this use case — they're smaller and faster than general-purpose libraries for the data shapes we actually use.

**Why 2-opt over more advanced methods?**
At 20–30 stops, 2-opt consistently finds near-optimal routes in under 100ms. OR-Tools or simulated annealing would give marginally better results at the cost of significant complexity and compute. The practical gain at this scale doesn't justify it.

---

## About Confidential*

Confidential* is a last-mile FMCG distribution startup based in Casablanca. As **Product & Ops Lead**, I built this tool to eliminate manual planning and create a feedback loop between route data and ops decisions.

The metrics engine embedded in this tool was the first step toward a data-driven dispatch system — tracking planning time, route efficiency, and order value per zone over time.

---

## Roadmap

- [ ] Multi-day planning view (J+1 / J+2 split)
- [ ] Van capacity auto-split (>30 stops → 2 days)
- [ ] Manual drag-and-drop stop reordering
- [ ] WhatsApp deep link for route card sharing
- [ ] Offline PWA with service worker for map tiles
- [ ] Multi-tenant SaaS version (per-distributor zones)

---

## License

MIT — free to use, adapt, and build on.

---

*Built by [Mohamed Amin OUARGUI](https://github.com/dreasy) · Product & Ops Lead @ Confidential* · Berrechid, Morocco*
