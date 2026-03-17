# zwift-racing-hud

A browser-based racing overlay for Zwift, designed for ultrawide monitors. Two standalone HTML panels sit side-by-side and pull live data from [Sauce for Zwift](https://www.sauce.llc/) via its local REST API.

> TSS®, Normalized Power® (NP®), and Intensity Factor® (IF®) are registered trademarks of TrainingPeaks, LLC, used here for display purposes only. This project is not affiliated with TrainingPeaks.

---

## Panels

### `left-panel.html`
- Live power with zone coloring, w/kg, and IF (Intensity Factor)
- W' balance gauge with color-coded bar
- Heart rate with zone coloring, cadence, speed, draft
- Trip stats: distance, elapsed time, laps, distance/time remaining
- TSS (Training Stress Score) and estimated calories (Keytel formula)
- Power peaks: 5s / 15s / 30s / 1m / 5m / 20m — each with w/kg and peak HR
- Rolling 5-minute chart: zone-colored power fill, smoothed line, HR overlay, FTP reference line
- Nearby riders table: gap, watts, w/kg, W' balance bar

### `right-panel.html`
- Race position, gap to rider ahead, gap to rider behind (lap-aware)
- Course map, groups visualization, and chat via embedded Sauce pages

---

## Architecture

Both panels are single self-contained HTML files with no build step, no framework, and no external dependencies beyond Google Fonts. All logic is vanilla JavaScript.

```
zwift-racing-hud/
├── pages/
│   ├── left-panel.html
│   └── right-panel.html
├── manifest.json
└── README.md
```

### Data flow

```
Sauce for Zwift (localhost:1080)
        │
        │  REST polling, 1s interval
        ▼
  Browser (Chrome)
  ├── left-panel.html   →  nearby/v2, athlete/v2/watching, athlete/v1/:id
  └── right-panel.html  →  nearby/v2, athlete/v1/:id (+ Sauce embedded pages)
```

- **No WebSocket.** Both panels use `setInterval` + `fetch` at 1-second intervals.
- **No shared state** between panels. Each polls independently.
- **Athlete cache.** Names and weights are fetched once per athlete via `athlete/v1/:id` and cached in memory for the session.

### API endpoints used

| Endpoint | Panel | Purpose |
|---|---|---|
| `nearby/v2` | Both | Rider positions, gaps, power, W' balance |
| `athlete/v2/watching` | Left | Self telemetry: power, HR, cadence, speed, peaks |
| `athlete/v1/:id` | Both | Athlete name and weight (cached per session) |

The right panel also embeds Sauce's built-in pages (`geo.html`, `groups.html`, `chat.html`) directly via iframes.

---

## Requirements

- [Sauce for Zwift](https://www.sauce.llc/) installed and running
- Chrome (recommended) or any modern browser
- Zwift running on the same machine, or Sauce accessible on the local network

---

## Setup

### Quickstart (no server needed)

1. Start Zwift and make sure Sauce for Zwift is running
2. Open `pages/left-panel.html` in Chrome — drag to the left side of your monitor
3. Open `pages/right-panel.html` in Chrome — drag to the right side
4. Both panels default to `localhost:1080` and connect automatically

No install, no server, no build step.

### If Sauce is running on a different machine

Edit the host field in the connection bar at the top of either panel, or change the default in the HTML:

```html
<input id="sauce-host" ... value="192.168.86.X:1080" ...>
```

### Rider inputs (left panel)

The **Self** section header contains three inputs used for HR zone coloring and calorie estimation:

| Field | Default | Purpose |
|---|---|---|
| MaxHR | 172 | HR zone thresholds (scales proportionally) |
| Age | 40 | Keytel calorie formula |
| Gender | M | Keytel calorie formula |

These are not persisted — update them at the start of each session.

---

## Design system

Both panels share an identical set of CSS variables and typography.

| Token | Value | Usage |
|---|---|---|
| `--bg` | `#080b0f` | Page background |
| `--bg2` | `#0d1117` | Section headers, footer |
| `--bg3` | `#111820` | Input fields, inner fills |
| `--border` | `#2a3f54` | Borders, gaps between cells |
| `--accent` | `#00aaff` | Self highlight, active states |
| `--green` | `#00e87a` | Positive / behind / healthy W' |
| `--red` | `#ff2d55` | Negative / ahead / depleted W' |
| `--yellow` | `#f5c400` | Warnings, power-ups, selected rider |

Power zone colors: Z1 `#555566` · Z2 `#4488ff` · Z3 `#00e87a` · Z4 `#f5c400` · Z5 `#ff7a00` · Z6+ `#ff2d55`

Fonts: **Roboto Mono** (numbers and data values) + **Barlow Condensed** (labels and badges), loaded from Google Fonts.

---

## Notes and limitations

- **FTP** is read from the Sauce API (`athlete.ftp`). IF, NP, and TSS will show `—` until this value is available and 30+ seconds of ride data have accumulated.
- **Calories** use current HR as a proxy for average effort — accurate over steady-state efforts, less so for highly variable intensity.
- **Peak HR** per duration is calculated from the recorded HR history (5-minute window). Short peaks (5s, 15s, 30s) populate quickly; longer peaks require sufficient ride time.
- **Peaks** display `—` until elapsed ride time exceeds the peak duration — no premature values.
- No persistent settings — host and rider inputs reset on page reload.

---

## License

MIT — see [LICENSE](LICENSE).

This project is not affiliated with or endorsed by Zwift or Sauce for Zwift.