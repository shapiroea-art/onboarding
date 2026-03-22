# Site Planner — Build Plan

## What We're Building (Simple Terms)

A digital "camera placement tool" — think Google Maps meets a drawing canvas. The user loads a floor plan or satellite map, drags cameras onto it, and the tool automatically draws colored cones showing exactly what each camera can see (and what it misses). As cameras are added, a live shopping list (BOM) builds itself on the side with prices and storage estimates. The final output is a professional PDF report the user can send to a client.

It is equivalent to tools like IPVM's Camera Calculator or Genetec Site Designer, but built into Lumana.

> **Scope decision (v1):** Phase 1 and Phase 2 focus exclusively on **video cameras**. Access control devices (door controllers, readers) and sensors (air quality, intercoms) will be added in a future phase once the camera placement workflow is solid.

---

## I. Recommended Tech Stack

### For the Interactive Prototype (this repo's style — standalone HTML)
| Layer | Choice | Why |
|---|---|---|
| Canvas / drawing | **Konva.js** (CDN) | Best 2D canvas lib for draggable, interactive objects. Handles FoV cones, walls, snap-to-grid natively. |
| Maps | **Leaflet.js** (CDN) + OpenStreetMap | Free, no API key needed for prototype. Swap to Google Maps API in production. |
| 3D preview | **Three.js** (CDN) | Industry standard WebGL. Lightweight enough for an in-browser preview pane. |
| PDF export | **jsPDF + html2canvas** (CDN) | Capture canvas + BOM table into a shareable PDF. |
| State | Vanilla JS (plain objects + event listeners) | Sufficient for prototype scope. |

### For Production (engineering handoff recommendation)
| Layer | Choice |
|---|---|
| Framework | **React 18** + TypeScript |
| Canvas | **Konva.js** via `react-konva` |
| Maps | **Google Maps JS API** + `@react-google-maps/api` |
| 3D | **Three.js** via `@react-three/fiber` + `@react-three/drei` |
| State | **Zustand** (lightweight, single store) |
| PDF | **@react-pdf/renderer** or Puppeteer (server-side) |
| File parsing | **pdf.js** (floor plan PDF upload), **dxf-parser** (CAD/DWG) |
| Backend sync | REST API calls to existing Lumana backend |

---

## II. File Structure

```
Lumana-Desktop/
├── site-planner-prototype.html          ← Main prototype file (Phase 1 ✅ + Phase 2 ✅ built)
│
├── Knowledge/
│   ├── SitePlanner-Plan.md              ← This file
│   └── SitePlanner-Spec.md             ← Future: detailed UX spec
│
└── assets/
    └── site-planner/
        ├── camera-dome.svg
        ├── camera-bullet.svg
        ├── camera-fisheye.svg
        └── camera-multisensor.svg
```

---

## III. Design Considerations

### Visual Language
- **Match existing Lumana UI**: dark sidebar, white canvas area, purple accent (`#7c3aed`).
- **Canvas background**: dark satellite-style (`#1c2333`) with dot grid pattern.
- **Camera icons**: small colored circle markers on canvas with camera-type symbol.
- **FoV cones**: single gradient cone using the camera's brand color — **no multi-color DORI zones** (no yellow/green/purple). Instead: one cone with three opacity layers of the same hue. Detection (outer) ~15% opacity, Recognition (middle) ~30% opacity, Identification (inner, near camera) ~50% opacity. Creates a depth gradient like the IPVM reference — richer near the camera, fading toward the edge. Floor plan stays readable underneath.
- **Walls/obstructions**: thick lines; red solid / blue dashed (glass) / amber dotted (fence).

### Layout (3-panel split)
```
┌─────────────────────────────────────────────────────┐
│  Toolbar (top bar): tools, zoom, undo, export       │
├──────────┬──────────────────────────┬───────────────┤
│          │                          │               │
│ Camera   │   Map Canvas (center)    │  Properties   │
│ Catalog  │   [Konva stage]          │  Panel        │
│ (left)   │                          │  + BOM        │
│          │                          │               │
└──────────┴──────────────────────────┴───────────────┘
│  Status bar: camera count, total storage, bandwidth │
└─────────────────────────────────────────────────────┘
```

### Camera Selection UX — Hybrid Approach (decision)

With 500+ camera models across 20+ brands, neither a pure sidebar list nor a modal-only flow works well:

