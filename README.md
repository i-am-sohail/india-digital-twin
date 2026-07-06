# 🇮🇳 India Digital Twin Dashboard

A comprehensive, real-time urban intelligence platform for monitoring and analyzing any district in India. Built with modern web technologies and AI-powered decision-making capabilities.

**[Live Demo](#)** · **[Documentation](#)** · **[GitHub](#)**

---

## ✨ Features

### 🎯 Core Capabilities
- **Dynamic Location Selection** - Select any State/UT and District in India
- **Real-Time Dashboards** - 6 comprehensive tabs covering all urban aspects
- **AI Analysis Engine** - Claude-powered decision making with natural language queries
- **Interactive Charts** - Chart.js powered visualizations with live data
- **Responsive Design** - Works on desktop, tablet, and mobile
- **Dark Mode Support** - Automatic light/dark theme detection
- **No Backend Required** - Fully static deployment, runs on GitHub Pages

### 📊 Dashboard Tabs

1. **Overview** - Population, area, density, literacy, GDP trends
2. **Water** - Supply levels, rainfall, per capita consumption, distribution schemes
3. **Energy** - Demand profiles, capacity, renewable mix, 24-hour trends
4. **Environment** - Air quality, temperature, humidity, green cover
5. **Infrastructure** - Hospitals, schools, roads, connectivity status
6. **AI Analysis** - Natural language queries powered by Claude API

---

## 🚀 Quick Start

### Option 1: Deploy on GitHub Pages

1. **Fork the repository** or clone it:
   ```bash
   git clone https://github.com/yourusername/india-digital-twin.git
   cd india-digital-twin
   ```

2. **Create `index.html` and `districts.json`** in the repository root (files provided)

3. **Push to GitHub**:
   ```bash
   git add .
   git commit -m "Initial commit: India Digital Twin Dashboard"
   git push origin main
   ```

4. **Enable GitHub Pages**:
   - Go to Settings → Pages
   - Set Source to: `Deploy from a branch`
   - Select: `main` branch, `/ (root)` folder
   - Click Save

5. **Access your dashboard** at: `https://yourusername.github.io/india-digital-twin`

### Option 2: Local Development

1. **Clone the repository**:
   ```bash
   git clone <repo-url>
   cd india-digital-twin
   ```

2. **Serve locally** (Python 3):
   ```bash
   python -m http.server 8000
   ```

3. **Open browser** to: `http://localhost:8000`

---

## 🔑 API Configuration

### AI Features (Claude API)

To enable AI decision making, configure your Anthropic API key:

1. **Get an API key**:
   - Visit [api.anthropic.com](https://api.anthropic.com)
   - Create a new API key

2. **Options for using the API**:

   **Option A: Client-side (Browser)**
   - Add a configuration file `config.js`:
   ```javascript
   window.ANTHROPIC_API_KEY = 'your-api-key-here';
   ```
   - **Note**: Not recommended for production (exposes API key)

   **Option B: Backend Proxy (Recommended)**
   - Set up a simple Node.js/Python backend
   - Forward requests through your server
   - Never expose API key to client

   **Option C: User-provided Keys**
   - Users enter their own API key in the app
   - Recommended for open-source deployments

### Example: Adding Backend Proxy (Node.js)

Create `api-proxy.js`:
```javascript
const express = require('express');
const axios = require('axios');
const cors = require('cors');

const app = express();
app.use(cors());
app.use(express.json());

const ANTHROPIC_API_KEY = process.env.ANTHROPIC_API_KEY;

app.post('/api/claude', async (req, res) => {
  try {
    const response = await axios.post('https://api.anthropic.com/v1/messages', 
      {
        model: 'claude-sonnet-4-6',
        max_tokens: 500,
        system: req.body.system,
        messages: req.body.messages
      },
      {
        headers: {
          'x-api-key': ANTHROPIC_API_KEY,
          'content-type': 'application/json'
        }
      }
    );
    res.json(response.data);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(3000, () => console.log('Proxy running on :3000'));
```

Deploy on Heroku or Vercel.

---

## 📊 Data Integration Guide

### Current Data Sources

The dashboard currently uses:
- **Generated Data** - Realistic randomized values based on district characteristics
- **Embedded Districts List** - All Indian states and districts
- **Wikipedia API** - Optional Wikipedia data (commented, can be uncommented)

### Integrating Real Government APIs

#### 1. Census Data (Census of India)

```javascript
// Replace in fetchDistrictData()
const censusData = await fetch(
  `https://api.census2021.gov.in/data?district=${district}`
);
// Map response to: population, area, density, literacy

const censusMapping = {
  'population': 'TOT_P',
  'literacy': 'LIT_RATE',
  'area': 'AREA_KM2',
  'density': 'DENSITY_PER_KM2'
};
```

**API Details**:
- Endpoint: Census India official API (when available)
- Data: 2021 Census figures for all districts
- Note: As of 2024, Census directly publishes on census2021.gov.in

#### 2. Weather Data (IMD or OpenWeatherMap)

```javascript
// India Meteorological Department
const weatherData = await fetch(
  `https://api.openweathermap.org/data/2.5/weather?q=${districtCity}&appid=YOUR_API_KEY`
);

// Maps to: temperature, humidity, rainfall
const weatherMapping = {
  'temperature': 'main.temp',
  'humidity': 'main.humidity',
  'description': 'weather[0].main'
};
```

#### 3. Air Quality (CPCB - Central Pollution Control Board)

```javascript
// CPCB Real-time AQI Data
const aqiData = await fetch(
  `https://api.waqi.info/feed/${city}/?token=YOUR_WAQI_TOKEN`
);

// Maps to: aqi, pm25, pm10, no2
const aqiMapping = {
  'aqi': 'data.aqi',
  'pm25': 'data.iaqi.pm25.v',
  'pm10': 'data.iaqi.pm10.v'
};
```

#### 4. Energy Data (Ministry of Power)

```javascript
// Ministry of Power Dashboard Integration
const energyData = {
  'demand': 'from POSOCO API',
  'generation': 'from state generation sources',
  'solar': 'from renewable energy dashboard'
};

// Example: POSOCO (Power System Operation Corporation Limited)
const posoco = await fetch(
  `https://posoco.in/api/generation?state=${state}`
);
```

#### 5. Water Resources (Central Water Commission)

```javascript
// CWC Dam level data
const waterData = await fetch(
  `https://cwc.gov.in/api/dam-levels?district=${district}`
);

// Maps to: waterSupply, rainfall, damLevels
const waterMapping = {
  'damLevel': 'data[0].level_percent',
  'capacity': 'data[0].capacity_mcm',
  'rainfall': 'data[0].rainfall_mm'
};
```

---

## 🛠️ Customization Guide

### Adding New Metrics

1. **In `populateDashboard()`**:
```javascript
const customKpis = [
  { 
    label: '🔬 New Metric', 
    value: customData.value, 
    unit: 'unit' 
  }
];

document.getElementById('custom-kpis').innerHTML = customKpis.map(...).join('');
```

2. **In `initializeCharts()`**:
```javascript
const customCtx = document.getElementById('chart-custom');
new Chart(customCtx, {
  type: 'line',
  data: { labels: [...], datasets: [...] },
  options: baseOpts
});
```

### Changing Color Scheme

Edit CSS variables in `:root`:
```css
:root {
  --dt-blue: #2a78d6;      /* Primary color */
  --dt-teal: #1baf7a;      /* Success color */
  --dt-amber: #eda100;     /* Warning color */
  --dt-red: #e34948;       /* Critical color */
  --dt-violet: #4a3aa7;    /* AI/Special color */
}
```

### Extending AI Analysis

Modify `askAI()` function to include custom data:

```javascript
const systemPrompt = `You are the AI brain of the Digital Twin for ${currentDistrict}.

CURRENT STATUS:
- Population: ${districtData.population}
- Water Supply: ${districtData.waterSupply}%
- Energy Demand: ${districtData.energyDemand} MW
[... add more custom data ...]

Your role: Provide DIRECT, ACTIONABLE insights in 2-3 sentences.`;
```

---

## 📁 File Structure

```
india-digital-twin/
├── index.html              # Main application (single file)
├── districts.json          # State-district mapping
├── README.md               # This file
├── config.example.js       # API configuration template
├── DEPLOYMENT.md           # Detailed deployment guide
├── DATA_SOURCES.md         # Government API documentation
└── assets/
    ├── screenshots/        # Dashboard screenshots
    └── examples/           # Example configurations
```

---

## 🔐 Security Considerations

### API Key Safety

1. **Never commit API keys** to version control:
   ```bash
   echo "ANTHROPIC_API_KEY=sk-..." >> .env
   echo ".env" >> .gitignore
   ```

2. **Use environment variables** in production:
   ```javascript
   const apiKey = process.env.ANTHROPIC_API_KEY;
   ```

3. **Implement rate limiting** if using a backend proxy

4. **Validate all user inputs** before sending to APIs

### CORS Issues

If you encounter CORS issues:
1. Use a backend proxy (recommended)
2. Add CORS headers to your server:
   ```javascript
   app.use(cors({
     origin: 'https://yourdomain.com',
     credentials: true
   }));
   ```

---

## 📈 Performance Optimization

### Chart Rendering
- Charts are lazy-loaded when tabs are activated
- Prevents unnecessary rendering on page load
- Instances are destroyed and recreated per tab

### Data Fetching
```javascript
// Cache district data
let cachedData = {};

async function fetchDistrictData(state, district) {
  const key = `${state}_${district}`;
  if (cachedData[key]) return cachedData[key];
  
  const data = await fetchFromAPI(...);
  cachedData[key] = data;
  return data;
}
```

### Bundle Size
- Single HTML file (~100KB)
- External: Chart.js (from CDN)
- No heavy frameworks (pure JavaScript)
- ~40KB compressed

---

## 🧪 Testing

### Manual Testing Checklist

- [ ] State/District selector works
- [ ] All 6 tabs load correctly
- [ ] Charts render without errors
- [ ] Dark mode toggle works
- [ ] Responsive design on mobile (width < 600px)
- [ ] AI queries return results (if API configured)
- [ ] No console errors

### Automated Testing (Optional)

```javascript
// test.js
describe('Digital Twin Dashboard', () => {
  it('should load districts', () => {
    // Test loading districts.json
  });

  it('should initialize charts', () => {
    // Test chart initialization
  });

  it('should handle API errors gracefully', () => {
    // Test error handling
  });
});
```

---

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Make your changes
4. Test thoroughly
5. Submit a pull request

### Areas for Contribution

- [ ] Integrate real government APIs
- [ ] Add more visualization types
- [ ] Improve mobile responsiveness
- [ ] Add multilingual support
- [ ] Create district-specific themes
- [ ] Add historical data comparisons
- [ ] Implement data export (CSV/PDF)

---

## 📚 Government Data Sources

### Official APIs & Portals

1. **Census of India** - census2021.gov.in
2. **Ministry of Power** - power.gov.in
3. **IMD Weather** - imd.gov.in
4. **CPCB Air Quality** - cpcb.nic.in
5. **Central Water Commission** - cwc.gov.in
6. **Ministry of Statistics** - mospi.gov.in
7. **NITI Aayog** - niti.gov.in

### Data.gov.in Integration

```javascript
// Fetch from India's open data portal
const openData = await fetch(
  `https://api.data.gov.in/resource/...?api-key=YOUR_API_KEY`
);
```

---

## 🐛 Troubleshooting

### Issue: "districts.json not found"
**Solution**: Ensure `districts.json` is in the root directory alongside `index.html`

### Issue: AI features not working
**Solution**: 
- Check API key configuration
- Verify API key has correct permissions
- Check browser console for error messages
- Ensure internet connection is active

### Issue: Charts not rendering
**Solution**:
- Clear browser cache (Ctrl+Shift+Del)
- Check Chart.js is loaded from CDN
- Verify canvas elements have unique IDs

### Issue: Mobile layout broken
**Solution**:
- Check viewport meta tag is present
- Verify CSS media queries are working
- Test on actual mobile device (not just resizing)

---

## 📊 Sample Data

The dashboard includes realistic sample data generation:

```javascript
// Example: Tiruppur District
{
  population: ~1,830,000,
  area: ~5,186 km²,
  density: ~353 per km²,
  literacy: ~76.5%,
  temperature: 25-32°C,
  waterSupply: 60-85%,
  energyDemand: 500-1000 MW,
  aqi: 40-100 (varies by zone)
}
```

### Generate Custom Sample Data

```javascript
function generateDistrictData(state, district) {
  const basePopulation = 1000000 + Math.random() * 5000000;
  const baseArea = 1000 + Math.random() * 5000;
  
  return {
    population: Math.round(basePopulation),
    area: Math.round(baseArea),
    density: Math.round(basePopulation / baseArea),
    // ... other metrics
  };
}
```

---

## 📱 Mobile Responsiveness

Responsive breakpoints:
- **Desktop**: 1400px+ (3-column layouts)
- **Tablet**: 1000px - 1399px (2-column layouts)
- **Mobile**: < 1000px (1-column layouts)

---

## 🎓 Educational Use

This project is suitable for:
- **Geography Classes** - Understanding district-level data
- **Data Visualization** - Learning Chart.js and D3.js
- **Civics Projects** - Exploring urban infrastructure
- **Computer Science** - Web development and API integration
- **Urban Planning** - Mock city administration
- **AI/ML Courses** - Natural language processing examples

---

## 📄 License

MIT License - See LICENSE file for details

This project uses:
- [Chart.js](https://www.chartjs.org/) - MIT License
- [Font Awesome](https://fontawesome.com/) - CC BY 4.0
- [Claude API](https://anthropic.com/) - Commercial API

---

## 🔗 Useful Links

- **Census Data**: https://census2021.gov.in
- **India Data Portal**: https://data.gov.in
- **Ministry of Power**: https://powermin.gov.in
- **CPCB**: https://cpcb.nic.in
- **Chart.js Docs**: https://www.chartjs.org/docs/latest/
- **Claude API Docs**: https://docs.anthropic.com

---

## 👥 Support & Contact

For issues or questions:
- **GitHub Issues**: Create an issue in the repository
- **Discussions**: Start a discussion for general questions
- **Email**: [your-email@example.com]

---

## 🙏 Acknowledgments

- Census of India for demographic data
- Ministry of Power for energy infrastructure data
- IMD for meteorological data
- CPCB for air quality monitoring
- All contributors and testers

---

## 🎯 Roadmap

- [ ] v2.0: Real-time API integrations
- [ ] v2.1: Historical data & trends
- [ ] v2.2: Predictive analytics (ML models)
- [ ] v2.3: Multi-language support
- [ ] v2.4: Mobile app (React Native)
- [ ] v2.5: Blockchain for data verification
- [ ] v3.0: IoT sensor integration

---

**Built with ❤️ for India's Smart Cities Initiative**

*Last Updated: 2024 | India Digital Twin Project v1.0*
