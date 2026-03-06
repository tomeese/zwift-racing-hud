# zwift-racing-hud

A browser-based racing overlay for Zwift, designed for ultrawide monitors. Two standalone HTML panels sit side-by-side and pull live data from [Sauce for Zwift](https://www.sauce.llc/) via its local REST API.

---

## Panels

### `index.html` — Left Panel
- Live power, W' balance gauge and bar
- Heart rate, cadence, speed, draft
- Trip stats: distance, elapsed time, distance/time remaining
- Power peaks: 5s, 15s, 1m, 5m, 20m
- Rolling 5-minute chart: power, HR, cadence
- Nearby riders table with gap, w/kg, W' balance

### `right.html` — Right Panel
- Race position, gap to rider ahead, gap to rider behind
- Mini-map with three rendering modes (see below)
- Groups visualization: gap between groups, avg w/kg, size, your position within your group
- In-game chat feed (if exposed by Sauce API)

---

## Architecture

Both panels are single self-contained HTML files with no build step, no framework, and no external dependencies beyond Google Fonts. All logic is vanilla JavaScript.

```
zwift-racing-hud/
├── index.html     # Left panel
├── right.html     # Right panel
└── README.md
```

<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 900 560" width="900" height="560" font-family="'Segoe UI', Arial, sans-serif">
  <defs>
    <marker id="arrow" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto">
      <path d="M0,0 L0,6 L8,3 z" fill="#5a7a96"/>
    </marker>
    <marker id="arrow-accent" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto">
      <path d="M0,0 L0,6 L8,3 z" fill="#00aaff"/>
    </marker>
    <marker id="arrow-green" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto">
      <path d="M0,0 L0,6 L8,3 z" fill="#00e87a"/>
    </marker>
    <marker id="arrow-dim" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto">
      <path d="M0,0 L0,6 L8,3 z" fill="#2a3f54"/>
    </marker>
    <filter id="glow-accent">
      <feGaussianBlur stdDeviation="3" result="blur"/>
      <feMerge><feMergeNode in="blur"/><feMergeNode in="SourceGraphic"/></feMerge>
    </filter>
    <filter id="glow-green">
      <feGaussianBlur stdDeviation="2.5" result="blur"/>
      <feMerge><feMergeNode in="blur"/><feMergeNode in="SourceGraphic"/></feMerge>
    </filter>
  </defs>

  <!-- Background -->
  <rect width="900" height="560" fill="#080b0f"/>

  <!-- Title -->
  <text x="450" y="38" text-anchor="middle" font-size="15" font-weight="600" fill="#8aafc8" letter-spacing="3" font-family="'Segoe UI', Arial, sans-serif">ZWIFT RACING HUD — ARCHITECTURE</text>
  <line x1="60" y1="50" x2="840" y2="50" stroke="#2a3f54" stroke-width="1"/>

  <!-- ── INTERNET CLOUD ── -->
  <g id="cloud">
    <ellipse cx="450" cy="105" rx="90" ry="38" fill="#0d1117" stroke="#2a3f54" stroke-width="1.5" stroke-dasharray="5,3"/>
    <text x="450" y="101" text-anchor="middle" font-size="11" fill="#5a7a96" font-weight="600" letter-spacing="1">INTERNET</text>
    <text x="450" y="116" text-anchor="middle" font-size="10" fill="#2a3f54">Zwift Servers</text>
  </g>

  <!-- ── ZWIFT PC BOX ── -->
  <g id="zwift-pc">
    <rect x="60" y="190" width="260" height="310" rx="4" fill="#0d1117" stroke="#2a3f54" stroke-width="1.5"/>
    <rect x="60" y="190" width="260" height="28" rx="4" fill="#111820"/>
    <rect x="60" y="204" width="260" height="14" fill="#111820"/>
    <text x="190" y="209" text-anchor="middle" font-size="10" font-weight="700" fill="#5a7a96" letter-spacing="2">ZWIFT PC</text>

    <!-- Zwift app -->
    <rect x="80" y="234" width="220" height="52" rx="3" fill="#080b0f" stroke="#2a3f54" stroke-width="1"/>
    <text x="190" y="253" text-anchor="middle" font-size="11" font-weight="600" fill="#d8eaf8">Zwift</text>
    <text x="190" y="269" text-anchor="middle" font-size="10" fill="#5a7a96">game client</text>

    <!-- Divider -->
    <line x1="80" y1="302" x2="300" y2="302" stroke="#1e2d3d" stroke-width="1"/>
    <text x="190" y="320" text-anchor="middle" font-size="9" fill="#2a3f54" letter-spacing="1.5">BROWSER WINDOWS</text>

    <!-- Left panel browser -->
    <rect x="80" y="330" width="100" height="78" rx="3" fill="#080b0f" stroke="#00aaff" stroke-width="1" opacity="0.7"/>
    <rect x="80" y="330" width="100" height="14" rx="3" fill="#111820"/>
    <rect x="80" y="340" width="100" height="4" fill="#111820"/>
    <text x="130" y="342" text-anchor="middle" font-size="8" fill="#5a7a96" letter-spacing="1">CHROME</text>
    <text x="130" y="360" text-anchor="middle" font-size="10" font-weight="600" fill="#00aaff">index.html</text>
    <text x="130" y="374" text-anchor="middle" font-size="9" fill="#5a7a96">Left Panel</text>
    <text x="130" y="388" text-anchor="middle" font-size="8" fill="#2a3f54">power · HR · nearby</text>

    <!-- Right panel browser -->
    <rect x="200" y="330" width="100" height="78" rx="3" fill="#080b0f" stroke="#00aaff" stroke-width="1" opacity="0.7"/>
    <rect x="200" y="330" width="100" height="14" rx="3" fill="#111820"/>
    <rect x="200" y="340" width="100" height="4" fill="#111820"/>
    <text x="250" y="342" text-anchor="middle" font-size="8" fill="#5a7a96" letter-spacing="1">CHROME</text>
    <text x="250" y="360" text-anchor="middle" font-size="10" font-weight="600" fill="#00aaff">right.html</text>
    <text x="250" y="374" text-anchor="middle" font-size="9" fill="#5a7a96">Right Panel</text>
    <text x="250" y="388" text-anchor="middle" font-size="8" fill="#2a3f54">position · map · groups</text>

    <!-- Monitor label -->
    <rect x="80" y="422" width="220" height="22" rx="2" fill="#080b0f" stroke="#1e2d3d" stroke-width="1"/>
    <text x="190" y="437" text-anchor="middle" font-size="9" fill="#2a3f54" letter-spacing="1">ULTRAWIDE MONITOR</text>

    <!-- bracket lines connecting panels to monitor label -->
    <line x1="130" y1="408" x2="130" y2="422" stroke="#1e2d3d" stroke-width="1"/>
    <line x1="250" y1="408" x2="250" y2="422" stroke="#1e2d3d" stroke-width="1"/>
    <line x1="130" y1="422" x2="250" y2="422" stroke="#1e2d3d" stroke-width="1"/>
  </g>

  <!-- ── SAUCE LAPTOP BOX ── -->
  <g id="sauce-laptop">
    <rect x="580" y="190" width="260" height="200" rx="4" fill="#0d1117" stroke="#2a3f54" stroke-width="1.5"/>
    <rect x="580" y="190" width="260" height="28" rx="4" fill="#111820"/>
    <rect x="580" y="204" width="260" height="14" fill="#111820"/>
    <text x="710" y="209" text-anchor="middle" font-size="10" font-weight="700" fill="#5a7a96" letter-spacing="2">LAPTOP</text>

    <!-- Sauce app -->
    <rect x="600" y="234" width="220" height="70" rx="3" fill="#080b0f" stroke="#00e87a" stroke-width="1" opacity="0.8"/>
    <text x="710" y="257" text-anchor="middle" font-size="12" font-weight="700" fill="#00e87a" filter="url(#glow-green)">Sauce for Zwift</text>
    <text x="710" y="274" text-anchor="middle" font-size="10" fill="#5a7a96">telemetry processor</text>
    <text x="710" y="289" text-anchor="middle" font-size="9" fill="#2a3f54">w/kg · W' balance · groups · gaps</text>

    <!-- REST API -->
    <rect x="600" y="320" width="220" height="52" rx="3" fill="#080b0f" stroke="#2a3f54" stroke-width="1"/>
    <text x="710" y="341" text-anchor="middle" font-size="11" font-weight="600" fill="#d8eaf8">REST API</text>
    <text x="710" y="357" text-anchor="middle" font-size="10" fill="#5a7a96">localhost:1080/api/</text>
    <text x="710" y="370" text-anchor="middle" font-size="9" fill="#2a3f54">nearby · groups · athlete · chat</text>
  </g>

  <!-- ── APPLE TV OPTIONAL BOX ── -->
  <g id="appletv">
    <rect x="580" y="420" width="260" height="80" rx="4" fill="#0d1117" stroke="#2a3f54" stroke-width="1.5" stroke-dasharray="6,3"/>
    <text x="710" y="448" text-anchor="middle" font-size="10" font-weight="700" fill="#5a7a96" letter-spacing="2">APPLE TV  <tspan font-size="9" fill="#2a3f54">(optional)</tspan></text>
    <text x="710" y="466" text-anchor="middle" font-size="11" font-weight="600" fill="#d8eaf8">Zwift</text>
    <text x="710" y="482" text-anchor="middle" font-size="9" fill="#5a7a96">same local network as laptop</text>
  </g>

  <!-- ── ARROWS ── -->

  <!-- Zwift PC ↔ Zwift Servers (internet) -->
  <line x1="190" y1="190" x2="380" y2="132" stroke="#2a3f54" stroke-width="1.5" stroke-dasharray="5,3" marker-end="url(#arrow-dim)"/>
  <line x1="380" y1="118" x2="190" y2="176" stroke="#2a3f54" stroke-width="1.5" stroke-dasharray="5,3" marker-end="url(#arrow-dim)"/>

  <!-- Sauce Laptop ↔ Zwift Servers (internet) -->
  <line x1="710" y1="190" x2="528" y2="132" stroke="#00e87a" stroke-width="1.5" stroke-dasharray="5,3" marker-end="url(#arrow-green)" opacity="0.6"/>
  <line x1="528" y1="118" x2="710" y2="176" stroke="#00e87a" stroke-width="1.5" stroke-dasharray="5,3" marker-end="url(#arrow-green)" opacity="0.6"/>

  <!-- Sauce REST API → index.html (poll) -->
  <path d="M580,340 C500,340 380,360 300,365" fill="none" stroke="#00aaff" stroke-width="1.5" marker-end="url(#arrow-accent)" opacity="0.8"/>
  <text x="430" y="335" text-anchor="middle" font-size="9" fill="#00aaff" opacity="0.7">REST poll · 1s</text>

  <!-- Sauce REST API → right.html (poll) -->
  <path d="M580,355 C520,355 400,385 300,378" fill="none" stroke="#00aaff" stroke-width="1.5" marker-end="url(#arrow-accent)" opacity="0.6"/>

  <!-- Apple TV ↔ Sauce (local network) -->
  <line x1="710" y1="420" x2="710" y2="390" stroke="#5a7a96" stroke-width="1.5" stroke-dasharray="4,3" marker-end="url(#arrow)"/>
  <text x="730" y="410" font-size="9" fill="#5a7a96">local network</text>

  <!-- Legend -->
  <g transform="translate(60, 510)">
    <line x1="0" y1="8" x2="28" y2="8" stroke="#00aaff" stroke-width="1.5" marker-end="url(#arrow-accent)"/>
    <text x="34" y="12" font-size="9" fill="#5a7a96">REST polling (1s)</text>

    <line x1="140" y1="8" x2="168" y2="8" stroke="#00e87a" stroke-width="1.5" stroke-dasharray="5,3" marker-end="url(#arrow-green)" opacity="0.7"/>
    <text x="174" y="12" font-size="9" fill="#5a7a96">Zwift telemetry</text>

    <line x1="280" y1="8" x2="308" y2="8" stroke="#2a3f54" stroke-width="1.5" stroke-dasharray="5,3" marker-end="url(#arrow-dim)"/>
    <text x="314" y="12" font-size="9" fill="#5a7a96">game traffic</text>

    <line x1="410" y1="8" x2="438" y2="8" stroke="#5a7a96" stroke-width="1.5" stroke-dasharray="4,3" marker-end="url(#arrow)"/>
    <text x="444" y="12" font-size="9" fill="#5a7a96">optional / local network</text>
  </g>
</svg>

### Data flow

```
Sauce for Zwift (localhost:1080)
        │
        │  REST polling, 1s interval
        ▼
  Browser (Chrome)
  ├── index.html  →  nearby/v2, athlete/v2/watching
  └── right.html  →  nearby/v2, groups/v2, chat (probed)
```

- **No WebSocket.** Both panels use `setInterval` + `fetch` at 1-second intervals.
- **No shared state** between panels. Each polls independently.
- **Athlete cache.** Names and weights are fetched once per athlete via `athlete/v1/:id` and cached in memory for the session.

### API endpoints used

| Endpoint | Panel | Purpose |
|---|---|---|
| `nearby/v2` | Both | Rider positions, gaps, power, W' balance |
| `athlete/v2/watching` | Left | Self telemetry: power, HR, cadence, speed, peaks |
| `groups/v2` | Right | Race group detection |
| `athlete/v1/:id` | Both | Athlete name and weight (cached) |
| `chat/v1` (probed) | Right | In-game chat feed |
| `worlds/v1` et al (probed) | Right | Route shape for mini-map |

### Mini-map rendering

`right.html` probes for route geometry on connect and falls back gracefully:

1. **Route map** — if a route shape endpoint returns a point array, draws the course as a polyline and plots rider dots on it. Supports `{x,y}`, `{lat,lng}`, and `[lng,lat]` coordinate formats. Riders with a `roadPosition` field (0–1 fraction) are interpolated onto the route.
2. **Scatter** — if no route shape is found but riders have `x`/`y` world coordinates, renders a dot-plot.
3. **Gap chart** — horizontal linear fallback. Left = ahead (red), right = behind (green). Your position is centered. Riders within 20 seconds get their gap labelled.

The active mode is shown in the bottom-right corner of the map panel.

---

## Requirements

- [Sauce for Zwift](https://www.sauce.llc/) installed and running
- Chrome (recommended) or any modern browser
- Zwift running on the same machine, or Sauce accessible on the local network

---

## Setup

### Quickstart (no server needed)

1. Start Zwift and make sure Sauce for Zwift is running
2. Open `index.html` in Chrome — drag to the left half of your monitor
3. Open `right.html` in Chrome — drag to the right half
4. Both panels default to `localhost:1080` and connect automatically

That's it. No install, no server, no build step.

### If Sauce is running on a different machine

Edit the host field in the connection bar at the top of either panel, or change the default value in the HTML:

```html
<input id="sauce-host" ... value="192.168.86.X:1080" ...>
```

### Dev iteration

```bash
npx live-server --port=3000              # left panel with auto-reload
npx live-server --port=3001 --entry-file=right.html  # right panel
```

---

## Design system

Both panels share an identical set of CSS variables and typography. Do not override these in one panel without updating the other.

| Token | Value | Usage |
|---|---|---|
| `--bg` | `#080b0f` | Page background |
| `--bg2` | `#0d1117` | Section headers, footer |
| `--bg3` | `#111820` | Input fields, inner fills |
| `--border` | `#2a3f54` | Borders, gaps between cells |
| `--accent` | `#00aaff` | Self highlight, active states |
| `--green` | `#00e87a` | Positive / behind / healthy |
| `--red` | `#ff2d55` | Negative / ahead / danger |
| `--yellow` | `#f5c400` | Warnings, power-ups, selected |

Fonts: **Roboto Mono** (numbers and code values) + **Barlow Condensed** (labels and badges), both loaded from Google Fonts. System font fallbacks are defined for offline use.

---

## Distribution

To share with others:

1. Zip `index.html`, `right.html`, and this `README.md`
2. Recipients open both files directly in Chrome — no server required
3. Sauce for Zwift defaults to `localhost:1080` so no configuration is needed for most users

---

## Known limitations / future work

- Groups `v2` endpoint shape is version-dependent — field mapping may need adjustment based on your Sauce version
- Chat endpoint is probed but not confirmed available on all Sauce builds
- Route geometry endpoints are probed opportunistically — map falls back to gap chart if none respond
- No persistent settings — host preference resets on page reload