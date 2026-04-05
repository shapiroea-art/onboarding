# Lumana Site Planner — Design System Rules

## Project Overview

This is a prototype for the Lumana Site Planner product — a video security system planning tool. The codebase consists of standalone HTML files with embedded CSS and JavaScript. No build system, no frameworks, no npm dependencies for the UI.

## Technology Stack

- **Language**: Vanilla HTML, CSS, JavaScript
- **Styling**: Embedded `<style>` blocks in each HTML file (no CSS variables, no preprocessors)
- **Framework**: None — plain DOM manipulation
- **Icons**: Inline SVG (stroke-based, `currentColor`, `viewBox="0 0 24 24"`)
- **Maps**: Leaflet.js (CDN)
- **Canvas**: Konva.js (CDN)

## File Structure

```
site-planner-prototype.html  — Main floor plan canvas with BOM panel
site-planner-location.html   — Location/plan BOM views (all-locations, single-location, single-plan)
site-planner-sites.html      — Projects list view
site-planner-plan.html       — Plan overview sidebar
```

## Figma Design System References

- **Components**: `figma.com/design/OC4dC118cRhq2L3yPFrO2X/02-Desktop-Components`
- **Colors**: `figma.com/design/mZiZnH3GDN5CMnuhi3pd6r/01-Foundations` (node `939-8`)
- **Typography**: `figma.com/design/mZiZnH3GDN5CMnuhi3pd6r/01-Foundations` (node `3-28`)
- **Shadows**: `figma.com/design/mZiZnH3GDN5CMnuhi3pd6r/01-Foundations` (node `3-30`)
- **Icons**: `figma.com/design/lXI4UA3B9OqKzU8nrlWdX8/04-Iconography` (node `3-23`)

## Figma MCP Integration Rules

### Required Flow (do not skip)

1. Run `get_design_context` first to fetch the structured representation for the exact node(s)
2. If the response is too large or truncated, run `get_metadata` to get the high-level node map, then re-fetch only the required node(s) with `get_design_context`
3. Run `get_screenshot` for a visual reference of the node variant being implemented
4. Only after you have both `get_design_context` and `get_screenshot`, download any assets needed and start implementation
5. Translate the output (usually React + Tailwind) into vanilla HTML/CSS/JS matching this project's conventions
6. Validate against Figma for 1:1 look and behavior before marking complete

### Implementation Rules

- IMPORTANT: Treat the Figma MCP output (React + Tailwind) as a representation of design and behavior, not as final code style. Convert to vanilla HTML/CSS/JS.
- IMPORTANT: All styling goes in embedded `<style>` blocks — no external CSS files, no CSS-in-JS
- IMPORTANT: Reuse existing HTML structure and JS logic when updating prototypes — do not rewrite functional code
- IMPORTANT: Preserve all existing JavaScript functionality when updating styles
- Strive for 1:1 visual parity with the Figma design
- Validate the final UI against the Figma screenshot for both look and behavior

### Asset Handling

- IMPORTANT: If the Figma MCP server returns a localhost source for an image or SVG, use that source directly
- IMPORTANT: DO NOT import/add new icon packages — all icons are inline SVGs
- Store downloaded image assets in `figma-assets/`

---

## Color Palette

### Semantic Color Tokens (from Figma: `color-*`)

**Text Colors**
| Token | Hex | Usage |
|-------|-----|-------|
| `color-text-default` | `#344054` | Body text, labels |
| `color-text-bold` | `#101828` | Headings, primary text |
| `color-text-subtle` | `#667085` | Secondary text, icons |
| `color-text-subtlest` | `#98a2b3` | Captions, placeholders, disabled |
| `color-text-inverse` | `#fff` | Text on dark/colored backgrounds |
| `color-text-selected` | `#7c3aed` | Active/selected state text |
| `color-text-disabled` | `#98a2b3` | Disabled controls |

**Brand / Primary**
| Hex | Usage |
|-----|-------|
| `#7c3aed` | Primary actions, active tabs, focus rings, brand accent |
| `#6d28d9` | Primary hover state |
| `#5b21b6` | Primary pressed state |
| `#ede9fe` | Active nav background |
| `#f5f3ff` | Light purple hover background |
| `#faf8ff` | Lightest purple tint |

**Backgrounds**
| Hex | Usage |
|-----|-------|
| `#fff` | Primary surface (cards, modals, panels) |
| `#f9fafb` | Secondary surface (page background, hover, section fills) |
| `#f2f4f7` | Tertiary surface (hover states, tag backgrounds) |
| `#eaecf0` | Borders (standard) |
| `#d0d5dd` | Borders (form inputs, buttons) |
| `#f8fafc` | Subtle surface |

