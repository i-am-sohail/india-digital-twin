# 🔗 API Integration Guide

Complete guide to integrating real government APIs and third-party data sources into the India Digital Twin Dashboard.

---

## Overview

The dashboard can fetch data from multiple sources:

| Data Type | Source | API | Status |
|-----------|--------|-----|--------|
| Population | Census of India | census2021.gov.in | ✅ Available |
| Weather | IMD / OpenWeatherMap | openweathermap.org | ✅ Free tier |
| Air Quality | CPCB / WAQI | waqi.info | ✅ Free API |
| Energy | Ministry of Power | posoco.in | ⚠️ Manual data |
| Water | Central Water Commission | cwc.gov.in | ⚠️ Rate-limited |
| Rainfall | IMD | imd.gov.in | ✅ Available |

---

## 1. Census Data Integration

### Census 2021 Official Data

**Source**: https://census2021.gov.in

**Available Data**:
- Total population
- Male/Female ratio
- Literacy rate
- Urban/Rural split
- Area
- Density

**Integration Code**:

```javascript
async function fetchCensusData(district) {
  try {
    // Census 2021 provides district-level data
    // Data is published on census2021.gov.in
    // You can manually compile or use available APIs
    
    const censusMap = {
      'Tiruppur': {
        population: 1831000,
        area: 5186,
        literacy: 76.5,
        density: 353,
        urbanization: 45.2,
        sexRatio: 996
      },
      'Chennai': {
        population: 7088000,
        area: 426,
        literacy: 82.3,
        density: 16635,
        urbanization: 100,
        sexRatio: 993
      },
      // Add more districts...
    };
    
    return censusMap[district] || generateDefaultData();
  } catch (error) {
    console.error('Census data fetch failed:', error);
    return null;
  }
}

// Usage in fetchDistrictData():
// const censusData = await fetchCensusData(currentDistrict);
// districtData.population = censusData.population;
// districtData.literacy = censusData.literacy;
```

**Manual Integration**:
1. Visit: https://census2021.gov.in
2. Download district-level data (Excel/CSV)
3. Convert to JSON: `districts-census.json`
4. Load in JavaScript:

```javascript
const censusDatabase = {
  // populated from districts-census.json
};
```

---

## 2. Weather Data Integration

### OpenWeatherMap API (Free Tier)

**Signup**: https://openweathermap.org/api

**Free Tier Includes**:
- Current weather
- 5-day forecast
- Temperature, humidity, wind
- Sunrise/sunset times

**Integration Code**:

```javascript
const OPENWEATHER_API_KEY = 'YOUR_API_KEY_HERE';

async function fetchWeatherData(districtCity) {
  try {
    const response = await fetch(
      `https://api.openweathermap.org/data/2.5/weather` +
      `?q=${districtCity}&appid=${OPENWEATHER_API_KEY}&units=metric`
    );
    
    if (!response.ok) throw new Error('Weather API failed');
    
    const data = await response.json();
    
    return {
      temperature: data.main.temp,
      humidity: data.main.humidity,
      pressure: data.main.pressure,
      condition: data.weather[0].main,
      windSpeed: data.wind.speed,
      cloudCover: data.clouds.all,
      sunrise: new Date(data.sys.sunrise * 1000),
      sunset: new Date(data.sys.sunset * 1000)
    };
  } catch (error) {
    console.error('Weather fetch failed:', error);
    return null;
  }
}

// For 5-day forecast:
async function fetchWeatherForecast(districtCity) {
  const response = await fetch(
    `https://api.openweathermap.org/data/2.5/forecast` +
    `?q=${districtCity}&appid=${OPENWEATHER_API_KEY}&units=metric`
  );
  const data = await response.json();
  
  // Extract daily forecasts
  return data.list.map(forecast => ({
    date: new Date(forecast.dt * 1000),
    temp: forecast.main.temp,
    humidity: forecast.main.humidity,
    rain: forecast.rain?.['3h'] || 0,
    description: forecast.weather[0].description
  }));
}
```

**Configuration (Secure)**:

Create `config.example.js`:
```javascript
// Copy this to config.js and add your API keys
// DO NOT commit config.js to git

