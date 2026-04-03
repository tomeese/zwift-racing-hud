# zwift-racing-hud

A browser-based racing overlay for Zwift, designed for ultrawide monitors. Two standalone HTML panels sit side-by-side and pull live data from [Sauce for Zwift](https://www.sauce.llc/) via its local REST API.

> TSS®, Normalized Power® (NP®), and Intensity Factor® (IF®) are registered trademarks of TrainingPeaks, LLC, used here for display purposes only. This project is not affiliated with TrainingPeaks.

---

## Panels

### `left_panel.html`
- Live power with Coggan zone coloring, w/kg
- W' balance gauge with color-coded fill bar
- Heart rate with zone coloring scaled to your MaxHR
- Cadence, speed, draft
- Trip stats: distance, elapsed time, laps, distance/time remaining (switches to meters under 1km)
- TSS, IF, kJ, and estimated calories (Keytel formula) — accumulated per second
- Power peaks: 5s / 15s / 1m / 5m / 20m — each with w/kg and peak HR
- Peaks suppressed until elapsed ride time exceeds the peak duration
- Cumulative HR and power zone distribution bars
- Full-width 10-minute chart: zone-colored power, HR overlay, cadence line, FTP reference
- Three-section resizable layout — drag handles between TOP / CHART / NEARBY; positions persist across reloads
- Settings modal (⚙) for FTP, MaxHR, Age, Gender, and Sauce host — all fields save history to localStorage
- Nearby riders table: gap, watts, w/kg, W' balance — with rider detail panel on click and event/bot filters

### `right_panel.html` — Right Panel
- Race position with lap-aware sorting
- Gap to rider ahead and rider behind with names
- Course map, groups visualization, and chat via embedded Sauce pages

---

## Architecture

All panels are single self-contained HTML files with no build step, no framework, and no external dependencies beyond Google Fonts. All logic is vanilla JavaScript.

```
zwift-racing-hud/
├── pages/
│   ├── left_panel.html
│   └── right_panel.html
├── manifest.json
├── README.md
└── LICENSE
```

### Data flow

```
Sauce for Zwift (localhost:1080)
        │
        │  REST polling, 1s interval
        ▼
  Browser (Chrome)
  ├── left_panel.html    →  nearby/v2, athlete/v2/watching, athlete/v1/:id
  └── right_panel.html   →  nearby/v2, athlete/v1/:id (+ Sauce embedded pages)
```

- **No WebSocket.** All panels use `setInterval` + `fetch` at 1-second intervals.
- **No shared state** between panels. Each polls independently.
- **Failure tolerance.** Panels tolerate up to 4 consecutive failed polls before showing disconnected. Each API endpoint is fetched independently — a slow nearby response won't stop self telemetry from updating.
- **Athlete cache.** Names and weights are fetched once per athlete via `athlete/v1/:id` and cached in memory for the session.

### API endpoints used

| Endpoint | Panel | Purpose |
|---|---|---|
| `nearby/v2` | All | Rider positions, gaps, power, W' balance |
| `athlete/v2/watching` | Left panels | Self telemetry: power, HR, cadence, speed, peaks |
| `athlete/v1/:id` | All | Athlete name and weight (cached per session) |

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
2. Open `pages/left_panel.html` (or `left_panel_v2.html`) in Chrome
3. Open `pages/right_panel.html` in Chrome
4. Both panels default to `localhost:1080` and connect automatically

No install, no server, no build step.

### If Sauce is running on a different machine

**v1:** Edit the host field in the connection bar at the top of the panel.

**v2:** Click the ⚙ button to open settings, enter the host, and click Connect. The host is saved and will be available in the dropdown on future loads.

### Rider settings

Both panels require a few rider-specific values for accurate calculations:

| Field | Default | Purpose |
|---|---|---|
| FTP | auto (from API) | TSS, IF, NP, zone coloring |
| MaxHR | 172 bpm | HR zone thresholds (scales proportionally to your max) |
| Age | 40 | Keytel calorie formula |
| Gender | M | Keytel calorie formula |

**v1:** These appear as inputs in the Self section header. They reset on page reload.

**v2:** These live in the Settings modal (⚙). All values are saved to localStorage and persist across reloads. Previously entered values appear in a dropdown for quick re-selection.

---

## Calculations

### Normalized Power (NP)
Standard algorithm: 30-second rolling average → raise each value to the 4th power → mean → 4th root. Requires 30+ seconds of ride data. Zero-power samples (coasting) are included in the denominator to match how Zwift and Garmin calculate NP.

### Intensity Factor (IF)
`NP / FTP`. Zone-colored using the same Coggan thresholds as power.

### TSS
`(elapsed_seconds × IF² × 100) / 3600`. Updates every second.

### kJ
`avg_power × elapsed_seconds / 1000`. Matches what Zwift reports.

### Calories
Keytel et al. (2005) formula using current HR, weight (from API), age, and gender. Accumulated per second rather than recalculated from total time, so the value is strictly monotonically increasing.

### Power zones (Coggan)
| Zone | % of FTP |
|---|---|
| Z1 | < 55% |
| Z2 | 55–75% |
| Z3 | 75–90% |
| Z4 | 90–105% |
| Z5 | 105–120% |
| Z6+ | > 120% |

### HR zones
Scaled proportionally to MaxHR:

| Zone | Description | % of MaxHR |
|---|---|---|
| Z1 | Recovery | < 60% |
| Z2 | Aerobic | 60–69% |
| Z3 | Tempo | 69–78% |
| Z4 | Threshold | 78–87% |
| Z5 | VO2 Max | > 87% |

---

## Design system

Both panels share an identical set of CSS variables and typography.

| Token | Value | Usage |
|---|---|---|
| `--bg` | `#080b0f` | Page background |
| `--bg2` | `#0d1117` | Section headers, cells |
| `--bg3` | `#111820` | Input fields, inner fills |
| `--border` | `#2a3f54` | Borders, gaps between cells |
| `--accent` | `#00aaff` | Self highlight, active states |
| `--green` | `#00e87a` | Positive / behind / healthy W' |
| `--red` | `#ff2d55` | Negative / ahead / depleted W' |
| `--yellow` | `#f5c400` | Warnings, power-ups, selected rider |

Power zone colors: Z1 `#555566` · Z2 `#4488ff` · Z3 `#00e87a` · Z4 `#f5c400` · Z5 `#ff7a00` · Z6+ `#ff2d55`

Fonts: **Roboto Mono** (numbers and data values) + **Barlow Condensed** (labels and badges), loaded from Google Fonts.

---

## Chart overlays (left panels)

The rolling power chart renders three overlapping data series:

- **Power** — zone-colored fill and smoothed line, fixed scale 0–750w (values above 750w flat-line at the top), 7-point smooth
- **HR** — zone-colored fill and line, fixed scale 0 to MaxHR+10, 25-point smooth
- **Cadence** — white line at 45% opacity, fixed scale 0–150rpm, 5-point smooth

All three use fixed scales so the chart never rescales mid-ride.

---

## Notes and limitations

- **FTP** is read from the Sauce API (`athlete.ftp`). IF, NP, TSS, and zone coloring will show `—` until this value is available and 30+ seconds of data have accumulated. Override by entering FTP manually in settings.
- **Calories** use current HR as a proxy for effort intensity — accurate over steady-state efforts, less so during highly variable efforts like sprints.
- **Peak HR** per duration is calculated from the recorded HR history window. Longer peaks (5m, 20m) require sufficient ride time.
- **Position** uses `eventPosition` from the API when available, falling back to gap-sorted position within the nearby array. The nearby array only contains riders within a proximity radius, so gap-sorted position may undercount in large events.
- **v2 layout** persists section heights as percentages of total window height, so they scale correctly on window resize.
- Font sizes do not scale dynamically when sections are resized.

---

## License

MIT — see [LICENSE](LICENSE).

This project is not affiliated with or endorsed by Zwift or Sauce for Zwift.
