# Low Level Design — Weather App

| Field | Value |
|---|---|
| Document ID | WTHR-LLD-001 |
| Version | 1.0 |
| Status | Draft |
| Author | Platform Team |
| References | WTHR-HLD-001 |

---

## 1. Module Breakdown

### 1.1 `useWeather` Hook

Fetches the One Call 3.0 response for a given lat/lon. Normalises the raw API response into the app's internal types.

**Interface:**
```typescript
interface WeatherData {
  current: CurrentConditions
  hourly: HourlyPoint[]     // 48 hours
  daily: DailyForecast[]    // 8 days
  alerts: WeatherAlert[]
}

function useWeather(
  location: LatLon | null
): UseQueryResult<WeatherData, Error>
```

**Query key:** `['weather', location.lat, location.lon]`
**Stale time:** 10 minutes
**Cache time:** 30 minutes (data remains accessible offline for this window)

**Normalisation:**
- Unix timestamps → ISO 8601 strings
- Temperature rounded to 1 decimal
- Wind degrees → cardinal direction string (`'NW'`, `'SSE'`, etc.)
- Precipitation probability `0–1` → percentage `0–100`

---

### 1.2 `useAlerts` Hook

Subscribes to severe weather alerts for the user's saved locations. Wraps the Service Worker message channel.

```typescript
interface WeatherAlert {
  id: string
  event: string         // e.g. "Severe Thunderstorm Warning"
  severity: 'extreme' | 'severe' | 'moderate' | 'minor' | 'unknown'
  headline: string
  description: string
  start: string         // ISO 8601
  end: string
  areas: string[]
}

function useAlerts(locations: SavedLocation[]): {
  alerts: WeatherAlert[]
  latestCheck: string | null
}
```

---

### 1.3 `useLocations` Hook

Manages user's saved locations in `localStorage`. Supports search via the geocoding API.

```typescript
interface SavedLocation {
  id: string            // stable UUID
  name: string          // "New York, NY"
  lat: number
  lon: number
  isDefault: boolean
  isPinned: boolean     // org-wide pinned (read-only for non-admins)
}

function useLocations(): {
  locations: SavedLocation[]
  defaultLocation: SavedLocation | null
  addLocation: (loc: SavedLocation) => void
  removeLocation: (id: string) => void
  setDefault: (id: string) => void
}
```

Location search uses the OpenWeatherMap Geocoding API (`/geo/1.0/direct?q={name}&limit=5`) proxied through the Cloudflare Worker.

---

### 1.4 `HourlyChart` Component

Renders a 24-hour precipitation + temperature chart using Recharts.

**Props:**
```typescript
interface HourlyChartProps {
  hours: HourlyPoint[]    // first 24 entries from hourly[]
  unit: 'metric' | 'imperial'
}
```

**Chart structure:**
- X-axis: time labels (every 3 hours: 6AM, 9AM, 12PM…)
- Left Y-axis: temperature (line chart)
- Right Y-axis: precipitation probability % (bar chart, semi-transparent)
- Tooltip: shows all values on hover

---

### 1.5 Alert Service Worker (`alert-sw.ts`)

Runs independently of the SPA. Registered on first app load. Polls NWS/Environment Canada GeoJSON feed every 5 minutes for each saved location.

**Algorithm:**
```
knownAlertIds = Set loaded from IndexedDB

every 5 minutes:
  for each saved location:
    fetch NWS GeoJSON alerts for lat/lon
    for each alert in response:
      if alert.id not in knownAlertIds:
        showNotification(alert.headline, { body: alert.event, icon })
        knownAlertIds.add(alert.id)
  persist knownAlertIds to IndexedDB
```

Expired alerts are pruned from `knownAlertIds` based on the alert's `expires` field.

---

## 2. Data Model

### 2.1 CurrentConditions

```typescript
interface CurrentConditions {
  dt: string              // ISO 8601
  temp: number            // °C or °F (normalised per user pref)
  feelsLike: number
  humidity: number        // 0-100 %
  windSpeed: number       // m/s or mph
  windDirection: string   // cardinal, e.g. 'NW'
  windGust: number | null
  uvIndex: number
  visibility: number      // meters
  description: string     // "Partly Cloudy"
  icon: string            // OWM icon code, e.g. "02d"
  precipitation: number   // mm in last hour
}
```

### 2.2 HourlyPoint

```typescript
interface HourlyPoint {
  dt: string
  temp: number
  feelsLike: number
  precipProbability: number   // 0-100 %
  precipAmount: number        // mm
  windSpeed: number
  windDirection: string
  description: string
  icon: string
}
```

### 2.3 DailyForecast

```typescript
interface DailyForecast {
  dt: string              // date only (YYYY-MM-DD)
  tempHigh: number
  tempLow: number
  precipProbability: number
  precipAmount: number
  windSpeed: number
  description: string
  icon: string
  sunrise: string
  sunset: string
  uvIndex: number
}
```

---

## 3. Cloudflare Worker Proxy

### 3.1 Routes

| Method | Path | Proxied To |
|---|---|---|
| GET | `/forecast` | `api.openweathermap.org/data/3.0/onecall` |
| GET | `/geocode` | `api.openweathermap.org/geo/1.0/direct` |
| GET | `/reverse-geocode` | `api.openweathermap.org/geo/1.0/reverse` |

### 3.2 Worker Logic

```typescript
// Pseudocode
export default {
  fetch(request, env) {
    const url = new URL(request.url)
    const upstream = buildUpstreamUrl(url.pathname, url.searchParams, env.OWM_API_KEY)
    const response = await fetch(upstream, { cf: { cacheTtl: 600 } })
    return new Response(response.body, {
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': env.ALLOWED_ORIGIN,
        'Cache-Control': 'public, max-age=600',
      },
    })
  },
}
```

The worker adds `appid={OWM_API_KEY}` and enforces an `ALLOWED_ORIGIN` check to prevent third-party use of the proxy.

---

## 4. Error Handling

| Scenario | Handling |
|---|---|
| Geolocation denied by user | Fall back to default saved location; no error shown |
| Weather API 429 (rate limit) | Show stale cached data with "Data may be delayed" badge |
| Weather API 5xx | Display last cached data + error banner with time of last update |
| Worker unreachable | Show full offline state; display last data from TanStack Query cache |
| Service worker registration failed | Alerts degraded to visible in-app banner only; no push notifications |
| NWS alert feed unavailable | Suppress alert banner; log warning; retry next poll cycle |

---

## 5. Performance Considerations

- **One Call 3.0:** Single API call returns all data types — current, hourly, daily, alerts — eliminating waterfall requests
- **Edge caching:** Cloudflare Worker caches OWM responses for 10 minutes at the edge — a CDN cache hit costs ~1ms vs ~200ms to origin
- **Lazy chart import:** `recharts` imported only when the hourly chart panel is in view (Intersection Observer + `React.lazy`)
- **Icon sprites:** OWM weather icons pre-downloaded as an SVG sprite and served from the same Pages origin — no icon fetch per condition