window.API_CONFIG = {
  OPENWEATHER_API_KEY: 'sk_openweather_...',
  WAQI_API_KEY: 'sk_waqi_...',
  ANTHROPIC_API_KEY: 'sk_anthropic_...',
};
```

In `index.html`:
```html
<script src="config.js"></script>
<script>
  const weatherApiKey = window.API_CONFIG?.OPENWEATHER_API_KEY;
</script>
```

**Usage in Dashboard**:

```javascript
async function fetchDistrictData(state, district) {
  // ... existing code ...
  
  // Add weather data
  const weatherData = await fetchWeatherData(district);
  if (weatherData) {
    districtData.temperature = weatherData.temperature;
    districtData.humidity = weatherData.humidity;
  }
  
  return districtData;
}
```

---

## 3. Air Quality Integration

### WAQI (World Air Quality Index) API

**Signup**: https://waqi.info/

**Free Tier**: 30 requests/minute

**Available Metrics**:
- AQI (0-500 scale)
- PM2.5, PM10
- NO₂, SO₂, O₃, CO
- Temperature, humidity
- Real-time data from CPCB stations

**Integration Code**:

```javascript
const WAQI_API_KEY = 'YOUR_WAQI_API_KEY';

async function fetchAQIData(districtCity) {
  try {
    // Search for city
    const searchResponse = await fetch(
      `https://api.waqi.info/feed/${districtCity}/?token=${WAQI_API_KEY}`
    );
    
    const data = await searchResponse.json();
    
    if (data.status !== 'ok') {
      throw new Error('City not found in WAQI database');
    }
    
    const aqiData = data.data;
    
    return {
      aqi: aqiData.aqi,
      pm25: aqiData.iaqi?.pm25?.v || null,
      pm10: aqiData.iaqi?.pm10?.v || null,
      no2: aqiData.iaqi?.no2?.v || null,
      so2: aqiData.iaqi?.so2?.v || null,
      o3: aqiData.iaqi?.o3?.v || null,
      co: aqiData.iaqi?.co?.v || null,
      humidity: aqiData.iaqi?.h?.v || null,
      temperature: aqiData.iaqi?.t?.v || null,
      lastUpdate: new Date(aqiData.time.s),
      dominantPollutant: getAQICategory(aqiData.aqi),
      city: aqiData.city?.name,
      coordinates: [aqiData.city?.geo?.[0], aqiData.city?.geo?.[1]]
    };
  } catch (error) {
    console.error('AQI fetch failed:', error);
    return null;
  }
}

// AQI Category mapping
function getAQICategory(aqi) {
  if (aqi <= 50) return { category: 'Good', color: '#1baf7a' };
  if (aqi <= 100) return { category: 'Satisfactory', color: '#eda100' };
  if (aqi <= 200) return { category: 'Moderately Polluted', color: '#e34948' };
  if (aqi <= 300) return { category: 'Poor', color: '#a32d2d' };
  return { category: 'Very Poor', color: '#601313' };
}

// Fetch data for multiple zones
async function fetchZoneAQIData(zones) {
  // zones = ['North Zone', 'South Zone', 'Central Zone']
  
  const zoneMap = {
    'North Zone': 'Tiruppur North',
    'South Zone': 'Tiruppur South',
    'Central Zone': 'Tiruppur Central',
    'East Zone': 'Tiruppur East',
    'West Zone': 'Tiruppur West',
    'Industrial Zone': 'SIPCOT Industrial Area'
  };
  
  const aqiData = await Promise.all(
    zones.map(zone => fetchAQIData(zoneMap[zone]))
  );
  
  return aqiData.map((data, i) => ({
    zone: zones[i],
    aqi: data?.aqi || 75,
    pm25: data?.pm25 || 35,
    category: data?.dominantPollutant?.category || 'Moderate'
  }));
}
```

**CPCB Direct Integration**:

```javascript
// CPCB (Central Pollution Control Board)
// Direct API: https://api.data.gov.in

