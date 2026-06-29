# Business Requirements Document — Weather App

| Field | Value |
|---|---|
| Document ID | WTHR-BRD-001 |
| Version | 1.0 |
| Status | Draft |
| Author | Product Team |
| Date | 2026-06-28 |

---

## 1. Executive Summary

Field operations, logistics, and facilities teams within the organisation make time-sensitive decisions that depend on weather conditions — scheduling outdoor work, routing deliveries, managing energy loads in buildings. Currently these teams use separate consumer weather apps with no enterprise integration, no audit trail, and no ability to embed weather context into operational workflows. The Weather App will provide a branded, integrated weather dashboard that surfaces hyperlocal forecasts and historical conditions directly within the organisation's internal tooling, backed by a reliable commercial weather data provider.

---

## 2. Business Context

### 2.1 Problem Statement

Operations teams report that weather-related scheduling decisions are made inconsistently:
- Each team member uses a different consumer app (AccuWeather, Weather.com, iPhone Weather)
- No shared view of forecast data at the decision point
- Weather context is never recorded alongside the operational decision — no audit trail
- Consumer apps do not integrate with the organisation's alerting or notification systems

### 2.2 Strategic Alignment

The Weather App supports the FY26 operations efficiency programme by embedding contextual data (weather) directly into the platforms where operational decisions are made, reducing information-gathering time and improving decision consistency.

### 2.3 Opportunity

| Metric | Current State | Target State |
|---|---|---|
| Avg. time to retrieve weather context | 3–5 min (app switch) | < 30 sec (in-tool) |
| Weather-related rescheduling decisions logged | 0% | 100% |
| Teams using consistent forecast source | ~10% | 100% |

---

## 3. Stakeholders

| Role | Name / Team | Interest |
|---|---|---|
| Sponsor | VP Operations | Operational efficiency |
| Product Owner | Platform Team | Feature delivery |
| Primary Users | Field Ops, Logistics, Facilities | Daily forecast access |
| Secondary Users | Safety Team | Severe weather alerts |
| Compliance | Legal / Risk | Decision audit trail |
| IT | Infra Team | API key management, cost controls |

---

## 4. Business Objectives

1. **BO-01** — Reduce average weather-lookup time from 3–5 minutes to under 30 seconds
2. **BO-02** — Create a consistent, organisation-wide source of truth for forecast data
3. **BO-03** — Enable weather context to be logged alongside operational decisions
4. **BO-04** — Provide proactive severe weather alerts to affected teams within 5 minutes of issuance

---

## 5. Scope

### 5.1 In Scope

- Hyperlocal current conditions (temperature, humidity, wind, precipitation, UV index)
- 7-day daily forecast
- 24-hour hourly forecast
- Saved locations (per user and organisation-wide)
- Severe weather alerts with configurable notification channels
- Historical conditions lookup (past 30 days)
- Embeddable forecast widget for other internal tools

### 5.2 Out of Scope

- Long-range forecasts beyond 14 days
- Custom weather modelling or proprietary data
- Native mobile applications (v1 is web only)
- Integration with external routing or scheduling systems in v1

### 5.3 Assumptions

- A commercial weather data API (OpenWeatherMap or equivalent) will be procured
- All target users have access to the internal GitHub organisation
- Operations teams work primarily from laptop / desktop during decision-making

---

## 6. Business Constraints

| Constraint | Detail |
|---|---|
| Budget | Weather API subscription ≤ $200/month |
| Timeline | MVP in 6 weeks |
| Data retention | Historical data retained for 90 days |
| Compliance | No personally identifiable location data stored without consent |

---

## 7. Success Criteria

- Average weather-lookup time ≤ 30 seconds for 90% of sessions (measured via analytics)
- Severe weather alerts delivered within 5 minutes of NWS/Environment Canada issuance
- 80% of field ops and logistics team members use the app weekly within 45 days
- Zero API cost overruns in first quarter

---

## 8. Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Weather API accuracy gaps in rural areas | Medium | High | Evaluate multiple providers; allow fallback |
| API cost overrun from high usage | Medium | Medium | Implement per-user caching; set cost alerts |
| Alert delivery latency > 5 min | Low | High | Redundant polling; direct webhook from provider |
| Low adoption — teams prefer consumer apps | Medium | High | Executive mandate + superior UX |
