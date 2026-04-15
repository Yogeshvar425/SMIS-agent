# 🛡️ Commute Safety Pulse — Smart Mobility AI Agent

> An AI-powered smart mobility assistant that helps urban commuters pick the **safest route** by scoring risk across **walk, bike, transit, and drive** modes using a **time-aware, multi-factor simulation engine**, with intelligent recommendations from **Google Gemini**.

---

## 📌 Chosen Vertical

**Smart Mobility / Urban Commute Safety**

Urban commuters face daily risk decisions — traffic congestion, incidents, poor air quality, and route safety. This agent consolidates those signals into an actionable, at-a-glance dashboard with AI-powered route recommendations.

---

## 🧠 Approach & Logic

### Problem
Commuters lack a unified view of risk factors across multiple travel modes. Existing tools show traffic *or* transit *or* weather — but never a combined **risk score** that compares walking, cycling, transit, and driving side by side.

### Solution Design

#### 1. Time-Aware Simulation Engine
Instead of random data, the system uses a **Gaussian rush-hour model** to generate realistic, correlated metrics:

```
Time-of-Day Factor (Gaussian peaks at 8:30 AM and 5:30 PM)
    ↓
Congestion Index (0-99, driven by time factor)
    ↓
Incidents (correlated with congestion, not random)
    ↓
AQI (derived from congestion + incident density)
    ↓
Route Risk Scores (multi-factor weighted model)
```

The key insight: **all metrics are logically coherent**. High congestion during morning rush produces more incidents, worse air quality, and higher risk scores — especially for congestion-sensitive modes like Drive.

#### 2. Multi-Factor Weighted Risk Scoring
Each route's risk score is computed from 4 weighted factors:

| Factor | Weight | Description |
|---|---|---|
| Congestion × Mode Sensitivity | 45% | Drive is 6× more affected by traffic than Walk |
| Incident Density | 25% | Nearby traffic incidents raise risk |
| Air Quality Penalty | 15% | AQI > 50 adds risk, especially for Walk/Bike |
| Route Intrinsic Offset | 15% | Protected bike lanes are inherently safer than busy avenues |

```javascript
score = baseRisk + (congestion × modeSensitivity × 0.45)
      + (incidents × 8 × 0.25)
      + (max(0, (AQI - 50) × 0.2) × 0.15)
```

#### 3. Color-Coded Severity
Instant visual triage:
- 🟢 **Green** (Low): score < 35
- 🟡 **Amber** (Moderate): score 35–64
- 🔴 **Red** (High): score ≥ 65

#### 4. Temporally Continuous Forecasting
The 2-hour forecast chart uses the same Gaussian rush-hour model projected into the future, with **60/40 exponential smoothing** to create smooth, believable trend curves instead of random jumps.

#### 5. Structured AI Prompt Engineering
The Gemini prompt uses 5 prompt engineering principles:
1. **ROLE** — "Smart Mobility Safety Advisor" persona
2. **CONTEXT** — Full metrics snapshot with semantic labels (time of day, AQI category)
3. **DATA** — All 3 routes with scores and ETAs
4. **TASK** — Specific multi-part instruction (recommend, suggest alternative, safety tip)
5. **FORMAT** — "Exactly 3 short sentences, plain text only"

#### 6. Google Maps with Live Directions
The Google Maps JavaScript API with **DirectionsService** renders real route paths between Bangalore waypoints:
- **Walk:** Majestic → Cubbon Park
- **Bike:** Majestic → Koramangala  
- **Transit:** Majestic → Whitefield
- **Drive:** Majestic → Electronic City

The map dynamically switches `TravelMode` (WALKING / BICYCLING / TRANSIT / DRIVING) when the user changes mode, and displays real distance and duration from the Directions API response.

### Architecture
```
┌──────────────────────────────────────────────────┐
│              index.html (SPA)                    │
├──────────────────────────────────────────────────┤
│  Simulation Engine         │  UI Layer           │
│  ─ Gaussian rush-hour model│  ─ Mode chips       │
│  ─ Correlated metrics      │  ─ 3 Metric cards   │
│  ─ Multi-factor risk scores│  ─ 3 Route cards    │
│  ─ Temporal forecast       │  ─ Forecast chart   │
│  ─ Weighted scoring model  │  ─ Google Map       │
│                            │  ─ AI response box  │
├──────────────────────────────────────────────────┤
│  Google Services                                 │
│  ─ Gemini 2.0 Flash (structured prompt AI)       │
│  ─ Google Maps JS API + DirectionsService        │
│  ─ Google Maps Places Library                    │
│  ─ Google Fonts (Inter typeface)                 │
├──────────────────────────────────────────────────┤
│  Quality & Safety                                │
│  ─ 45+ automated self-tests                      │
│  ─ CSP headers, input sanitization               │
│  ─ AbortController for API race conditions       │
│  ─ Page Visibility API (pause on tab hide)       │
│  ─ Rate limiting (5s refresh cooldown)            │
│  ─ Full ARIA/WCAG 2.1 accessibility              │
└──────────────────────────────────────────────────┘
```

---

## 🚀 How It Works

### Setup
1. Clone this repository
2. Open `index.html` in any modern browser (Chrome, Firefox, Edge)
3. Enter your **Google Gemini API Key** in the input field at the top
4. The dashboard loads immediately with time-aware simulated data