async function fetchCPCBData(station, pollutant) {
  // Requires API key from data.gov.in
  
  const apiKey = 'YOUR_DATA_GOV_API_KEY';
  
  const response = await fetch(
    `https://api.data.gov.in/resource/` +
    `3b01bcb8-0b14-41a6-bf81-c80b194c5086` +
    `?api-key=${apiKey}` +
    `&format=json` +
    `&filters[Station]=${station}` +
    `&filters[Pollutant]=${pollutant}`
  );
  
  const data = await response.json();
  return data.records;
}
```

---

## 4. Energy Data Integration

### Ministry of Power APIs

**Sources**:
- POSOCO (Power System Operation Corporation)
- NIXI Open Data
- Ministry of Power Dashboard

**Integration Code**:

```javascript
async function fetchEnergyData(state) {
  try {
    // POSOCO real-time data
    // Note: Direct API may not be publicly available
    // Alternative: Scrape from dashboard or use data.gov.in
    
    // Using cached data from Energy Ministry
    const energyMap = {
      'Tamil Nadu': {
        demand: 13500,  // MW
        generation: 14200,
        solar: 2500,
        wind: 8500,
        thermal: 3000,
        hydro: 200,
        capacity: 14500
      },
      'Andhra Pradesh': {
        demand: 13200,
        generation: 14100,
        solar: 2800,
        wind: 4500,
        thermal: 6000,
        hydro: 800,
        capacity: 15000
      },
      // Add more states...
    };
    
    return energyMap[state] || generateDefaultEnergyData();
  } catch (error) {
    console.error('Energy data fetch failed:', error);
    return null;
  }
}

// 24-hour demand profile
async function fetch24HourDemandProfile(state) {
  // Pattern: Low at night, peaks in afternoon
  const hours = Array.from({length: 24}, (_, i) => i);
  const baseDemand = 10000; // MW
  
  return hours.map(hour => {
    const factor = 0.7 + 0.3 * Math.sin((hour - 6) * Math.PI / 24);
    return Math.round(baseDemand * factor);
  });
}

// Renewable energy tracking
async function fetchRenewableStatus(state) {
  return {
    solar: {
      capacity: 2500,  // MW
      generation: 1200, // Current MW (depends on time of day)
      plants: 45,
      trend: 'up'
    },
    wind: {
      capacity: 8500,
      generation: 4200,
      plants: 180,
      trend: 'stable'
    },
    hydro: {
      capacity: 800,
      generation: 150,
      plants: 12,
      trend: 'down'  // Dry season
    },
    total_renewable_percentage: 42.5
  };
}
```

---

## 5. Water Resources Integration

### Central Water Commission APIs

**Sources**:
- CWC Dam Level Data
- State Water Boards
- NDMA Flood Data

**Integration Code**:

```javascript
async function fetchWaterData(district) {
  try {
    // CWC publishes dam levels (varies by state)
    // Create manual database from CWC website
    
    const waterDatabase = {
      'Tiruppur': {
        dams: [
          {
            name: 'Amaravathi Dam',
            level: 68,
            capacity: 100,
            inflow: 45,  // m³/s
            outflow: 38
          },
          {
            name: 'Thirumurthy Dam',
            level: 52,
            capacity: 100,
            inflow: 32,
            outflow: 28
          }
        ],
        schemes: [
          {
            name: 'Bhavani Scheme',
            status: 'Active',
            capacity: 80,  // MLD
            distribution: [
              { area: 'Tiruppur North', percentage: 40 },
              { area: 'Tiruppur South', percentage: 35 },
              { area: 'Rural Areas', percentage: 25 }
            ]
          }
        ],
        rainfall: {
          ytd: 312,  // mm
          annual_average: 700,
          next_rainfall: 'NE Monsoon in 14 days'
        },
        quality: {
          ph: 7.2,
          turbidity: 2.5,
          hardness: 85,
          tds: 450,
          chloride: 45
        }
      },
      // Add more districts...
    };
    
    return waterDatabase[district] || null;
  } catch (error) {
    console.error('Water data fetch failed:', error);
    return null;
  }
}

// Fetch dam levels from CWC
async function fetchDamLevels(state) {
  try {
    // CWC provides: https://cwc.gov.in
    // Data is typically updated daily
    
    const response = await fetch(
      `https://cwc.gov.in/api/dam-levels?state=${state}`
    );
    
    // Note: This endpoint may not be directly available
    // Alternative: Manually scrape from CWC website
    
    const data = await response.json();
    return data.dams;
  } catch (error) {
    console.log('CWC API not available, using cached data');
    return null;
  }
}