**Status Colors**
| Status | Default | Dark/Hover | Light BG |
|--------|---------|------------|----------|
| Success | `#10b981` | `#065f46` | `#ecfdf5` |
| Error | `#d92d20` | `#d92d20` | `#fef3f2` |
| Warning | `#d97706` | — | — |
| Info | `#0369a1` | — | `#e0f2fe` |

**Device/Camera Accent Colors**
- Purple: `#7c3aed`
- Blue: `#2563eb`
- Teal: `#0891b2`
- Green: `#059669`
- Orange: `#d97706`
- Red: `#dc2626`
- Pink: `#be185d`
- Indigo: `#4338ca`

### IMPORTANT: Color Rules

- NEVER introduce new hex colors — always use values from this palette
- Text must use the semantic text colors above
- Borders: use `#eaecf0` for standard, `#d0d5dd` for inputs/buttons, `#f2f4f7` for subtle
- Hover backgrounds: use `#f9fafb` (light) or `#f5f3ff` (purple tint)
- Focus rings: `box-shadow: 0 0 0 3px rgba(124,58,237,0.12)` with `border-color: #7c3aed`

---

## Typography

### Font Stack

```css
font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
```

### Type Scale (from Figma: `Text-*`)

| Name | Size | Weight | Usage |
|------|------|--------|-------|
| `Text-sm` (caption) | `11px` | `500–600` | Badges, captions, fine labels |
| `Text-base` (small) | `12px` | `500–600` | Small buttons, toolbar items, tags |
| `Text-lg` (body) | `13px` | `400–500` | Form inputs, list items, body text |
| `Text-xl` (body-lg) | `14px` | `400–600` | Default body, breadcrumbs, row text |
| `Text-2xl` (heading-sm) | `15px` | `600–700` | Section headers, panel titles |
| `Text-3xl` (heading) | `16px` | `600` | Major section titles |
| `Text-4xl` (title) | `18px` | `600` | Modal titles |
| `Text-5xl` (display) | `24px` | `700–800` | Grand total values |

### IMPORTANT: Typography Rules

- NEVER use font sizes outside this scale
- Body text: `14px` weight `400`–`500`, color `#344054`
- Headings: `15px`–`18px` weight `600`–`700`, color `#101828`
- Labels: `11px`–`12px` weight `500`–`600`, color `#98a2b3`, often `text-transform: uppercase; letter-spacing: 0.6px`
- Prices/values: `font-variant-numeric: tabular-nums` for alignment

---

## Spacing

### Base Scale (4px grid)

| Token | Value | Common Usage |
|-------|-------|--------------|
| `space-1` | `4px` | Tight gaps, icon margins |
| `space-2` | `6px` | Small gaps (breadcrumbs, inline elements) |
| `space-3` | `8px` | Standard gap (flex items, list spacing) |
| `space-4` | `10px` | Medium padding |
| `space-5` | `12px` | Section gaps, button padding |
| `space-6` | `16px` | Standard section padding, panel padding |
| `space-8` | `20px` | Large sections |
| `space-10` | `24px` | Modal padding, content headers |
| `space-12` | `28px` | Modal inner padding |

### Row Heights

- Standard row: `min-height: 40px`–`44px`
- Location/section row: `min-height: 48px`
- Compact row (toolbar items): `28px`–`32px`

---

## Border Radius

| Value | Usage |
|-------|-------|
| `4px` | Tags, tooltips, small elements |
| `6px` | Buttons, icon buttons, form inputs |
| `8px` | Cards, dropdowns, section containers, standard rounding |
| `10px` | Feature cards, included sections |
| `12px` | Modals |
| `50%` | Avatars, circular badges |

---

## Shadows

### Elevation Levels (from Figma: `Shadows`)

| Level | Value | Usage |
|-------|-------|-------|
| `xs` | `0 1px 4px rgba(0,0,0,0.12)` | Small buttons |
| `sm` | `0 4px 12px rgba(124,58,237,0.35)` | Primary CTA |
| `md` | `0 8px 20px rgba(16,24,40,0.1)` | Dropdowns, menus |
| `lg` | `0 8px 24px rgba(16,24,40,0.12)` | Large dropdowns |
| `xl` | `0 20px 60px rgba(16,24,40,0.2)` | Standard modals |
| `2xl` | `0 24px 64px rgba(16,24,40,0.22)` | Large modals |
| Focus | `0 0 0 3px rgba(124,58,237,0.12)` | Input/button focus ring |

---

## Icon System

### Conventions