### Features
| Feature | Description |
|---|---|
| **Mode Selector** | Chip buttons to switch between Walk, Bike, Transit, and Drive |
| **3 Metric Cards** | Congestion Index, Incidents Nearby, Air Quality (AQI) — all correlated |
| **3 Route Cards** | Per mode — name, ETA, risk bar, severity tag, and **risk factor breakdown** |
| **Forecast Chart** | 6-bar chart with rush-hour-aware temporal continuity and hover tooltips |
| **Live Route Map** | Google Maps with real DirectionsService route rendering, distance + duration info |
| **Route Info Panel** | Shows real distance, duration, and route label from Directions API |
| **AI Recommendation** | Context-rich, structured Gemini prompt with format constraints |
| **Refresh Button** | Regenerates data + re-calls Gemini (5s cooldown to prevent abuse) |
| **Self-Test Suite** | 45+ automated tests including correlation and prompt engineering validation |

### Gemini API Prompt (Structured)
```
You are a Smart Mobility Safety Advisor for urban commuters.

CURRENT CONDITIONS (morning rush, 4/15/2026):
• Travel mode: Drive
• Congestion index: 72/100
• Nearby incidents: 3
• Air quality (AQI): 86 (Moderate)

ROUTE OPTIONS (sorted by risk, best first):
1. Toll Road — Risk: 38/100 (moderate), ETA: 25 min
2. Local Roads — Risk: 45/100 (moderate), ETA: 40 min
3. Highway 5 — Risk: 68/100 (high), ETA: 30 min

TASK: Based on these conditions, provide:
1. A clear recommendation for the top route
2. If risk is moderate or high, suggest a specific alternative
3. One practical safety tip relevant to Drive mode during morning rush

FORMAT: Reply in exactly 3 short sentences. No markdown, no bullets, plain text only.
```

### Response Handling
```javascript
data.candidates[0].content.parts[0].text → displayed in AI box
```

---

## 🔐 Security Considerations

- **API Key** — Never hardcoded; entered at runtime, stored only in a JS variable
- **Content Security Policy** — Strict CSP meta tag restricts all resource origins
- **Input Sanitization** — `sanitize()` function + `textContent` for all dynamic content
- **Rate Limiting** — 5-second cooldown on Refresh to prevent API quota abuse
- **AbortController** — Cancels in-flight API requests on mode switch to prevent race conditions
- **API Key Validation** — Length check before making API calls

---

## ♿ Accessibility

- **Skip Navigation** link for keyboard users
- **Semantic HTML5** — `<header>`, `<main>`, `<section>`, proper heading hierarchy
- **ARIA Labels** — All interactive elements and data displays have descriptive labels
- **`role="progressbar"`** — Risk bars include `aria-valuenow`, `aria-valuemin`, `aria-valuemax`
- **`aria-live="polite"`** — Dynamic content regions announce updates to screen readers
- **`aria-pressed`** — Mode chips communicate toggle state
- **Focus Styles** — Visible `:focus-visible` outlines on all interactive elements
- **Screen Reader Chart Summary** — Hidden text provides a textual summary of chart data
- **Color + Text** — Risk severity is communicated through both color and labels (color-blind safe)

---

## 🔧 Google Services Integration

| Service | Usage | Why |
|---|---|---|
| **Google Gemini 2.0 Flash** | Context-rich route recommendations via structured prompts | Fast inference, good at following format constraints |
| **Google Maps JavaScript API** | Interactive map with real route rendering | Full DirectionsService + DirectionsRenderer |
| **Google Maps DirectionsService** | Computes real routes between Bangalore waypoints | Provides actual distance, duration, and polyline |
| **Google Maps Places Library** | Places support for map interactions | Loaded alongside Maps JS API |
| **Google Fonts (Inter)** | Professional typography | Clean, accessible at all sizes |

---

## 📋 Assumptions

1. **Simulated Data** — Metrics use a time-aware Gaussian model (not random numbers). In production, these would connect to Google Maps Directions API, OpenAQ, and TomTom Traffic API.
2. **Single File** — The entire application is contained in one `index.html`. The Google Maps JS API is loaded dynamically at runtime.
3. **API Key Scope** — The user's API key needs access to the Generative Language API, Maps JavaScript API, and Directions API.
4. **Modern Browser** — Requires ES6+, CSS Custom Properties, Fetch API, and Page Visibility API.
5. **Route Data** — Route names and base ETAs are illustrative; risk scores are computed, not random.

---

## 🧪 Testing

### Built-in Self-Test (45+ assertions)
Open your browser's developer console and run:
```javascript
runSelfTest()
```

This validates:
- ✅ All required DOM elements exist
- ✅ Risk color and tag mapping across boundary values
- ✅ Route data generation (count, sorting, value ranges, types)
- ✅ **Risk factor breakdown objects** exist on all routes
- ✅ Metric values are within valid ranges
- ✅ **Data correlation** — high congestion produces incidents in >50% of trials
- ✅ **Time-of-day factor** validity (0.15–1.0 range)
- ✅ **Forecast temporal continuity** — max bar-to-bar jump < 40
- ✅ Mode chips, chart bars, and route cards rendered correctly
- ✅ **Risk breakdown UI** displayed on all route cards
- ✅ Accessibility landmarks and ARIA attributes present
- ✅ **Prompt engineering** — role, all 3 routes, AQI, format constraint, time context
- ✅ Constants match expected values

### Manual Testing
- Switch between all 4 modes and confirm routes update with mode-appropriate risk levels
- Enter API key → click Refresh → confirm AI response appears in 3 sentences
- Rapid-click Refresh to verify cooldown enforcement
- Tab through all interactive elements to verify keyboard navigation
- Hover over forecast chart bars to see time tooltips
- Check risk factor breakdown on each route card
- Verify map loads with directions and route info (distance + duration)
- Switch modes and confirm map re-renders with correct TravelMode

---

## 📁 Project Structure

```
SMIS-agent/
├── index.html     ← Complete single-file application (1147 lines)
├── README.md      ← This file
└── .gitignore     ← Standard ignores
```

---

## 📄 License

This project was built for the Google Antigravity Challenge.