// Flood risk assessment
function assessFloodRisk(damLevel, rainfall, riverFlow) {
  const baseRisk = (damLevel / 100) * 0.4;
  const rainfallRisk = Math.min(rainfall / 500, 1) * 0.3;
  const flowRisk = Math.min(riverFlow / 1000, 1) * 0.3;
  
  const totalRisk = baseRisk + rainfallRisk + flowRisk;
  
  return {
    riskScore: Math.round(totalRisk * 100),
    category: totalRisk < 0.33 ? 'Low' : totalRisk < 0.66 ? 'Moderate' : 'High',
    affectedZones: totalRisk > 0.5 ? ['Zone D3', 'Zone D4', 'Zone D5'] : []
  };
}
```

---

## 6. Government Census Database (CSV to JSON)

### Converting Census Data

**Original CSV** (from census2021.gov.in):

```csv
District,State,Population,Area,Literacy,Density
Tiruppur,Tamil Nadu,1831000,5186,76.5,353
Chennai,Tamil Nadu,7088000,426,82.3,16635
...
```

**Python Script to Convert**:

```python
import csv
import json

# Read CSV
data = {}
with open('census-districts.csv', 'r') as f:
    reader = csv.DictReader(f)
    for row in reader:
        state = row['State']
        if state not in data:
            data[state] = {}
        
        data[state][row['District']] = {
            'population': int(row['Population']),
            'area': float(row['Area']),
            'literacy': float(row['Literacy']),
            'density': float(row['Density'])
        }

# Write JSON
with open('census-data.json', 'w') as f:
    json.dump(data, f, indent=2)

print('Converted successfully!')
```

**Usage in JavaScript**:

```javascript
let censusDatabase = {};

async function loadCensusDatabase() {
  const response = await fetch('census-data.json');
  censusDatabase = await response.json();
}

function getCensusData(state, district) {
  return censusDatabase[state]?.[district] || null;
}
```

---

## 7. Real-Time Data Updates

### Auto-refresh Mechanism

```javascript
let refreshInterval = null;
const REFRESH_INTERVALS = {
  weather: 30 * 60 * 1000,    // 30 minutes
  aqi: 60 * 60 * 1000,         // 1 hour
  energy: 15 * 60 * 1000,      // 15 minutes
  water: 24 * 60 * 60 * 1000   // 24 hours
};

function startAutoRefresh() {
  // Refresh weather every 30 minutes
  setInterval(async () => {
    if (currentDistrict) {
      const weather = await fetchWeatherData(currentDistrict);
      updateWeatherUI(weather);
    }
  }, REFRESH_INTERVALS.weather);
  
  // Refresh AQI every hour
  setInterval(async () => {
    if (currentDistrict) {
      const aqi = await fetchAQIData(currentDistrict);
      updateAQIUI(aqi);
    }
  }, REFRESH_INTERVALS.aqi);
}

function updateWeatherUI(weather) {
  // Update temperature, humidity displays
  document.getElementById('temp-display').textContent = 
    Math.round(weather.temperature) + '°C';
}

function updateAQIUI(aqi) {
  // Update AQI card
  document.getElementById('aqi-value').textContent = aqi.aqi;
  document.getElementById('aqi-category').textContent = 
    getAQICategory(aqi.aqi).category;
}
```

---

## 8. Caching Strategy

### LocalStorage Caching

```javascript
const CACHE_DURATION = {
  census: 7 * 24 * 60 * 60 * 1000,  // 7 days
  weather: 30 * 60 * 1000,           // 30 minutes
  aqi: 60 * 60 * 1000,               // 1 hour
};

function cacheData(key, data, duration) {
  const cache = {
    data: data,
    timestamp: Date.now()
  };
  localStorage.setItem(key, JSON.stringify(cache));
  localStorage.setItem(key + '_expiry', Date.now() + duration);
}

function getCachedData(key) {
  const expiry = localStorage.getItem(key + '_expiry');
  
  if (!expiry || Date.now() > parseInt(expiry)) {
    localStorage.removeItem(key);
    return null;
  }
  
  const cache = localStorage.getItem(key);
  return cache ? JSON.parse(cache).data : null;
}

// Usage
async function fetchWithCache(key, fetchFn, duration) {
  const cached = getCachedData(key);
  if (cached) return cached;
  
  const data = await fetchFn();
  cacheData(key, data, duration);
  return data;
}

