# BOM Panel Redesign — Implementation Plan

## Context
The BOM (Bill of Materials) right panel needs a major redesign to improve structure, usability, and presentation. Currently it shows a flat "Devices" list grouped by core, with a toolbar containing Reassign and Bulk Edit buttons. The redesign introduces Hardware/Software categories, collapsible sections, quick-switch controls, drag-on-hover reassignment, and a marketing "What's included" block.

## File to modify
`site-planner-prototype.html`

---

## Phase 1: BOM Structure — Hardware/Software Categories & Ordering

**Goal**: Replace the flat "Devices" list with two collapsible category blocks: **Hardware** and **Software**, reorder groups so unassigned cameras come first, add per-category totals.

### 1a. Unassigned cameras first
In `renderBOMPane()`, move the unassigned cameras group rendering **before** the core groups loop. The order becomes:
1. Unassigned cameras
2. Core groups (each with assigned cameras)

### 1b. Hardware category wrapper
Wrap all device groups (unassigned cameras + core groups + add-ons) in a collapsible **Hardware** block:
```
┌─ Hardware ─────────────────── ▼ collapse ─┐
│  ┌ Unassigned Cameras (N)     $X,XXX      │
│  │  Camera 1 ...                           │
│  │  Camera 2 ...                           │
│  └                                         │
│  ┌ Core 1 (N cameras)         $X,XXX      │
│  │  Camera 3 ...                           │
│  └                                         │
│  ┌ Add-on section ...                      │
│  └                                         │
│  ─── Hardware Total: $XX,XXX ────          │
└────────────────────────────────────────────┘
```

### 1c. Software category
Add a new **Software** block below Hardware:
```
┌─ Software ─────────────────── ▼ collapse ─┐
│  Lumana License    ×N cameras    $X,XXX    │
│  ─── Software Total: $X,XXX ────           │
└────────────────────────────────────────────┘
```
- License row: `N × $199 = total` (using `LICENSE_PRICE.Standard`)
- License count = `state.cameras.length`

### 1d. Category collapse
- New CSS class `.bom-category` with collapse behavior
- Category header row: title + chevron + total price (always visible)
- When collapsed: hide all children, show only header + total row
- Track in new state: `_bomCategoryCollapse = { hardware: false, software: false }`
- New function `toggleBomCategory(key)`

### 1e. Category totals
- **Hardware Total** = cameras total + cores total + add-ons total
- **Software Total** = cameras.length × LICENSE_PRICE.Standard
- Display at footer of each category block

### 1f. Grand total
Below both categories. `Grand Total = Hardware Total + Software Total`.

---

## Phase 2: Core Row Simplification & Tag Alignment

**Goal**: Simplify core rows and align their Payment/Duration tags with camera columns.

### 2a. Remove total price from core rows
In the core group header, replace `$core.price | $totalPrice` with just `$core.price`. Remove separator and group total spans.

### 2b. Align core Payment/Duration with camera columns
Core tags use the same `bom-tags` 5-column grid as cameras (`58px 40px 68px 40px 44px` → Payment, Duration, License, Retention, Resolution). Core fills columns 1–2 (Payment, Duration), columns 3–5 remain empty. Verify vertical alignment with camera rows after price simplification.

---

## Phase 3: "What's Included" Marketing Section

**Goal**: Add a visually appealing benefits card below the grand total.

### Content
- Lumana AI engine
- Real time alerts
- Fast investigation
- 24/7 local storage
- Unlimited cloud backup for video clips
- Unlimited users
- Automatic software updates
- Dedicated onboarding, training, and U.S.-based support
- 100% Warranty

### Design
- Card-style container with purple gradient background (`#faf5ff → #f0ebff`)
- "What's included" as bold heading
- Each benefit as a row with a small purple check icon
- Compact (12px font), good line spacing
- Sits below `bom-grand` inside the BOM pane

---

## Phase 4: Quick Switches (Payment & Duration)

**Goal**: Add quick-switch pill buttons in the toolbar that instantly apply Payment or Duration to all devices.

### UI
```
[Upfront | Yearly]  [1yr | 3yr | 5yr | 10yr]  [Bulk Edit]
```

### Behavior
- **Payment toggle**: Clicking Upfront/Yearly applies to ALL cameras and cores immediately. Active state highlighted purple.
- **Duration toggle**: 1/3/5/10 year buttons. Only enabled when payment is Yearly (grayed out when Upfront). Clicking applies to all devices.
- Each click: `saveUndo()`, update all cameras + cores, re-render BOM + project total.
- Mixed state (not all devices share same value) = no button highlighted.

### Implementation
- `quickSwitchPayment(value)` — set payment on all cameras + cores. If Upfront, reset duration to 1.
- `quickSwitchDuration(value)` — set duration on all cameras + cores.
- Pill-style button group with `.bom-quick-switch` / `.bom-quick-switch-btn` classes.

---

## Phase 5: Drag-on-Hover Reassignment (Remove Reassign Button)

**Goal**: Remove the explicit "Reassign" button. Show a drag handle on camera row hover; pressing it activates drag-and-drop.

### 5a. Remove Reassign button
Delete the Reassign button from toolbar. Remove `toggleDragMode()` from toolbar.

### 5b. Always-available drag handle
In `renderBomCamItem()`, add a 6-dot grip icon to the **left** of the camera row. Hidden by default, visible on hover:
```css
.bom-item .bom-drag-handle { opacity: 0; transition: opacity 0.15s; }
.bom-item:hover .bom-drag-handle { opacity: 1; }
```

### 5c. Drag interaction without mode toggle
- Camera rows are always `draggable="true"`
- `dragstart`: activate drag state, show drop zones by adding CSS class to BOM container
- `drop`: move camera to target core immediately, `saveUndo()`, show success toast
- `dragend`: hide drop zones, clear drag state
- No Apply/Cancel bar — each drop is an immediate atomic action with undo

### 5d. Simplify drag state
- Remove `_dragMode.moveCount`, `_dragMode.snapshot`
- Keep `_dragMode.active` and `_dragMode.dragCamId` for in-progress tracking
- Remove `dragApply()`, `dragCancel()`, `toggleDragMode()`
- Simplify to: `bomDragStart` → show drop zones, `bomDropOnGroup` → move + undo, `bomDragEnd` → hide drop zones

---

## Verification Checklist

### Phase 1
- [ ] BOM shows Hardware block (unassigned first, then cores, then add-ons) and Software block (license row)
- [ ] Each block collapses — collapsed shows only header + total
- [ ] Category totals correct, grand total = hardware + software

### Phase 2
- [ ] Core rows show only core price (no separator, no group total)
- [ ] Core Payment/Duration tags align with camera Payment/Duration columns

### Phase 3
- [ ] "What's included" card appears below grand total with all 9 benefits
- [ ] Purple gradient background, check icons, readable

### Phase 4
- [ ] Quick switches visible in toolbar next to Bulk Edit
- [ ] Clicking "Yearly" → all devices switch, Duration buttons enable
- [ ] Clicking duration → all devices update
- [ ] Mixed state = no button highlighted

### Phase 5
- [ ] No Reassign button in toolbar
- [ ] Hovering camera row shows drag grip on left
- [ ] Dragging camera shows drop zones on cores/unassigned
- [ ] Dropping moves camera immediately with undo support
