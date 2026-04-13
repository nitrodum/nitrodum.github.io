# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a static single-page photography portfolio site (`index.html`) for Derek Kwok, deployed via GitHub Pages. There is no build step, no package manager, and no framework — everything lives in one self-contained HTML file.

## Development

Open `index.html` directly in a browser (or use a local static server like `npx serve .`) to preview. There is no build, lint, or test pipeline.

## Architecture

The entire application is a single `index.html` with inline CSS and JavaScript — no external JS libraries, no bundler.

### Data layer (top of `<script>`)
- `PROJ` — project categories with display name and color (portraits, weddings, landscapes, commercial, fineart)
- `CLUSTER_CENTRES` — canvas coordinates where each project group gravitates toward on the 10000×10000 virtual canvas
- `PHOTOS` — array of photo objects with id, proj, title, desc, meta (loc/yr/cam/fmt), tags, rel (related photo IDs), and a seeded thumbnail placeholder
- `LINKS` — explicit spring connections between photos driving the physics graph

### Physics simulation
A custom force-directed graph runs every animation frame via `requestAnimationFrame`. Forces applied per tick:
1. **Repulsion** — O(n²) charge repulsion between all node pairs (`CFG.repulse`)
2. **Link springs** — spring force along `LINKS` edges toward `CFG.ldist` rest distance
3. **Tag attraction** — when a node is selected, tag-siblings are pulled toward it (`TAG_PULL_DIST`, `TAG_PULL_STR`)
4. **Cluster gravity** — each node is pulled toward its project's `CLUSTER_CENTRES` entry
5. **Global center gravity** — weak pull toward canvas center
6. **Collision** — prevents node overlap

`CFG` holds live physics config; `DEFAULTS` and `PRESETS` (balanced/tight/loose/orbit/frozen) are the named configurations. Node sizes scale with connection count (`nodeW`/`nodeH`/`nodeR`).

### Canvas / viewport
The canvas is a 10000×10000 px absolutely-positioned `#canvas` div inside `#wrap`. Pan/zoom state is `tx`, `ty`, `sc`. Node DOM elements are positioned absolutely and moved by updating `left`/`top` style. SVG lines for edges are drawn in an overlay `#svg`.

### UI panels
- **Nav** (`#nav`) — project filter pills, tag active indicator, About/Contact buttons, zoom controls
- **Detail panel** (`#panel`) — slides in from right on photo click; shows full metadata, tags, and related photos
- **Physics panel** (`#phys-panel`) — slides in from right via toggle button; sliders/number inputs bound two-way to `CFG`
- **About overlay** (`#about-ov`) and **Contact overlay** (`#contact-ov`) — full-screen overlays with blur backdrop

### Interaction model
- Canvas pan: pointer drag on `#wrap`
- Canvas zoom: wheel + keyboard `+`/`-`/`F`/`R`
- Node click: opens detail panel, dims unrelated nodes, triggers tag attraction
- Node drag: fixes node position (`NS[id].fixed = true`) until released
- Filter pills: filter visible nodes by project; clicking a tag in the detail panel filters by tag

### Placeholder images
Photos are rendered as colored gradient placeholders seeded from `photo.seed` using a simple `mulberry32` PRNG — no actual image files are required. Replace with real `<img src>` by setting `photo.img` on each entry.

## Customization

To add photos: add entries to `PHOTOS`, add connections to `LINKS`, and (optionally) tune `CLUSTER_CENTRES`. To add a project category: add to `PROJ` and `CLUSTER_CENTRES`. The contact form uses Netlify Forms (`netlify` attribute on `<form>`).