// In fetchDistrictData:
const censusData = await fetchWithCache(
  'census_' + district,
  () => fetchCensusData(district),
  CACHE_DURATION.census
);
```

---

## 9. Error Handling

### Graceful Degradation

```javascript
async function fetchDistrictDataWithFallback(state, district) {
  let data = generateDefaultData(state, district);
  
  try {
    const censusData = await fetchCensusData(district);
    if (censusData) Object.assign(data, censusData);
  } catch (error) {
    console.warn('Census data unavailable:', error);
    // Continue with default data
  }
  
  try {
    const weatherData = await fetchWeatherData(district);
    if (weatherData) Object.assign(data, weatherData);
  } catch (error) {
    console.warn('Weather data unavailable:', error);
  }
  
  try {
    const aqiData = await fetchAQIData(district);
    if (aqiData) data.aqi = aqiData.aqi;
  } catch (error) {
    console.warn('AQI data unavailable:', error);
  }
  
  return data;
}
```

---

## 10. Best Practices

### API Key Management

```javascript
// ❌ DON'T
const API_KEY = 'sk-12345';  // Exposed!

// ✅ DO
const API_KEY = process.env.REACT_APP_API_KEY;

// ✅ BETTER: Backend proxy
fetch('/api/weather', {
  method: 'POST',
  body: JSON.stringify({ district: 'Tiruppur' })
});
```

### Rate Limiting

```javascript
class RateLimiter {
  constructor(maxRequests, windowMs) {
    this.maxRequests = maxRequests;
    this.windowMs = windowMs;
    this.requests = [];
  }
  
  canRequest() {
    const now = Date.now();
    this.requests = this.requests.filter(t => now - t < this.windowMs);
    
    if (this.requests.length < this.maxRequests) {
      this.requests.push(now);
      return true;
    }
    return false;
  }
}

const limiter = new RateLimiter(30, 60000);  // 30 requests/minute

async function fetchWithRateLimit(url) {
  if (!limiter.canRequest()) {
    throw new Error('Rate limit exceeded');
  }
  return fetch(url);
}
```

### Monitoring API Usage

```javascript
const apiStats = {
  calls: 0,
  errors: 0,
  averageTime: 0,
  lastError: null
};

async function fetchWithMetrics(url) {
  const start = Date.now();
  apiStats.calls++;
  
  try {
    const response = await fetch(url);
    const duration = Date.now() - start;
    apiStats.averageTime = (apiStats.averageTime + duration) / 2;
    return response;
  } catch (error) {
    apiStats.errors++;
    apiStats.lastError = error.message;
    throw error;
  }
}

// Monitor in console
setInterval(() => {
  console.log('API Stats:', apiStats);
}, 60000);
```

---

## Quick Reference

| API | Endpoint | Free Tier | Rate Limit | Response Time |
|-----|----------|-----------|-----------|----------------|
| OpenWeatherMap | api.openweathermap.org | ✅ Yes | 60/min | 100-300ms |
| WAQI | api.waqi.info | ✅ Yes | 30/min | 200-400ms |
| data.gov.in | api.data.gov.in | ✅ Yes | 100/hr | 500-1000ms |
| Census | census2021.gov.in | ❌ Manual | - | - |
| CWC | cwc.gov.in | ❌ Manual | - | - |

---

## Troubleshooting API Integration

**Issue**: CORS Error
```javascript
// Use CORS proxy
const response = await fetch(
  'https://cors-anywhere.herokuapp.com/' + originalUrl
);
```

**Issue**: API Timeout
```javascript
// Add timeout wrapper
function fetchWithTimeout(url, timeout = 5000) {
  return Promise.race([
    fetch(url),
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error('Timeout')), timeout)
    )
  ]);
}
```

**Issue**: Rate Limit Exceeded
```javascript
// Implement exponential backoff
async function fetchWithRetry(url, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await fetch(url);
    } catch (error) {
      if (i === retries - 1) throw error;
      await new Promise(r => setTimeout(r, Math.pow(2, i) * 1000));
    }
  }
}
```

---

**Last Updated**: 2024 | v1.0  
**Maintainer**: India Digital Twin Project
