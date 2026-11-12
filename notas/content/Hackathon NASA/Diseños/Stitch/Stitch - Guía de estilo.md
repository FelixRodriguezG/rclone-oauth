---
name: Ares Cockpit Monitoring
colors:
  surface: '#131411'
  surface-dim: '#131411'
  surface-bright: '#393936'
  surface-container-lowest: '#0e0e0c'
  surface-container-low: '#1b1c19'
  surface-container: '#1f201d'
  surface-container-high: '#2a2a27'
  surface-container-highest: '#353532'
  on-surface: '#e4e2dd'
  on-surface-variant: '#e2bfb2'
  inverse-surface: '#e4e2dd'
  inverse-on-surface: '#30312d'
  outline: '#a98a7e'
  outline-variant: '#5a4137'
  surface-tint: '#ffb597'
  primary: '#ffb597'
  on-primary: '#581d00'
  primary-container: '#ff6a1a'
  on-primary-container: '#591e00'
  inverse-primary: '#a43d00'
  secondary: '#ffb956'
  on-secondary: '#462b00'
  secondary-container: '#c3841e'
  on-secondary-container: '#3d2500'
  tertiary: '#ffb4a5'
  on-tertiary: '#650a00'
  tertiary-container: '#ff684c'
  on-tertiary-container: '#660a00'
  error: '#ffb4ab'
  on-error: '#690005'
  error-container: '#93000a'
  on-error-container: '#ffdad6'
  primary-fixed: '#ffdbcd'
  primary-fixed-dim: '#ffb597'
  on-primary-fixed: '#360f00'
  on-primary-fixed-variant: '#7d2d00'
  secondary-fixed: '#ffddb5'
  secondary-fixed-dim: '#ffb956'
  on-secondary-fixed: '#2a1800'
  on-secondary-fixed-variant: '#643f00'
  tertiary-fixed: '#ffdad3'
  tertiary-fixed-dim: '#ffb4a5'
  on-tertiary-fixed: '#3f0400'
  on-tertiary-fixed-variant: '#8e1300'
  background: '#131411'
  on-background: '#e4e2dd'
  surface-variant: '#353532'
typography:
  display-lg:
    fontFamily: Archivo Narrow
    fontSize: 64px
    fontWeight: '300'
    lineHeight: 64px
    letterSpacing: -0.02em
  display-lg-mobile:
    fontFamily: Archivo Narrow
    fontSize: 48px
    fontWeight: '300'
    lineHeight: 48px
    letterSpacing: -0.02em
  display-md:
    fontFamily: Archivo Narrow
    fontSize: 36px
    fontWeight: '300'
    lineHeight: 40px
    letterSpacing: -0.01em
  headline-sm:
    fontFamily: Space Mono
    fontSize: 16px
    fontWeight: '700'
    lineHeight: 24px
    letterSpacing: 0.1em
  body-md:
    fontFamily: Space Mono
    fontSize: 13px
    fontWeight: '400'
    lineHeight: 20px
    letterSpacing: 0.02em
  body-sm:
    fontFamily: Space Mono
    fontSize: 11px
    fontWeight: '400'
    lineHeight: 16px
    letterSpacing: 0.04em
  label-lg:
    fontFamily: Space Mono
    fontSize: 12px
    fontWeight: '700'
    lineHeight: 16px
    letterSpacing: 0.14em
  label-md:
    fontFamily: Space Mono
    fontSize: 10px
    fontWeight: '700'
    lineHeight: 14px
    letterSpacing: 0.18em
  label-sm:
    fontFamily: Space Mono
    fontSize: 9px
    fontWeight: '400'
    lineHeight: 12px
    letterSpacing: 0.2em
spacing:
  gutter: 0.5rem
  gutter-mobile: 0.25rem
  margin: 1rem
  margin-mobile: 0.75rem
  space-xs: 0.125rem
  space-sm: 0.25rem
  space-md: 0.5rem
  space-lg: 1rem
  space-xl: 1.5rem
---

## Brand & Style

The design system establishes a high-criticality telemetry interface modeled after spacecraft cockpit instruments, the 1975 NASA Graphics Standards Manual, and Swiss functionalism. It rejects decorative tech tropes: no neon gradients, no simulated depth, no ambient card fills, and no rounded surfaces. 

Every pixel is functional, legible under harsh ambient light or extreme darkness, and engineered for rapid scanning during operational duties on the Martian surface. The emotional tenor is quiet, disciplined, and clinical. Information architecture relies on rigid spatial coordinate grids, disciplined hairline borders, and deliberate typographic rhythm to convey situational awareness.

## Colors

The palette enforces a strict visual economy where color serves as an operational state indicator rather than surface styling.

- **Background & Canvas:** Pure deep cockpit black (`#07080A`). Recessed monitor modules use `#0B0D12` only when spatial demarcation cannot be achieved via rule lines alone.
- **Structural Hairstyle:** `#1F2329` strictly 1px borders and grid delineators.
- **Text & Core Readouts:** Primary data points, typography, and normal states use `#E8E6E1`. Secondary metadata, inactive states, units, and structural lines use `#6E737B`.
- **System Accent:** Signal orange (`#FF6A1A`) is restricted entirely to application branding, the active navigation marker, and manual operational inputs.
- **Telemetry States:**
  - **NOMINAL:** Rendered in quiet off-white (`#E8E6E1`) with explicit mono text labels `[NOMINAL]` and hollow squares (`▢`). No green lights.
  - **CAUTION:** Industrial amber (`#E8A33D`) accompanied by warning text `[CAUTION]` and hollow triangles (`△`).
  - **CRITICAL:** High-visibility cockpit red (`#FF4D2E`) paired with `[CRITICAL]` and solid squares (`■`) or 1px structural accent rules.