| Option | Problem |
|---|---|
| Sidebar list only | 500 models can't fit in 240px — unusable |
| Modal only | Extra click for every placement, disrupts drag-drop flow |
| **Hybrid (chosen)** | Fast for common types, full power when precision needed |

**Chosen design:**
- **Left panel** shows 5 form-factor quick-pick tiles: Dome / Bullet / Fisheye / PTZ / Multisensor. Dragging any tile places a generic camera of that form factor instantly.
- **"Browse Models →" button** at panel top opens a full **Camera Model Picker modal** (IPVM-style) for precise brand/model selection.
- The modal has: search bar, form factor filter chips, resolution + IR range sliders, NDAA/EOL toggles, a 3-column layout (Brands | Models | Info panel), and a "Place Camera" CTA.
- Selecting a model in the modal pre-configures the camera's FoV angle, detection ranges, and specs before placing it on canvas.

### Interaction Principles
1. **Click to select** a camera → right panel shows its configuration knobs.
2. **Drag from catalog** (form-factor tile) → drops onto map → FoV cone appears instantly.
3. **"Browse Models"** → opens picker modal → select brand/model → "Place Camera" → placed at canvas center, ready to drag.
4. **Rotate handle** on selected camera → cone rotates.
5. **Wall tool** → click-click to draw an obstruction line that clips cones.
6. **Scale tool** → draw a line, type real-world length → sets pixels-per-meter ratio.
7. **Right-click** on device → Delete / Duplicate / Properties.

### DORI Zone Logic (FoV cone math)
For a camera at mounting height `H`, tilt angle `θ`, and focal length `f`:
- **Detection range** = sensor_width / (focal_length_mm * 0.25 px/m)  *(simplified)*
- Each DORI zone is a nested arc segment within the main cone.
- Cone clips against wall segments using line-segment intersection (ray casting).
- **Glass walls**: ray continues past the wall at 55% of remaining range.
- **Fence/barriers**: ray continues at 35% of remaining range.

