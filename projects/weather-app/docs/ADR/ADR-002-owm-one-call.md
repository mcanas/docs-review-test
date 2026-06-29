# ADR-002: Use OpenWeatherMap One Call 3.0 as the Weather Data Source

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-06-28 |
| Deciders | Platform Team |

---

## Context

The Weather App requires current conditions, hourly forecasts, daily forecasts, and severe weather alerts — all for arbitrary coordinates worldwide. Several commercial weather APIs were evaluated.

| Provider | Current | Hourly | Daily | Alerts | Price (1k calls/day) | Notes |
|---|---|---|---|---|---|---|
| OpenWeatherMap One Call 3.0 | ✓ | 48h | 8d | NWS/NOAA | Free up to 1k/day | Single call for all data |
| Tomorrow.io | ✓ | 120h | 15d | ✓ | Free up to 500/day | Better accuracy; stricter limits |
| WeatherAPI.com | ✓ | 24h | 14d | ✓ (US/UK/AU) | Free up to 1M/month | Good free tier; less global coverage |
| Weatherbit | ✓ | 48h | 16d | ✓ | $35/month | Most comprehensive; no free tier |

---

## Decision

Use **OpenWeatherMap One Call 3.0**.

Key factors:
1. **Single endpoint for all data types** — current + hourly + daily + alerts in one HTTP call. Competing APIs require separate calls for each data type, increasing complexity and quota consumption.
2. **Generous free tier** — 1,000 calls/day is sufficient for our user base with Cloudflare edge caching (10-minute TTL).
3. **Global coordinate support** — no regional restrictions on coordinates.
4. **Mature SDK ecosystem** — well-documented, stable API since 2020.

---

## Consequences

**Positive:**
- Single API call per location update simplifies the data layer significantly
- 48-hour hourly data satisfies the 24-hour requirement with headroom
- Free tier eliminates budget spend until user base scales beyond ~700 daily users (assuming 10-minute cache hit rate of 80%)

**Negative:**
- OWM accuracy is rated lower than Tomorrow.io in rural/remote areas — acceptable for v1
- Alert data is sourced from NWS (US only) — international teams will need to rely on the direct NWS/EC feed in the Service Worker
- Forecast horizon is 8 days (not the 14 days some teams requested) — deferred to v2

**Review trigger:** If the user base exceeds 700 daily active users, evaluate upgrading to a paid OWM plan or migrating to Tomorrow.io.
