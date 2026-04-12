# Corpometrics

> **Bioimpedance body composition tracker — PWA with Google Sheets backend and AI-powered calorie analysis**

[![Live App](https://img.shields.io/badge/Live%20App-ricgon--apps.github.io-34e8a8?style=flat-square&logo=github)](https://ricgon-apps.github.io/corpometrics/)
[![Backend](https://img.shields.io/badge/Backend-Google%20Apps%20Script-4285F4?style=flat-square&logo=google)](https://developers.google.com/apps-script)
[![License](https://img.shields.io/badge/License-MIT-f5c842?style=flat-square)](#license)

---

## Overview

**Corpometrics** is a personal health tracking Progressive Web App (PWA) designed for daily bioimpedance scale measurements. It provides a rich dashboard with body composition trends, historical analysis, and an AI-powered calorie analyser that estimates nutritional values from food photos.

**Problem it solves:** Bioimpedance scales provide raw numbers (weight, fat %, water %, muscle %) but no context, trends, or analysis. Corpometrics turns those daily readings into actionable insights with beautiful visualisations — without needing a dedicated mobile app or a paid subscription.

**Target users:** Individuals tracking body composition progress using a bioimpedance scale, who want persistent cloud storage, trend analysis, and AI nutritional guidance — all in one place.

---

## Features

- 📊 **Live Dashboard** — weight, BMI, body fat, muscle and water with delta indicators vs previous measurement
- 📉 **Trend Charts** — interactive sparklines for all metrics; tap to zoom with min/max/average stats
- 📋 **History Log** — full measurement history with per-entry edit and delete
- ➕ **Smart Entry Form** — accepts comma or period as decimal separator; iOS-keyboard friendly
- 🍽️ **AI Calorie Analyser** — photo-based food recognition powered by Claude (Anthropic API) via Google Apps Script proxy
- ☁️ **Google Sheets Backend** — data stored permanently in your own Google Sheet; no third-party database
- 📱 **Installable PWA** — add to iOS home screen via Safari; works offline with local cache fallback
- 🔒 **Privacy-first** — API keys stored in Google Apps Script properties, never exposed in client code
- 🔄 **Optimistic UI** — entries saved locally first, synced to Sheets in background

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML5 / CSS3 / JavaScript (no frameworks) |
| Hosting | GitHub Pages (static) |
| Backend | Google Apps Script (serverless) |
| Database | Google Sheets |
| AI / Vision | Anthropic Claude API (`claude-haiku-4-5`) |
| Fonts | Google Fonts — DM Serif Display + DM Mono |
| Icon | Custom SVG → PNG (180×180) |

---

## Architecture

```
┌─────────────────────────────────┐
│        iPhone / Browser         │
│                                 │
│  corpometrics PWA               │
│  (index.html — GitHub Pages)    │
│                                 │
│  ┌──────────┐  ┌─────────────┐  │
│  │ Dashboard│  │  Calorie    │  │
│  │ History  │  │  Analyser   │  │
│  │ Add Form │  │  (photo)    │  │
│  └────┬─────┘  └──────┬──────┘  │
└───────┼───────────────┼─────────┘
        │ fetch (GET)   │ fetch (POST + base64 image)
        ▼               ▼
┌───────────────────────────────────┐
│     Google Apps Script            │
│     (Code.gs — doGet / doPost)    │
│                                   │
│  Actions:                         │
│  • getAll  → read Sheets          │
│  • save    → write/update Sheets  │
│  • delete  → remove row           │
│  • analyzeFood → call Anthropic   │
│                                   │
│  API Key stored in                │
│  Script Properties (secure)       │
└───────┬───────────────────────────┘
        │
   ┌────┴────────────────┐
   │  Google Sheets      │  ←── "Medições" tab
   │  (user's account)   │       Date | Weight | Fat% | Water% | Muscle%
   └─────────────────────┘
```

The PWA communicates exclusively with a single Google Apps Script Web App URL. The script acts as a secure proxy — it reads/writes the user's Google Sheet and, for food analysis, calls the Anthropic API using a key stored in Script Properties (never visible to the client).

---

## Getting Started

### Prerequisites

- A **Google account** (for Sheets + Apps Script)
- An **Anthropic API key** (free tier available at [console.anthropic.com](https://console.anthropic.com)) — only required for the calorie analyser
- A **GitHub account** — only required if you want to self-host

### Installation

#### 1 — Fork / clone the repository

```bash
git clone https://github.com/ricgon-apps/corpometrics.git
cd corpometrics
```

#### 2 — Set up the Google Apps Script backend

1. Open [Google Sheets](https://sheets.google.com) and create a new spreadsheet named **Corpometrics**
2. Go to **Extensions → Apps Script**
3. Replace all code with the contents of `Code.gs`
4. Save the project (name it **Corpometrics**)
5. Edit `appsscript.json` to include the required OAuth scopes:

```json
{
  "timeZone": "Europe/Lisbon",
  "dependencies": {},
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8",
  "oauthScopes": [
    "https://www.googleapis.com/auth/spreadsheets",
    "https://www.googleapis.com/auth/script.external_request"
  ]
}
```

6. Go to **Project Settings → Script Properties** and add:

| Property | Value |
|---|---|
| `ANTHROPIC_KEY` | `sk-ant-...` (your Anthropic API key) |

7. Click **Deploy → New deployment**:
   - Type: **Web App**
   - Execute as: **Me**
   - Who has access: **Anyone**
8. Authorise the required permissions when prompted
9. Copy the deployment URL (ends in `/exec`)

#### 3 — Configure the frontend

Open `index.html` and update the default script URL constant:

```js
const DEFAULT_URL = "https://script.google.com/macros/s/YOUR_DEPLOYMENT_ID/exec";
```

#### 4 — Deploy to GitHub Pages

1. Push `index.html` and `icon.png` to your repository
2. Go to **Settings → Pages → Source: Deploy from branch → main**
3. Your app will be live at `https://YOUR-USERNAME.github.io/corpometrics/`

#### 5 — Install on iPhone

1. Open the app URL in **Safari** (not Chrome)
2. Tap the **Share** button → **Add to Home Screen**
3. The app installs as a native-looking PWA with a custom icon

---

## Usage

### Adding a measurement

Navigate to the **Medição** tab and enter:
- Date
- Weight (kg)
- Body fat (%)
- Total body water — TBW (%)
- Muscle mass (%)

Tap **Guardar medição** — the entry is saved locally immediately and synced to Google Sheets in the background.

### Calorie analyser

Navigate to the **Calorias** tab:
1. Tap the camera icon to take a photo or choose from the gallery
2. Tap **Analisar calorias**
3. The image is compressed client-side (max 900px, JPEG quality 0.75) before being sent to the Apps Script proxy, which calls the Anthropic Claude API
4. Results show: total kcal, macros (protein / carbs / fat / fibre), classification, healthier alternatives, and a nutritional note

### Chart zoom

On the Dashboard, tap any trend chart to open a full-screen modal with:
- Expanded chart with labelled data points
- Min / max / average statistics
- Total variation with directional colour coding

---

## API (Google Apps Script endpoints)

All requests go to the single deployment URL. Actions are passed as query parameters (GET) or form-encoded body (POST).

| Action | Method | Parameters | Description |
|---|---|---|---|
| `getAll` | GET | `action=getAll` | Returns all measurements as JSON |
| `save` | GET | `action=save&date=YYYY-MM-DD&weight=X&fat=X&tbw=X&mus=X` | Creates or updates an entry |
| `delete` | GET | `action=delete&date=YYYY-MM-DD` | Deletes an entry by date |
| `analyzeFood` | POST | `action=analyzeFood&mediaType=image/jpeg&imageBase64=...` | Sends image to Anthropic API and returns nutritional JSON |

**Response format (getAll):**
```json
{
  "ok": true,
  "entries": [
    { "date": "2026-03-15", "weight": 95.9, "fat": 29.0, "tbw": 53.7, "mus": 30.1 }
  ]
}
```

**Response format (analyzeFood):**
```json
{
  "ok": true,
  "result": {
    "nome": "Maçã, pêra, amêndoas, iogurte proteico",
    "porcao": "~500g total",
    "calorias": 520,
    "proteinas": 28,
    "carboidratos": 55,
    "gorduras": 22,
    "fibra": 9,
    "classificacao": "saudavel",
    "alternativas": ["..."],
    "nota": "Refeição equilibrada e rica em proteína."
  }
}
```

---

## Folder Structure

```
corpometrics/
├── index.html      # Complete PWA — all HTML, CSS and JS in one file
├── Code.gs         # Google Apps Script backend (copy to Apps Script editor)
├── icon.png        # PWA home screen icon (180×180 PNG)
└── README.md       # This file
```

> **Note:** The entire frontend is intentionally a single `index.html` file with no build step, no bundler, and no dependencies. This keeps deployment trivial (GitHub Pages serves it directly) and makes the codebase easy to audit.

---

## Configuration

### Frontend (`index.html`)

| Constant | Description |
|---|---|
| `DEFAULT_URL` | Fallback Apps Script URL if none stored in `localStorage` |
| `LS_URL` | `localStorage` key for the script URL (`cm_script_url`) |
| `LS_DATA` | `localStorage` key for the offline data cache (`cm_entries_cache`) |

### Backend (`Code.gs`)

| Script Property | Description |
|---|---|
| `ANTHROPIC_KEY` | Anthropic API key — stored securely in Apps Script, never sent to client |

### User profile (hardcoded)

The current version has the user profile hardcoded for a **50-year-old male, 1.82 m** tall. Reference ranges for body fat, muscle mass, and water are calibrated accordingly. To change this, search for `1.82` and `50` in `index.html`.

---

## Contributing

Contributions are welcome! To get started:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-improvement`
3. Make your changes to `index.html` and/or `Code.gs`
4. Test locally by opening `index.html` in a browser (configure your own Apps Script URL)
5. Submit a pull request with a clear description of the change

**Before submitting, please ensure:**
- JavaScript syntax is valid (run `node --check index.html` or check browser console)
- HTML entities (`&#x...`) are not used inside `<script>` tags — use actual UTF-8 characters or `\uXXXX` escape sequences instead
- The app works on mobile Safari (iOS) — this is the primary target platform

---

## Roadmap / Future Improvements

- [ ] **AI nutritional recommendations** — integrate Gemini API to generate personalised diet and exercise plans based on body composition history and calorie log
- [ ] **User profile settings** — make height, age, and sex configurable from the UI
- [ ] **Multiple users** — support for family members or multiple profiles
- [ ] **Export data** — download measurements as CSV or PDF report
- [ ] **Offline sync queue** — queue failed API calls and retry when back online
- [ ] **Push notifications** — daily reminder to log measurements (Web Push API)
- [ ] **Dark/light theme toggle**
- [ ] **PWA manifest** — add `manifest.json` for richer install experience and splash screen

---

## Known Limitations

- **Single-file architecture** — `index.html` contains all HTML, CSS, and JS (~600 lines). Maintainable at current scale but will need splitting if the app grows significantly
- **No authentication** — the Apps Script URL is effectively a public API endpoint. Anyone with the URL can read and write data. Suitable for personal use; not recommended for shared or sensitive deployments
- **iOS Safari only** — PWA install ("Add to Home Screen") works best in Safari on iOS. Chrome on iOS does not fully support PWA installation
- **Image compression** — food photos are resized to max 900px and compressed to JPEG 0.75 quality before analysis. Very detailed images may still exceed API limits
- **Anthropic API cost** — the calorie analyser uses `claude-haiku-4-5` (~$0.001 per analysis). A $5 credit covers thousands of analyses
- **Date handling** — Google Sheets sometimes returns dates as JavaScript Date objects rather than strings. The `toYMD()` function normalises these, but edge cases may exist in non-UTC timezones

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

## Acknowledgements

Built iteratively with [Claude](https://claude.ai) (Anthropic) as an AI pair programmer.  
Data stored securely in [Google Sheets](https://sheets.google.com) via [Google Apps Script](https://developers.google.com/apps-script).  
Hosted for free on [GitHub Pages](https://pages.github.com).