### Accessibility & Performance
- Canvas stage redraws only dirty regions (Konva's default).
- Camera model DB loaded once, filtered in-memory.
- Keyboard shortcuts: `V` = select, `C` = camera, `W` = wall, `S` = scale, `Del` = delete, `⌘Z` = undo.

---

## IV. Step-by-Step Implementation Plan

### Phase 1 — Canvas Foundation ✅ COMPLETE
**Built in:** `site-planner-prototype.html`

1. **[✅] HTML shell** — Three-panel layout (catalog / canvas / properties), Lumana nav sidebar, top bar, toolbar, status bar
2. **[✅] Map Canvas** — Konva stage, demo floor plan with rooms + furniture, zoom/pan, grid toggle
3. **[✅] Camera Placement** — Drag-from-catalog, click-to-place, rotation handle drag, Arrow key nudge
4. **[✅] FoV Cone Rendering** — Ray-cast DORI zones (Detection/Recognition/Identification), clips against demo walls in real-time
5. **[✅] Scale Calibration Tool** — Draw line → enter real-world length → sets px/m ratio
6. **[✅] Wall Drawing Tool** — Click-click to draw walls; cones clip against them immediately
7. **[✅] Properties Panel** — DORI zone toggles, resolution, FoV angle, mount height, tilt, retention, license; live dead zone / storage / bandwidth calc
8. **[✅] Live BOM** — Auto-updating table with unit + total MSRP
9. **[✅] Floor Plan Upload** — JPG/PNG replaces demo floor plan on canvas
10. **[✅] Undo/Redo** — Full stack (⌘Z / ⌘⇧Z)
11. **[✅] Context Menu** — Right-click: Duplicate / Properties / Delete

---

### Phase 2 — Camera Model Picker ✅ COMPLETE
**Built in:** `site-planner-prototype.html`

6. **[✅] Left Panel — Form Factor Quick-Picks** *(replaced old catalog list)*
   - 5 quick-add tile grid: Indoor Dome / Outdoor Dome / Bullet / Fisheye / Multi-sensor (SVG icon + label)
   - **"Browse 500+ Models"** purple button at panel top
   - "Recently used" section below tiles (last 4 CAMERA_DB models placed, session-persistent)

7. **[✅] Camera Model Picker Modal** *(IPVM-style 3-column browser)*
   - **Filter bar:** search input + form-factor chips (All / Dome / Bullet / Fisheye / Multi-sensor / PTZ) + NDAA-only toggle + min detection range slider
   - **3-column body:**
     - Left 180px: scrollable brand list with live model counts (Verkada, Axis, Hanwha, Bosch, Hikvision, Dahua, Avigilon, Pelco)
     - Middle 300px: model list with form factor icon, model name, NDAA badge, price
     - Right flex: info panel — model name, brand, resolution/form-factor badges, FoV cone SVG preview with DORI zones, 6-spec grid, MSRP
   - **Footer:** live model count, Cancel, **Place Camera** CTA
   - `CAMERA_DB` — 36 models across 8 brands with DORI ranges, NDAA flags, form factors
   - Placing pre-fills all camera specs (FoV angle, detection/recognition/identification ranges, color)

---

### Phase 3 — Gradient FoV Cone Visualization ✅ COMPLETE
**Built in:** `site-planner-prototype.html`

**Goal:** Replace the current yellow/green/purple DORI zone cones with a single gradient cone using the camera's brand color. Each camera type gets a unique color identity; the cone depth is shown via opacity layering, not hue switching.

**Visual spec (reference: IPVM camera calculator screenshot):**
- **Single brand color** per camera — no separate yellow/green/purple zones
- **3 opacity layers** of the same hue, rendered back-to-front:
  - Detection (outer, full range): camera color at ~15% opacity
  - Recognition (middle): camera color at ~30% opacity
  - Identification (inner, nearest): camera color at ~50% opacity
- Effect: cone is rich/dark near the camera body and fades to nearly transparent at the far edge
- DORI zone toggle in properties panel still controls visibility of each layer — just all in the same hue
- Wall clipping still applies to all three layers independently

8. **[✅] Replace multi-color DORI polygons with single-color gradient layers**
   - Remove yellow (detection) / green (recognition) / purple (identification) fill colors
   - Each camera has one `color` field (from CATALOG / CAMERA_DB); all three zone layers use it
   - Konva `opacity` per layer: `idR` zone = 0.50, `recR` zone = 0.30, `detR` zone = 0.15
   - Add a subtle stroke outline (same color, 0.4 opacity) at the outer detection boundary for crispness

---

### Phase 4 — Wall Types & Obstruction Modeling ✅ COMPLETE
**Built in:** `site-planner-prototype.html`

9. **[✅] Wall Type Selector**
   - Sub-toolbar slides in below main toolbar when Wall tool (W) is active: 🔴 Solid · 🔵 Glass · 🟡 Fence
   - **Solid:** full FoV block (existing behavior)
   - **Glass:** ray continues past wall at 55% remaining range (`attenuation: 0.55`)
   - **Fence/barrier:** ray continues at 35% remaining range (`attenuation: 0.35`)
   - Multiple attenuation walls compound correctly (each reduces remaining range further)
   - Visual distinction: solid red / blue dashed / amber dotted — preview line matches active type while drawing
   - Wall type stored per segment in `state.userWalls[].type`; persists through undo/redo
   - Canvas legend appears (bottom-left overlay) whenever any user walls are present

---

### Phase 5 — Camera Configuration ✅ COMPLETE
**Built in:** `site-planner-prototype.html`

10. **[✅] Focal Length → FoV binding**
    - Focal length slider (1.8–25mm) in properties panel, with preset chips: 2.8 / 4 / 6 / 8 / 12 / 25mm
    - Formula uses 1/2.8" sensor (6.4mm width): `FoV = 2 × atan(3.2 / focalMm) × 180/π`
    - Changing focal length updates FoV angle live + scales all DORI ranges proportionally
    - FoV slider remains editable and syncs focal length back when dragged
    - Preset chip highlights update in both directions
    - Fisheye (360°) cameras show "Fixed fisheye — 360°" instead of sliders
    - Each camera stores `baseFocalLength` + `baseDetR/Rec/Id` at placement for correct proportional scaling

---

### Phase 6 — Enhanced BOM Panel ✅
**Goal:** Right panel shows a professional cost breakdown with SKUs, matching the Verkada site designer output.

12. **[x] Project Total Header**
    - Always-visible top section of right panel: large MSRP total + "estimate" label + purple filled bar

13. **[x] Tabbed Right Panel: Properties | BOM**
    - Properties tab: existing camera config (unchanged)
    - BOM tab: full project breakdown

14. **[x] BOM — Grouped Sections**
    - **Cameras** section with subtotal: `📷 1 × CD63-E · $899`
    - **Licensing** section: `🔑 1 × LIC-CAM-1Y-CAP · $199`
    - Section subtotals + grand total row
    - "Export as CSV" link

---

### Phase 7 — Bandwidth & Storage Calculator ✅
**Goal:** Quantified network and storage impact per camera and for the whole site.

15. **[x] Per-camera calculations** *(extend properties panel)*
    - Formula: `storage_GB_day = bitrate_mbps × 86400 / 8 / 1024`
    - Bitrate table by resolution: 1080p=4Mbps, 4K=16Mbps, 8MP=10Mbps
    - 3-chip display: Mbps · GB/day · TB/retention — plus visual storage bar
    - "Bandwidth & Storage" section replaced flat Calculated Values

16. **[x] Site-wide summary** *(BOM pane — Storage & Network section)*
    - Total bandwidth (Mbps) across all cameras
    - Total storage (TB) at chosen retention period
    - NVR recommendation via `calcNVR()`: picks smallest drive count from 2TB/4TB/8TB/16TB tiers
    - Included in CSV export

---

### Phase 8 — Satellite Map & Floor Plan Enhancement ✅
**Goal:** Place cameras on real-world maps, not just uploaded plans.

16. **[x] Address Search + Satellite Background**
    - Globe button in toolbar → toggles satellite subtoolbar
    - Mode toggle: Canvas | Satellite | Both
    - ESRI World Imagery satellite tiles via Leaflet (no API key)
    - Address search → Nominatim geocoding → flies map to location
    - Konva stage renders transparently over Leaflet when map active

17. **[x] Floor Plan Opacity Slider (Both mode)**
    - Shown only in "Both" mode; controls Konva floor layer opacity (10–100%)
    - Allows aligning uploaded plan against satellite view

18. **[x] Scale Auto-Detection**
    - `syncScaleFromMap()` — derives scalePPM from Leaflet zoom + latitude using Web Mercator formula
    - Status bar shows "(map)" suffix when scale is map-derived
    - Updates on every zoom/pan via Leaflet `zoomend`/`moveend` events

---

### Phase 9 — 3D Scene Preview ✅
**Goal:** Validate what a camera actually sees at a given mounting height + tilt.

19. **[x] 3D Preview Panel**
    - 3D toolbar button → slide-over panel (360px, absolute over canvas right edge)
    - Three.js (r160) WebGL scene: grid floor, mounting pole, camera body with lens
    - Semi-transparent FoV frustum cones (Det/Rec/Id) in camera's brand color
    - Dead-zone red circle at ground level (blind spot directly beneath camera)
    - DORI range rings on floor (ID/REC/DET radii)
    - Mouse-drag orbit, scroll zoom; standard isometric default view

20. **[x] Mounting Variables in 3D**
    - Mount height (ft→m) and tilt angle update frustum in real-time via updateUI()
    - Target distance slider (0.5–20m) moves test person avatar; info bar shows active DORI zone
    - Selection change → 3D refreshes instantly (renderProperties / renderPropertiesEmpty hooks)

---

### Phase 10 — PDF Report Export ✅
**Goal:** Deliver a professional output document for clients or handoff.

21. **[x] PDF Report Generation**
    - "Export PDF" button → generates 3-page A4 PDF via jsPDF 2.5 (UMD CDN, no build step)
    - **Page 1 — Cover:** dark background, purple hero band, project name + date, 4 stat cards (cameras / MSRP / storage / bandwidth), NVR recommendation line
    - **Page 2 — Floor Plan:** Konva `stage.toDataURL()` screenshot (2× pixel ratio), camera dots + numbered labels mapped to image coordinates, numbered legend below image
    - **Page 3 — BOM:** Cameras table (model / qty / unit / total) + Licensing + Storage & Network sections + grand total bar; shared section band helper, alternating row shading
    - Button shows spinner while generating; restored on completion
    - File saved as `{project-name}-site-plan.pdf`

---

### Phase 11 — Save, Share & Collaborate ✅
**Goal:** Plans can be saved, shared as read-only links, and resumed later.

22. **[x] Save / Load Plan (JSON)**
    - `planSnapshot()` — serializes cameras, walls, scale, nextId, projectName, savedAt (v2 schema)
    - "Save" button → downloads `{project-name}.json`
    - "Open" button (topbar) + share modal → file picker → `loadDesign()` restores full state via `restoreState` + `rebuildCanvas`
    - Handles both v1 (cameras+walls only) and v2 (full metadata) files

23. **[x] URL Share Link** *(serverless — encoded in URL hash)*
    - "Share" button → opens share modal with generated link
    - Plan JSON compressed via LZString `compressToEncodedURIComponent` → `#plan=...` URL fragment
    - "Copy link" button → `navigator.clipboard` with "✓ Copied!" feedback state
    - On page load → `checkHashPlan()` auto-restores plan from hash, then clears hash from address bar
    - Modal also surfaces Download JSON + Open saved file as secondary actions

---

### Phase 12 — Integrations (Production)
**Goal:** Connect the plan to real systems for instant activation and quoting.

24. **[ ] Salesforce Push** — Map BOM line items to Salesforce Opportunity via API
25. **[ ] Verkada Command Sync** — Push placed cameras + positions to Command platform
26. **[ ] Access Control + Sensors** — Add door controllers, card readers, intercoms, air sensors to catalog (deferred from v1)

---

## V. Current Status

| Feature | Phase | Status |
|---|---|---|
| Canvas with draggable cameras | 1 | ✅ Done |
| FoV cones with ray-cast DORI zones | 1 | ✅ Done |
| Camera rotation + FoV angle drag | 1 | ✅ Done |
| Wall drawing + cone clipping | 1 | ✅ Done |
| Scale calibration tool | 1 | ✅ Done |
| Floor plan image upload | 1 | ✅ Done |
| Properties panel (resolution, tilt, retention…) | 1 | ✅ Done |
| Basic BOM table | 1 | ✅ Done |
| Bandwidth/storage calc (status bar) | 1 | ✅ Done |
| Undo/redo, context menu, keyboard shortcuts | 1 | ✅ Done |
| Left panel form-factor quick-picks | 2 | ✅ Done |
| Camera Model Picker modal (500+ models, 8 brands) | 2 | ✅ Done |
| Gradient FoV cone (single-color opacity layers) | 3 | ✅ Done |
| Wall types — Glass / Fence attenuation | 4 | ✅ Done |
| Focal length ↔ FoV binding | 5 | ✅ Done |
| Project total header + tabbed right panel | 6 | ⬜ |
| BOM with Hardware / Licensing / Add-Ons | 6 | ⬜ |
| Per-camera storage/bandwidth details | 7 | ⬜ |
| Site-wide storage summary | 7 | ⬜ |
| Satellite map + address search | 8 | ⬜ |
| Floor plan alignment over satellite | 8 | ⬜ |
| 3D scene preview | 9 | ⬜ |
| PDF report export | 10 | ⬜ |
| Save / load JSON plan | 11 | ⬜ |
| View-only share link | 11 | ⬜ |
| Salesforce / Command sync | 12 | ⬜ |
| Access Control + Sensor devices | 12 | ⬜ |

---

## VI. Key Libraries (CDN links for prototype)

```html
<!-- Canvas (Phase 1 — in use) -->
<script src="https://unpkg.com/konva@9/konva.min.js"></script>

<!-- Maps (Phase 8) -->
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<!-- 3D (Phase 9) -->
<script src="https://unpkg.com/three@0.160.0/build/three.min.js"></script>

<!-- PDF export (Phase 10) -->
<script src="https://unpkg.com/jspdf@2.5.1/dist/jspdf.umd.min.js"></script>
<script src="https://html2canvas.hertzen.com/dist/html2canvas.min.js"></script>
```

---

## VII. Phase Size Guide

Each phase is scoped to **one focused feature** buildable in a single session:

| Phase | Feature | Size |
|---|---|---|
| 1 | Canvas + FoV cones | ✅ Done |
| 2 | Camera Model Picker modal | ✅ Done |
| 3 | Gradient FoV cone visualization | ✅ Done |
| 4 | Wall types (Glass / Fence) | Small logic + sub-toolbar |
| 5 | Focal length ↔ FoV binding | Small — properties extension |
| 6 | Enhanced BOM panel | Medium — right panel redesign |
| 7 | Storage/bandwidth calculator | Small — math + UI |
| 8 | Satellite map + floor plan align | Medium — Leaflet integration |
| 9 | 3D scene preview | Large — Three.js setup |
| 10 | PDF report export | Medium — jsPDF |
| 11 | Save / Share | Small–Medium |
| 12 | Integrations | Large — needs backend |
