# API Specification — Weather App

| Field | Value |
|---|---|
| Document ID | WTHR-API-001 |
| Version | 1.0 |
| Status | Draft |
| External API | OpenWeatherMap One Call 3.0 |
| Proxy | Cloudflare Worker (`/api/*`) |

---

## 1. Internal API (Cloudflare Worker Proxy)

All requests from the SPA go to the Cloudflare Worker proxy. The proxy adds the OWM API key and forwards the response. No OWM API key is exposed to the browser.

**Base URL:** `https://weather-proxy.{your-org}.workers.dev`

**Auth:** None required from the SPA — the proxy validates `Origin` header.

---

### GET /forecast

Returns current conditions, 48-hour hourly forecast, 8-day daily forecast, and active alerts for a coordinate.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `lat` | number | Yes | Latitude, decimal degrees |
| `lon` | number | Yes | Longitude, decimal degrees |
| `units` | string | No | `metric` (default) or `imperial` |
| `lang` | string | No | Language code, default `en` |

**Example request:**
```
GET /forecast?lat=40.7128&lon=-74.0060&units=metric
```

**Response shape:**
```json
{
  "lat": 40.7128,
  "lon": -74.006,
  "timezone": "America/New_York",
  "current": {
    "dt": 1751055600,
    "temp": 22.4,
    "feels_like": 21.8,
    "humidity": 65,
    "wind_speed": 4.2,
    "wind_deg": 270,
    "wind_gust": 7.1,
    "uvi": 5.2,
    "visibility": 10000,
    "weather": [{ "id": 802, "main": "Clouds", "description": "scattered clouds", "icon": "03d" }],
    "rain": { "1h": 0 }
  },
  "hourly": [
    {
      "dt": 1751055600,
      "temp": 22.4,
      "feels_like": 21.8,
      "wind_speed": 4.2,
      "wind_deg": 270,
      "pop": 0.12,
      "weather": [{ "id": 802, "main": "Clouds", "description": "scattered clouds", "icon": "03d" }]
    }
    // ... 47 more
  ],
  "daily": [
    {
      "dt": 1751040000,
      "sunrise": 1751019000,
      "sunset": 1751072400,
      "temp": { "day": 23.1, "min": 17.2, "max": 25.8, "night": 18.4 },
      "pop": 0.25,
      "rain": 0.5,
      "wind_speed": 5.1,
      "wind_deg": 260,
      "uvi": 6.1,
      "weather": [{ "id": 500, "main": "Rain", "description": "light rain", "icon": "10d" }]
    }
    // ... 7 more
  ],
  "alerts": [
    {
      "sender_name": "NWS New York",
      "event": "Dense Fog Advisory",
      "start": 1751040000,
      "end": 1751068800,
      "description": "Visibility below 1/4 mile expected...",
      "tags": ["Fog"]
    }
  ]
}
```

**Error responses:**

| Code | Meaning |
|---|---|
| 400 | Missing or invalid `lat`/`lon` |
| 403 | Request `Origin` not in allowlist |
| 429 | OWM rate limit reached (proxy passes through) |
| 502 | OWM API unreachable |

---

### GET /geocode

Converts a place name to coordinates (forward geocoding).

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `q` | string | Yes | City name, optionally with state/country: `London,GB` |
| `limit` | number | No | Max results (default 5, max 10) |

**Example:**
```
GET /geocode?q=Portland,OR,US&limit=5
```

**Response:**
```json
[
  {
    "name": "Portland",
    "lat": 45.5051,
    "lon": -122.675,
    "country": "US",
    "state": "Oregon"
  }
]
```

---

### GET /reverse-geocode

Converts coordinates to a human-readable location name.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `lat` | number | Yes | Latitude |
| `lon` | number | Yes | Longitude |

**Example:**
```
GET /reverse-geocode?lat=45.5051&lon=-122.675
```

**Response:**
```json
[
  {
    "name": "Portland",
    "state": "Oregon",
    "country": "US"
  }
]
```

---

## 2. NWS Alert Feed (Direct — Service Worker)

The Service Worker polls the NWS (National Weather Service) CAP/GeoJSON feed directly from the browser — no proxy needed, as this is a public unauthenticated endpoint.

**Endpoint:**
```
GET https://api.weather.gov/alerts/active
  ?point={lat},{lon}
  &status=actual
  &message_type=alert
```

**Headers required:**
```
User-Agent: WeatherApp/1.0 (your-org.example.com; ops@your-org.example.com)
Accept: application/geo+json
```

NWS requires a descriptive `User-Agent` — requests without it are rejected.

**Key response fields:**

```json
{
  "features": [
    {
      "properties": {
        "id": "urn:oid:2.49.0.1.840.0.abc123",
        "areaDesc": "New York City",
        "severity": "Severe",
        "certainty": "Likely",
        "event": "Severe Thunderstorm Warning",
        "headline": "Severe Thunderstorm Warning issued...",
        "description": "...",
        "onset": "2026-06-28T14:00:00-04:00",
        "expires": "2026-06-28T16:45:00-04:00"
      }
    }
  ]
}
```

---

## 3. Rate Limits & Quotas

| API | Limit | Notes |
|---|---|---|
| OWM One Call 3.0 | 1,000 calls/day (free), 1M/month (paid) | Proxy edge-cached 10 min |
| OWM Geocoding | 1,000 calls/day (free) | Not cached; triggered by user search |
| NWS Alerts | No stated limit | Public gov endpoint; courteous polling at 5-min intervals |

**Cost control measures:**
- Proxy Cloudflare cache (`cacheTtl: 600`) — repeat requests within 10 min are free
- TanStack Query `staleTime: 600_000` — SPA does not request until stale
- Geocoding debounce: 500ms after last keystroke before issuing search request
