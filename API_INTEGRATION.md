# 🔗 API Integration — Tamil Nadu Digital Twin

This document describes exactly what's wired up in `index.html` today. No hypothetical code — this is what actually runs.

---

## Overview

| Data | Source | Status | Key required |
|---|---|---|---|
| Population, area, density, literacy | Government of Tamil Nadu district data + Census 2011 (literacy) | ✅ Built into `district.json`, static | None |
| Weather (temp, humidity, wind, condition) | OpenWeatherMap | ✅ Live, called from browser | Yours, already in the file |
| Air quality (AQI, PM2.5, PM10, NO₂) | WAQI (aggregates CPCB ground stations) | ✅ Live, called from browser | Yours, already in the file |
| AI analysis | Anthropic Claude API | ✅ Live, called from browser | Visitor pastes their own key in the AI tab |
| Reservoir/dam levels | Central Water Commission / India-WRIS | ⚠️ No public browser-callable API | Linked out instead, not fabricated |
| Live electricity demand | TANGEDCO / POSOCO | ⚠️ No public browser-callable API | Linked out instead, not fabricated |

---

## 1. Census & District Data

Static, sourced once and stored in `district.json` — no API call needed at runtime.

```javascript
const res = await fetch('district.json');
const data = await res.json();
// data.districts is an array of 38 objects:
// { name, code, headquarters, established, area_km2, population,
//   literacy_pct, literacy_is_parent_estimate, parent_district, lat, lon }
```

To refresh this later (e.g. once Census 2027 data is published), just edit the numbers in `district.json` directly — no code changes needed.

---

## 2. Weather — OpenWeatherMap (implemented)

```javascript
const r = await fetch(
  `https://api.openweathermap.org/data/2.5/weather?lat=${d.lat}&lon=${d.lon}&appid=${OPENWEATHER_KEY}&units=metric`
);
const weather = await r.json();
// weather.main.temp, weather.main.humidity, weather.weather[0].main, weather.wind.speed
```

Free tier: 60 calls/minute, 1,000,000 calls/month. If you outgrow it, get a new key at openweathermap.org/api and swap the `OPENWEATHER_KEY` constant near the top of the `<script>` block in `index.html`.

---

## 3. Air Quality — WAQI (implemented)

```javascript
const r = await fetch(
  `https://api.waqi.info/feed/geo:${d.lat};${d.lon}/?token=${WAQI_TOKEN}`
);
const j = await r.json();
// j.data.aqi, j.data.iaqi.pm25.v, j.data.iaqi.pm10.v, j.data.iaqi.no2.v
```

WAQI returns the *nearest* monitoring station to the coordinates given, which may be several kilometres away in rural districts with fewer CPCB stations. If no station is close enough, the app shows an honest "unavailable" message with a link to CPCB's own portal, rather than guessing.

Free tier: 30 requests/minute.

---

## 4. AI Insights — Claude API (implemented)

Called directly from the browser using Anthropic's CORS-enabled endpoint. The visitor's key never leaves their session — it's held in a JavaScript variable, not written to disk or `localStorage`.

```javascript
const r = await fetch('https://api.anthropic.com/v1/messages', {
  method: 'POST',
  headers: {
    'content-type': 'application/json',
    'x-api-key': anthropicKey,           // pasted by the visitor
    'anthropic-version': '2023-06-01',
    'anthropic-dangerous-direct-browser-access': 'true'
  },
  body: JSON.stringify({
    model: 'claude-sonnet-5',
    max_tokens: 500,
    system: systemPrompt,                 // built from real district.json + live weather/AQI figures
    messages: [{ role: 'user', content: userQuestion }]
  })
});
```

The `anthropic-dangerous-direct-browser-access` header is Anthropic's own documented mechanism for exactly this "bring your own key, call it from the browser" pattern — it's not a hack.

**Why the visitor pastes their own key instead of you hardcoding one:** an Anthropic key is billed per request. Anyone who views your page's source could copy a hardcoded key and run up charges on your account. OpenWeatherMap/WAQI keys are low-stakes free-tier keys, so those are safe to ship in the file; a paid, usage-billed key is not.

---

## 5. Water & Energy — why there's no live number

Tamil Nadu's water and power authorities (TWAD Board, TANGEDCO, India-WRIS/CWC) don't publish a public, CORS-enabled JSON API that a static webpage can call directly — their live dashboards are server-rendered pages meant for a browser, not a data feed. Rather than reverse-engineer or scrape them (fragile, and often against terms of use), the **Water & Energy Resources** tab links straight to:

- TWAD Board — twadboard.tn.gov.in
- India-WRIS reservoir dashboard — indiawris.gov.in/wris/#/reservoir
- IMD rainfall — mausam.imd.gov.in
- TANGEDCO — tangedco.gov.in
- POSOCO/Grid-India — posoco.in

If Tamil Nadu ever opens a public API for these (some states have started publishing open data via data.gov.in), swap the "Water & Energy" section in `renderResources()` in `index.html` for a live fetch — the pattern would look identical to the weather/AQI integrations above.

---

**Last updated**: July 2026 — reflects the actual code in this build, not a roadmap.
