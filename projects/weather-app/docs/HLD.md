# High Level Design — Weather App

| Field | Value |
|---|---|
| Document ID | WTHR-HLD-001 |
| Version | 1.0 |
| Status | Draft |
| Author | Platform Team |
| References | WTHR-URS-001 |

---

## 1. Architecture Overview

The Weather App is a static SPA that fetches weather data from a commercial weather API via a thin serverless proxy (to protect the API key from browser exposure). User preferences and saved locations are stored in the browser's `localStorage`. Severe alert polling runs as a background service worker. The app is deployed to GitHub Pages.

```mermaid
graph TB
    subgraph Browser
        SPA[React SPA]
        SW[Service Worker\nalert polling]
        LS[localStorage\npreferences & locations]
    end

    subgraph GitHub
        AUTH[GitHub OAuth]
        PAGES[GitHub Pages CDN]
    end

    subgraph Serverless
        PROXY[API Proxy\nCloudflare Worker]
    end

    subgraph External
        OWM[OpenWeatherMap API]
        NWS[NWS / Environment Canada\nAlert Feed]
    end

    SPA -->|Device Flow| AUTH
    AUTH -->|token| SPA
    SPA -->|forecast requests| PROXY
    PROXY -->|keyed request| OWM
    OWM -->|JSON response| PROXY
    PROXY -->|sanitised response| SPA
    SW -->|poll alerts| NWS
    SW -->|push notification| Browser
    SPA <-->|read/write prefs| LS
    PAGES --> SPA
```

---

## 2. Component Architecture

```mermaid
graph TD
    App --> AuthGate
    AuthGate --> Layout

    Layout --> LocationHeader
    Layout --> CurrentConditions
    Layout --> HourlyForecast
    Layout --> DailyForecast
    Layout --> AlertBanner
    Layout --> Sidebar

    Sidebar --> FavouritesList
    Sidebar --> LocationSearch
    Sidebar --> UnitToggle

    CurrentConditions --> ConditionsGrid
    CurrentConditions --> WeatherIcon

    HourlyForecast --> HourlyChart
    HourlyForecast --> HourlyScroll

    subgraph Data Layer
        useWeather
        useAlerts
        useLocations
        usePreferences
    end

    CurrentConditions --> useWeather
    HourlyForecast --> useWeather
    DailyForecast --> useWeather
    AlertBanner --> useAlerts
    FavouritesList --> useLocations
    UnitToggle --> usePreferences
```

---

## 3. Data Flow

### 3.1 Forecast Data Flow

```mermaid
sequenceDiagram
    participant SPA
    participant Cache as TanStack Query Cache
    participant Proxy as CF Worker Proxy
    participant OWM as OpenWeatherMap

    SPA->>Cache: useWeather(location)
    alt cache hit (< 10min old)
        Cache-->>SPA: cached data
    else cache miss / stale
        Cache->>Proxy: GET /forecast?lat=X&lon=Y
        Proxy->>OWM: GET /data/3.0/onecall?lat=X&lon=Y&appid=SECRET
        OWM-->>Proxy: full weather payload
        Proxy-->>Cache: sanitised payload (no API key)
        Cache-->>SPA: fresh data
    end
```

### 3.2 Alert Polling Flow

```mermaid
sequenceDiagram
    participant SW as Service Worker
    participant NWS as NWS/EC Alert Feed
    participant Browser

    loop Every 5 minutes
        SW->>NWS: GET alerts for saved locations (Atom/GeoJSON)
        NWS-->>SW: active alerts
        SW->>SW: compare with known alerts
        alt new alert detected
            SW->>Browser: showNotification(alert headline)
        end
    end
```

---

## 4. Technology Stack

| Layer | Choice | Rationale |
|---|---|---|
| Frontend framework | React 18 | Component model, rich ecosystem |
| Build tool | Vite | Fast dev, static output for GitHub Pages |
| Language | TypeScript (strict) | Type safety across API response shapes |
| Styling | Tailwind CSS | Utility-first, no runtime |
| Charts | Recharts | Composable, accessible charts for forecast visualisation |
| Server state | TanStack Query | Caching with stale-time, background refetch |
| Hosting | GitHub Pages | Zero-cost static hosting |
| Auth | GitHub OAuth Device Flow | No redirect server; works with static hosting |
| API proxy | Cloudflare Worker | Free tier, edge-cached, protects API key |
| Alert polling | Service Worker | Background polling without tab needing to be focused |
| Weather data | OpenWeatherMap One Call 3.0 | Comprehensive: current + hourly + daily + alerts |

---

## 5. Deployment Architecture

```mermaid
graph LR
    subgraph weather-app repo
        SRC[Source Code]
        GHA[GitHub Actions]
        PAGES[GitHub Pages\ngh-pages branch]
    end

    subgraph Cloudflare
        WORKER[API Proxy Worker\nweather-proxy.example.workers.dev]
    end

    SRC -->|push to main| GHA
    GHA -->|vite build + gh-pages deploy| PAGES
    PAGES -->|serve| Browser
    Browser -->|forecast req| WORKER
    WORKER -->|OWM API key| External[OpenWeatherMap]
```

The Cloudflare Worker acts as a thin, stateless proxy — it adds the `appid` query parameter and forwards the response. No data is stored in the worker. The API key is stored as a Cloudflare Worker Secret.

---

## 6. Key Design Decisions

- **API proxy instead of direct browser calls:** Exposing the weather API key in the browser would allow abuse and cost overruns. A serverless proxy adds negligible latency while protecting the key.
- **Service Worker for alert polling:** Polling from the main SPA thread would stop when the tab is hidden on mobile. A Service Worker continues running and can deliver push notifications even when the app is backgrounded.
- **localStorage for user preferences:** Weather preferences (unit, saved locations, default) are lightweight and user-specific. Using GitHub Discussions or Issues for this would over-engineer a simple problem.
- **OpenWeatherMap One Call 3.0:** Single API call returns current conditions, hourly, daily, minutely precipitation, and national weather alerts — minimising round trips and simplifying the data model.
