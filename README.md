# Cidade Labs — map tool template

Shared foundation for Cidade Labs map tools, extracted from Bus Works (the
first tool, and the canonical reference for this pattern). Copy
`template.html` to start a new tool; keep the `SHARED` blocks as-is unless
there's a real reason to diverge, and fill in the `TOOL-SPECIFIC` blocks.

Each tool stays its own independently-deployed static repo under the
`cidade-labs` org — this template is a starting point to copy, not a live
shared dependency every site loads at runtime. That's deliberate: one bad
edit to a shared stylesheet shouldn't be able to break every deployed tool
at once.

## Design tokens

```
--ink       #16181a   primary text
--muted     #6b7177   secondary text
--faint     #9aa0a6   tertiary text, captions, attribution
--hairline  #e2e3e5   borders
--paper     #fbfbfa   page background
--panel     #ffffff   card/rail/panel background
--rest      #b4b8bc   inactive/resting state (badges, lines)
--error     #a50813   alert/error state — override per project if a more
                       fitting color exists (this one is Tranvías' brand red,
                       borrowed for Bus Works specifically, not a lab color)
```

**The chrome is monochrome on purpose.** There's no fixed "brand accent"
color used decoratively anywhere in the masthead, panel, or UI text — color
is reserved entirely for the data itself (bus line colors, ownership
categories, whatever the tool is showing). That restraint *is* the lab's
visual identity. Don't add a signature hue to make a tool "feel branded" —
if every tool's chrome looks the same and only the data colors differ,
that's the system working correctly.

Typography: `Inter, "Helvetica Neue", Arial, system-ui, sans-serif` — no web
font is loaded. If the visitor has Inter installed, they see it; otherwise a
clean system fallback. Deliberate: one less network request.

## Structural components

- **Masthead** — gradient fade (not a card), flush top-left. Lab wordmark
  links to cidadelabs.org, tool name, one-line description, optional italic
  note for a caveat (Bus Works uses it for "these buses are simulated").
- **Language switcher** — plain text buttons, not a bordered pill. State
  lives in `?lang=` + localStorage, not separate `/es/` `/en/` pages — one
  HTML file per tool, not three.
- **Rail** *(optional)* — persistent scrollable list docked right, for
  tools with a natural "browse everything" list (bus lines; candidate use:
  a list of catchment zones for the área de influencia tool). Delete if the
  tool doesn't need one.
- **Callout, default: native MapLibre popup** — small bubble anchored to the
  click point. Use for 1–3 short facts (a line number and destination; a
  stop name and code).
- **Callout, variant: docked panel** — same tokens, corner radius, and
  shadow language as the rail (not the lighter popup shadow — it's a
  structural surface, not a transient bubble). Use when a single click needs
  to show more than a popup comfortably holds: several fields, an embedded
  image. Delete if the tool never needs more than the popup.
- **Toast** — centered loading/error state, shown before data arrives.
- **Camera** — a deliberate default view (`center`/`zoom`, currently A Coruña
  city center at zoom 14), not fit to each tool's data and not fit to the
  whole `SERVICE_AREA` box either — the box is purely the outer panning
  limit via `maxBounds`, and `minZoom` (currently 12) separately caps how far
  out a user can zoom regardless of position. Edit `SERVICE_AREA` (plain
  north/south/east/west numbers, not MapLibre's `[lng,lat]` corner format)
  to change the panning limit per project; edit `center`/`zoom`/`minZoom` on
  the map constructor to change what the user sees and how far they can
  zoom out. Pad `SERVICE_AREA` generously: a too-tight box was the cause of
  an earlier bug where `maxBounds` and a per-tool `fitBounds` call fought
  each other.
- **Zoom control** — bottom-left, desktop only (hidden under the same mobile
  breakpoint as the rail). Bottom-left specifically because the rail
  occupies the entire right edge as a sibling element with its own stacking
  position — placing the control at bottom-right, MapLibre's default, means
  the rail silently covers it. Touch devices get pinch-to-zoom instead.

## Stack conventions

Static HTML, no build step, no framework. MapLibre GL JS 4.7.1 via
`cdnjs.cloudflare.com` (pinned version, not a floating `@latest`). Basemap:
CARTO Positron (`basemaps.cartocdn.com/gl/positron-gl-style/style.json`,
free, no key). Rotation disabled (`dragRotate:false`,
`pitchWithRotate:false`, `touchZoomRotate.disableRotation()`) — these are
utility tools, not exploratory 3D maps. `attributionControl:false` on the
map itself; attribution lives in the rail's footer (or a minimal footer
line if there's no rail) rather than a floating box. A `ResizeObserver` on
`#map` calls `map.resize()` whenever the container's box changes — keep
this. Without it, a same-tab navigation into the tool (as opposed to a hard
refresh or a fresh tab) can construct the map before layout has settled,
permanently baking a clipped canvas size into the WebGL drawing buffer.

Trilingual: Galician default, then Spanish, English — UI strings only,
never feed-native content (place names, addresses stay as the source gives
them).

Data: cite sources, document method, prefer open data. GeoJSON is the
standard interchange format between a data pipeline and the map.

## Starting a new tool

1. Copy `template.html` into a new `cidade-labs/<tool-name>` repo.
2. Fill in `I18N` strings, `STORAGE_KEY`, and the masthead title/description.
3. Delete the rail and/or panel blocks if the tool doesn't need them.
4. Build the data pipeline (source → clean GeoJSON) separately, documented
   in its own script with a header comment — see Bus Works' or MisColes'
   `scripts/` for the pattern.
5. Wire up the `TOOL-SPECIFIC` sections: sources, layers, popup/panel
   content.
6. Deploy independently (GitHub Pages), link from cidadelabs.org.