- All icons are **inline SVG** — no icon fonts, no external packages
- Standard `viewBox="0 0 24 24"`
- Stroke-based: `stroke="currentColor"`, `stroke-width="1.5"` to `1.8`, `fill="none"`
- Size set via CSS (`width`/`height`): typically `14px`–`20px`
- Color inherits from parent via `currentColor`, or set explicitly
- Default icon color: `#667085` (subtle), active: `#7c3aed`

### Common Icon Sizes

| Context | Size |
|---------|------|
| Navigation | `20px` |
| Toolbar buttons | `15px` |
| Inline with text | `14px`–`16px` |
| Form field icons | `14px` |
| Large decorative | `24px`+ |

### IMPORTANT: Icon Rules

- NEVER import icon libraries (Lucide, Heroicons, etc.)
- Always use inline `<svg>` with `stroke="currentColor"` for theme compatibility
- Camera type icons use specific shapes: dome (ellipse), bullet (rect+circle), fisheye (concentric circles)

---

## Component Patterns

### Buttons

```css
/* Standard button */
.btn {
  padding: 5px 12px;
  border-radius: 6px;
  font-size: 13px;
  font-weight: 500;
  border: 1px solid #d0d5dd;
  background: #fff;
  color: #344054;
}

/* Primary button */
.btn-primary {
  background: #7c3aed;
  border-color: #7c3aed;
  color: #fff;
}
.btn-primary:hover { background: #6d28d9; }

/* Icon button */
.btn-icon {
  width: 32px;
  height: 32px;
  border: 1px solid #d0d5dd;
  border-radius: 6px;
  background: #fff;
  color: #667085;
}
```

### Modals

```css
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(16,24,40,0.5);
  z-index: 1000;
}
.modal {
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 20px 60px rgba(16,24,40,0.2);
  width: 400px–420px;
  padding: 28px;
}
.modal-title { font-size: 18px; font-weight: 600; color: #101828; }
.modal-input {
  padding: 9px 12px;
  font-size: 14px;
  border: 1px solid #d0d5dd;
  border-radius: 8px;
}
.modal-input:focus {
  border-color: #7c3aed;
  box-shadow: 0 0 0 3px rgba(124,58,237,0.12);
}
```

### Segmented Controls / Toggle Switches

```css
.bom-quick-switch {
  display: inline-flex;
  border: 1px solid #d0d5dd;
  border-radius: 8px;
  overflow: hidden;
}
.bom-quick-switch-btn {
  padding: 4px 8px;
  font-size: 12px;
  font-weight: 500;
  color: #667085;
  background: #fff;
}
.bom-quick-switch-btn.active {
  background: #7c3aed;
  color: #fff;
}
```

### Collapsible Rows

- Use chevron SVG that rotates `-90deg` when `.collapsed`
- Hide children with `display: none` on `.collapsed .children`
- Row height: `40px`–`48px`
- Hover: `background: #f9fafb`

### Tags/Badges

```css
/* Badge (count) */
.badge {
  font-size: 11px;
  font-weight: 600;
  padding: 2px 8px;
  border-radius: 10px;
  background: #f2f4f7;
  color: #667085;
}

/* Tag (property) */
.tag {
  font-size: 11px;
  padding: 2px 8px;
  border-radius: 4px;
  background: #f2f4f7;
  color: #667085;
}
```

---

## Project-Specific Conventions

### Navigation

- Left nav: `64px` wide, `#f9fafb` background, vertical icon buttons
- Active nav icon: `background: #ede9fe; color: #7c3aed`

### Topbar

- Height: `50px`
- Breadcrumbs: `14px`, links in `#667085`, current in `#344054 font-weight: 500`
- Separator: `›` in `#d0d5dd`

### BOM Panel (Prototype)

- Right panel width: `850px`, collapsible
- Hardware/Software category blocks with collapsible headers
- Camera items show inline tags grid: payment, duration, license, retention, resolution
- Grand total: purple value `#7c3aed`, `font-weight: 700`

### Location Page

- Location rows: clean flat style, `min-height: 48px`, bottom border
- Plan rows: `min-height: 44px`, with "View plan" button
- Core groups: colored left border accent (`4px solid`)
- Camera indentation: `68px` left padding under cores

### Map Integration

- Uses Leaflet.js for map tiles
- Floor plan SVG overlay on map
- Camera/device pins as interactive SVG elements

---

## What to Avoid

- NEVER add npm packages or external dependencies
- NEVER create separate CSS files — styles go in `<style>` blocks
- NEVER use CSS variables or custom properties (not used in this project)
- NEVER use Tailwind or any CSS framework classes
- NEVER change JavaScript logic when updating visual styles
- NEVER hardcode colors outside the defined palette
- NEVER add comments like `// removed` or `// deprecated` — just delete unused code
- NEVER create helper abstractions for one-time patterns
