# 🪷 Tamil Nadu Digital Twin

A district-level intelligence dashboard for all 38 districts of Tamil Nadu — real census data, real live weather and air quality, and a Claude-powered AI analysis tab. No simulated or random numbers anywhere.

---

## ✨ What's in the dashboard

- **Overview** — real population, area, density, and Census 2011 literacy rate per district, compared against the Tamil Nadu average
- **Live Weather & Air Quality** — real-time data from OpenWeatherMap and WAQI (CPCB ground stations) for the selected district's headquarters
- **Water & Energy Resources** — direct links to the actual official dashboards (TWAD Board, India-WRIS, TANGEDCO, IMD), since no public live API exists for these at district level
- **AI Insights** — ask Claude questions about a district; answers are grounded only in the real data shown in the dashboard

No backend required — it's two files (`index.html` + `district.json`) that run entirely in the browser.

---

## 🚀 Quick Start

### Run locally
```bash
python -m http.server 8000
# visit http://localhost:8000
```
Keep `index.html` and `district.json` in the same folder.

### Deploy on GitHub Pages
1. Push `index.html` and `district.json` to a repo
2. Settings → Pages → Deploy from branch `main`, folder `/ (root)`
3. Live at `https://yourusername.github.io/your-repo-name`

See `DEPLOYMENT.md` for Vercel, Netlify, AWS, and Docker instructions.

---

## 🔑 API Keys

| Service | Used for | Where it lives |
|---|---|---|
| OpenWeatherMap | Live weather | Already embedded in `index.html` |
| WAQI | Live air quality | Already embedded in `index.html` |
| Anthropic (Claude) | AI Insights tab | Each visitor pastes their own key in the app — never saved to a file |

If you need to rotate the weather/AQI keys later, they're two constants near the top of the `<script>` block in `index.html`: `OPENWEATHER_KEY` and `WAQI_TOKEN`.

Full technical details on every integration: see `API_INTEGRATION.md`.

---

## 📊 Data Sources

- **District data**: Government of Tamil Nadu, current district-wise figures
- **Literacy**: Census 2011 (India's most recent literacy census). The six districts formed after 2011 — Chengalpattu, Kallakurichi, Ranipet, Tenkasi, Tirupathur, Mayiladuthurai — show their undivided parent district's 2011 figure, clearly labeled as such
- **Weather**: OpenWeatherMap
- **Air quality**: WAQI, aggregating CPCB monitoring stations
- **AI analysis**: Anthropic Claude API

## 📁 File Structure

```
tamil-nadu-digital-twin/
├── index.html              # The full application
├── district.json           # All 38 districts with real data
├── README.md                # This file
├── DEPLOYMENT.md            # Hosting instructions (GitHub Pages, Vercel, Netlify, AWS, Docker)
└── API_INTEGRATION.md       # Exactly how each data source is wired up
```

## 🛠️ Updating district.json

Each district entry looks like:
```json
{ "name": "Coimbatore", "code": "COI", "headquarters": "Coimbatore",
  "established": "1956-11-01", "area_km2": 4950.7, "population": 3458045,
  "literacy_pct": 83.98, "literacy_is_parent_estimate": false,
  "lat": 11.02, "lon": 76.96 }
```
Edit values directly — e.g. once new census figures are published — no code changes needed elsewhere.

## 🐛 Troubleshooting

- **"Could not load district.json"** → it must sit in the same folder as `index.html`, and you must be serving over `http://` (not opening the file directly with `file://`), since browsers block local JSON fetches otherwise.
- **Weather/AQI shows "unavailable"** → the free-tier key may be rate-limited, or (for AQI) there's no monitoring station near that district's headquarters. The app tells you which, and links to the official source.
- **AI tab says to paste a key** → enter an Anthropic API key (starts `sk-ant-`) in the AI Insights tab and click "Use this key".

---

**Last updated**: July 2026
