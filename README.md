# Aviation Safety Analytics & Anomaly Detection Dashboard

Real-time aviation safety monitoring dashboard that tracks live flight data over Turkish airspace and detects anomalies indicating potentially dangerous situations.

![Dashboard Preview](screenshots/dashboard-preview.png)

---

## About

### What does this project do?
This dashboard connects to the [OpenSky Network](https://opensky-network.org/) API to fetch **real-time ADS-B flight data** (position, altitude, speed, vertical rate, squawk codes) for aircraft flying over Turkish airspace. It processes this data through a **rule-based anomaly detection engine** that identifies potentially dangerous flight conditions — such as rapid altitude drops, excessive speeds, abnormal vertical rates, and emergency transponder codes — and visualizes everything on an interactive live map with a real-time alert feed.

### Why was it built?
Aviation safety depends on continuous monitoring and early detection of abnormal flight behavior. This project demonstrates how publicly available ADS-B data can be leveraged to build a lightweight, browser-based safety monitoring tool — without requiring expensive commercial flight tracking subscriptions or specialized hardware. It serves as a proof-of-concept for real-time data ingestion, anomaly detection, and data visualization in the aviation domain.

### Technologies Used
- **OpenSky Network REST API** — Real-time ADS-B flight data (free, open-source)
- **Leaflet.js** — Interactive map with CartoDB Dark Matter tiles
- **Chart.js** — Statistical data visualization
- **Vanilla JavaScript** — Anomaly detection engine, data processing, DOM rendering
- **HTML5 / CSS3** — Responsive dark-themed UI (air traffic control aesthetic)
- **Python http.server** — Zero-dependency local server for CORS compliance

---

## Quick Start (3 Steps)

```bash
# 1. Clone the repository
git clone https://github.com/zeynepsusosun/aviation-safety-analytics.git
cd aviation-safety-analytics

# 2. Start local server (Python 3 required)
python -m http.server 8081

# 3. Open in browser
# Navigate to http://localhost:8081
```

That's it. The dashboard will start fetching real-time flight data from OpenSky Network.

---

## Prerequisites

| Requirement | Version | Check Command | Install |
|---|---|---|---|
| **Web Browser** | Chrome, Firefox, Edge (modern) | — | — |
| **Python 3** | 3.7+ | `python --version` | [python.org/downloads](https://www.python.org/downloads/) |

> **Why Python?** A local HTTP server is needed to avoid CORS restrictions when calling the OpenSky API. Python's built-in `http.server` is the simplest zero-dependency option. Alternatives: `npx serve .` (Node.js) or any other static file server.

---

## Step-by-Step Setup Guide

### 1. Clone or Download

**Option A — Git Clone:**
```bash
git clone https://github.com/zeynepsusosun/aviation-safety-analytics.git
cd aviation-safety-analytics
```

**Option B — Download ZIP:**
1. Go to the GitHub repository page
2. Click **Code** → **Download ZIP**
3. Extract the ZIP to a folder
4. Open a terminal in that folder

### 2. Start the Local Server

```bash
python -m http.server 8081
```

You should see:
```
Serving HTTP on :: port 8081 (http://[::]:8081/) ...
```

> **Port busy?** Change `8081` to any available port (e.g., `8082`, `9000`).

> **Windows:** If `python` doesn't work, try `python3` or `py`.

### 3. Open the Dashboard

Open your browser and navigate to:
```
http://localhost:8081
```

### 4. Verify Real Data

- If you see live aircraft on the map and **no** "SIMULATION MODE" badge → **real data is flowing**
- If you see "SIMULATION MODE" in the header → the API is unreachable (check your internet connection or try again later)

### 5. Stop the Server

Press `Ctrl + C` in the terminal where the server is running.

---

## Features

### Real-Time Flight Map
- Interactive Leaflet.js map with dark CartoDB tiles
- Aircraft markers color-coded by status: green (normal), orange (warning), red (critical)
- Click any aircraft for detailed popup (callsign, altitude, speed, status)
- Auto-refreshes every 15 seconds
- Centered on Turkish airspace (configurable)

### Anomaly Detection Engine
Automated detection of 5 anomaly categories:

| Rule | Condition | Severity |
|---|---|---|
| Rapid Altitude Drop | Descent > 1000 ft/min below 10,000 ft | **Critical** |
| Excessive Speed | Ground speed > 600 knots | **Warning** |
| Abnormal Vertical Rate | Climb/descent > 4000 ft/min | **Warning** |
| Unexpected Low Altitude | Below 500m and not on ground | **Warning** |
| Emergency Squawk | 7500 (hijack), 7600 (radio fail), 7700 (emergency) | **Critical** |

### Alert Feed
- Timestamped warnings with severity color coding
- Auto-scrolling sidebar (max 50 entries)
- Example: `"CRITICAL: TK1234 — Rapid altitude drop (alt: 2400m, vrate: -18 m/s)"`

### Aircraft Data Table
- Sortable by any column (click header)
- Searchable by callsign
- Filterable by country and anomaly status
- Row highlighting matches anomaly severity

### Dashboard Statistics
- Total aircraft tracked
- Normal / Warning / Critical breakdown
- Active alerts count
- Status distribution chart (Chart.js doughnut)

### Simulation Fallback
When the OpenSky API is unreachable (CORS, rate limit, offline), the dashboard automatically switches to simulation mode with 55+ realistic flights over Turkey using actual airline callsign prefixes (TK, PC, XQ, THY, SHT).

---

## Tech Stack

| Component | Technology |
|---|---|
| Data Source | [OpenSky Network REST API](https://opensky-network.org/apidoc/) (free, no auth) |
| Mapping | [Leaflet.js](https://leafletjs.com/) v1.9.4 + CartoDB Dark Matter tiles |
| Charts | [Chart.js](https://www.chartjs.org/) |
| Frontend | Vanilla HTML5, CSS3, JavaScript |
| Server | Python `http.server` (built-in, zero dependencies) |

**No npm, no Node.js, no build step, no dependencies to install.** Just Python for the local server.

---

## API Reference

### Endpoint
```
GET https://opensky-network.org/api/states/all
```

### Bounded Area (Turkish Airspace)
```
GET https://opensky-network.org/api/states/all?lamin=35.8&lomin=25.5&lamax=42.2&lomax=45.0
```

### Rate Limits
- Anonymous: 10-second minimum interval between requests
- Registered (free): 5-second interval — [register here](https://opensky-network.org/index.php?option=com_users&view=registration)

### Response Fields

| Index | Field | Type | Description |
|---|---|---|---|
| 0 | icao24 | string | ICAO 24-bit transponder address |
| 1 | callsign | string | Callsign (8 chars max) |
| 2 | origin_country | string | Country of origin |
| 3 | time_position | int | Unix timestamp of last position |
| 4 | last_contact | int | Unix timestamp of last contact |
| 5 | longitude | float | WGS-84 longitude (degrees) |
| 6 | latitude | float | WGS-84 latitude (degrees) |
| 7 | baro_altitude | float | Barometric altitude (meters) |
| 8 | on_ground | bool | Is aircraft on ground |
| 9 | velocity | float | Ground speed (m/s) |
| 10 | true_track | float | Track angle (degrees, clockwise from N) |
| 11 | vertical_rate | float | Vertical rate (m/s) |
| 12 | sensors | int[] | Receiver IDs |
| 13 | geo_altitude | float | Geometric altitude (meters) |
| 14 | squawk | string | Transponder squawk code |
| 15 | spi | bool | Special purpose indicator |
| 16 | position_source | int | Position data origin (0=ADS-B, 1=ASTERIX, 2=MLAT) |

Full API docs: https://opensky-network.org/apidoc/rest.html

---

## Project Structure

```
aviation-safety-analytics/
├── index.html          # Complete application (HTML + CSS + JS)
├── README.md           # This file
├── .gitignore          # Git ignore rules
└── screenshots/        # Dashboard screenshots (optional)
```

---

## Troubleshooting

| Problem | Solution |
|---|---|
| "SIMULATION MODE" showing | Start a local server (`python -m http.server 8081`) and open `http://localhost:8081` |
| `python` command not found | Install Python 3 from [python.org](https://www.python.org/downloads/) or try `python3` / `py` |
| Port already in use | Use a different port: `python -m http.server 9000` |
| No aircraft appearing | OpenSky may be temporarily down or rate-limited. Wait 30s and refresh. |
| Map tiles not loading | Check internet connection. CartoDB tiles require online access. |

---

## Future Improvements

- [ ] OpenSky registered account for faster refresh (5s vs 10s)
- [ ] Historical flight data recording and playback
- [ ] Machine learning model trained on NTSB accident data
- [ ] Email/SMS alert notifications
- [ ] Multi-region support (Europe, Asia, Americas)
- [ ] Airport proximity detection for low-altitude filtering

---

## Data Sources & Credits

- **Flight Data**: [OpenSky Network](https://opensky-network.org/) — free and open ADS-B data
- **Map Tiles**: [CartoDB](https://carto.com/) Dark Matter basemap
- **Accident Data Reference**: [NTSB Aviation Database](https://www.ntsb.gov/Pages/AviationQueryV2.aspx), [NASA ASRS](https://asrs.arc.nasa.gov/)

---

## Contributors

**Zeynep Su Sosun** — Software Engineering Student, Near East University

- GitHub: [github.com/zeynepsusosun](https://github.com/zeynepsusosun)
- LinkedIn: [linkedin.com/in/zeynep-su-sosun-65ab28296](https://www.linkedin.com/in/zeynep-su-sosun-65ab28296)

**Enes Erul** - Software Engineering Student, Near East University

- GitHub: [github.com/0Ens] (https://github.com/0Ens)
- LinkedIn: [linkedin.com/in/enes-erul-082b4028a] (https://www.linkedin.com/in/enes-erul-082b4028a)

---

## License

This project is for educational and portfolio purposes. Flight data provided by OpenSky Network under their [terms of use](https://opensky-network.org/about/terms-of-use).