## Typography

The typographic hierarchy juxtaposes condensed, high-legibility telemetry numbers with rigid monospaced data structures.

- **Telemetry Readouts:** Powered by light-weight Archivo Narrow. Numbers are tabular, condensed, and set large (`48px`–`64px`) to replicate analog-to-digital flight monitors that remain legible at arm's length or in low-gravity vibration.
- **Labels, Metadata, & Control States:** Set strictly in monospaced uppercase with wide tracking (`0.1em`–`0.2em`). Field labels must precede data values with standard aerospace abbreviation standards (`O2_LVL`, `CABIN_P`, `BAT_SYS`).
- **Formatting:** No mixed-case headers. All operational statuses, coordinates, units, and timestamps appear in uppercase monospace.

## Layout & Spacing

The layout is built upon a dense, disciplined instrument grid inspired by Swiss modular systems and aerospace instrumentation consoles.

- **Grid Architecture:** 4-column structured mobile grid expanding to an 8-column tablet display. Components sit edge-to-edge separated by hairline dividers rather than floating canvas margins.
- **Spacing Rhythm:** Micro-increments based on a strict 4px/8px rule. Whitespace is compressed and purposeful, maximizing screen real estate for sensor readouts without visual clutter.
- **Section Enclosures:** Information modules are bounded by hairline borders (`1px solid #1F2329`) or aligned directly along continuous vertical registration axes. No margins exist between juxtaposed data cells; they share cell walls like structural bulkheads.

## Elevation & Depth

This design system uses zero drop shadows, zero blurs, and zero surface extrusion. Elevation is communicated through flat, technical layering:

- **Base Layer:** Raw background `#07080A`.
- **Sub-surface Layer:** `#0B0D12` used exclusively for recessed telemetry modules, data logs, and nested coordinate frames.
- **Lineal Delimitation:** 1px hairline borders (`#1F2329`) define spatial zones and active instrument perimeters. 
- **Z-Index Layering:** Overlays (such as emergency alerts or critical task overrides) render as opaque `#07080A` panels with a complete 1px perimeter border in `#FF4D2E` or `#E8E6E1`, blocking out background instrumentation entirely without blurred passthrough.

## Shapes

The geometric vernacular is strictly orthoginal:
- All corners are sharp right angles (`0px`). Under no circumstances are rounded pills, soft buttons, or circular indicators permitted.
- Secondary technical flourishes must be strictly functional: 45-degree chamfers on corners (`2px` max) may be used solely on structural bracket dividers or technical cut lines to indicate physical module edges.
- Visual anchors (rulers, progress marks, ticks) use clean 1px rectangles.

## Components

### Top Application Bar & Branding
- **RYUOK Monogram:** Set in uppercase Space Mono, `#FF6A1A`, locked to top left. Followed by mission elapsed time (`MET: +142:08:12`) in `#6E737B`.
- **Perimeter Line:** A continuous 1px `#1F2329` border anchors the top bar across the full screen width.

### Navigation (Bottom Bar)
- Composed of 4 persistent text tabs in uppercase monospace: `HOME` · `MEASURE` · `TASKS` · `CHANNEL`.
- Height: 48px fixed. Background: `#07080A`. Top edge: 1px hairline border `#1F2329`.
- **Inactive Tabs:** Set in `#6E737B`, unadorned.
- **Active Tab:** Set in `#E8E6E1` with a dedicated 2px high underline in `#FF6A1A` spanning the width of the active tab label.

### Telemetry Modules & Data Readouts
- Structured as open wireframe cells. No background fills.
- Top of cell: 1px hairline bar with embedded uppercase label (`CABIN_PRESSURE // KPA`).
- Telemetry Value: Archivo Narrow numerals in `#E8E6E1` with inline unit indicators (`kPa`, `ppm`, `°C`) set in 9px Space Mono in `#6E737B`.
- Status Tags: Placed directly within the module header or footer using raw text and geometric glyphs:
  - `▢ NOMINAL` (`#E8E6E1`)
  - `△ CAUTION` (`#E8A33D`)
  - `■ CRITICAL` (`#FF4D2E`)
- Optional state border: A single 2px colored border along the left edge of the cell indicates warning or alarm states.

### Segmented Meters & Bar Graphs
- Meters do not use continuous solid progress bars.
- Composed of discrete 1px or 2px tick blocks separated by 1px gaps.
- Scale ticks: Regular 1px marks at 10%, 50%, and 90% thresholds with hairline numeric index markers.

### Interactive Controls (Buttons)
- **Primary Action:** Rectangular button with 1px border `#FF6A1A`. Text set in Space Mono Bold `#FF6A1A`. On press: solid fill `#FF6A1A` with text shifting to `#07080A`.
- **Standard Action:** 1px border `#1F2329`, text `#E8E6E1`. On press: hairline shifts to `#E8E6E1`.
- **Destructive/Critical Action:** 1px border `#FF4D2E`, text `#FF4D2E`.

### Inputs & Toggles
- **Checkboxes:** 12x12px square border (`1px solid #6E737B`). Active checked state displays a solid 6x6px interior square in `#E8E6E1`.
- **Text Entry:** Hairline bottom border only (`1px solid #1F2329`), changing to `#E8E6E1` on focus, paired with a blinking non-proportional block cursor (`█`).

### Line Icons & Technical Glyphs
- All iconography drawn strictly with 1px stroke weight, sharp corners, and zero fill.
- Symbols emulate technical schematics, radar vector graphics, and standard engineering blue-print symbols.