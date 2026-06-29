# User Requirements Specification — Weather App

| Field | Value |
|---|---|
| Document ID | WTHR-URS-001 |
| Version | 1.0 |
| Status | Draft |
| Author | Platform Team |
| References | WTHR-BRD-001 |

---

## 1. User Personas

### 1.1 Field Operations Lead Jordan

- **Role:** Field Operations Lead — manages outdoor crew scheduling
- **Goals:** Know tomorrow's forecast before 7am to set the crew schedule; receive severe weather warnings immediately
- **Frustrations:** Switching between consumer weather apps and operational tools; no record of what forecast was seen when making a scheduling call
- **Tech comfort:** Medium — uses apps fluently, not developer

### 1.2 Logistics Coordinator Sam

- **Role:** Logistics Coordinator — routes deliveries and vehicle dispatch
- **Goals:** Hourly precipitation forecast for the next 12 hours; road condition context
- **Frustrations:** Inaccurate forecasts from free consumer apps; no hourly granularity by default
- **Tech comfort:** Medium

### 1.3 Facilities Manager Robin

- **Role:** Facilities Manager — oversees multiple building sites
- **Goals:** Monitor conditions at several fixed locations simultaneously; track heating/cooling load impact
- **Frustrations:** Checking conditions at each site individually; no comparative view
- **Tech comfort:** Low-Medium — prefers large clear displays

---

## 2. Functional Requirements

### 2.1 Location Management

| ID | Requirement | Priority |
|---|---|---|
| FR-LOC-01 | Users must be able to search for a location by city name, postcode, or coordinates | Must |
| FR-LOC-02 | Users must be able to save locations to a personal favourites list | Must |
| FR-LOC-03 | Users must be able to set a default location that loads on app open | Must |
| FR-LOC-04 | Administrators must be able to create organisation-wide pinned locations | Should |
| FR-LOC-05 | Users must be able to detect and use their current browser location | Should |

### 2.2 Current Conditions

| ID | Requirement | Priority |
|---|---|---|
| FR-CURR-01 | Display current temperature in °C or °F (user preference) | Must |
| FR-CURR-02 | Display "feels like" temperature | Must |
| FR-CURR-03 | Display wind speed and direction | Must |
| FR-CURR-04 | Display humidity percentage | Must |
| FR-CURR-05 | Display precipitation type and intensity (if active) | Must |
| FR-CURR-06 | Display UV index | Should |
| FR-CURR-07 | Display visibility in km/miles | Should |
| FR-CURR-08 | Display conditions icon/description (e.g. "Partly Cloudy") | Must |

### 2.3 Forecasts

| ID | Requirement | Priority |
|---|---|---|
| FR-FORE-01 | Display a 24-hour hourly forecast | Must |
| FR-FORE-02 | Display a 7-day daily forecast | Must |
| FR-FORE-03 | Each daily entry must show high/low temps and precipitation probability | Must |
| FR-FORE-04 | Each hourly entry must show temperature, precipitation probability, and wind | Must |
| FR-FORE-05 | Display a precipitation chart for the next 24 hours | Should |

### 2.4 Alerts

| ID | Requirement | Priority |
|---|---|---|
| FR-ALERT-01 | Display active severe weather alerts for the current location | Must |
| FR-ALERT-02 | Alerts must include type, severity, headline, and expiry time | Must |
| FR-ALERT-03 | Users must be able to opt in to browser push notifications for severe alerts | Should |
| FR-ALERT-04 | Administrators must be able to configure Slack webhook destinations for org-wide alerts | Should |

### 2.5 Historical Conditions

| ID | Requirement | Priority |
|---|---|---|
| FR-HIST-01 | Users must be able to view historical daily conditions for the past 30 days | Should |
| FR-HIST-02 | Historical view must show high/low, precipitation total, and wind peak | Should |

### 2.6 Widget / Embed

| ID | Requirement | Priority |
|---|---|---|
| FR-WIDGET-01 | A minimal embed widget must be available for inclusion in other internal pages | Could |
| FR-WIDGET-02 | The widget must accept a location parameter and auto-refresh every 15 minutes | Could |

---

## 3. Non-Functional Requirements

### 3.1 Performance

| ID | Requirement |
|---|---|
| NFR-PERF-01 | Current conditions must load within 1.5 seconds on standard broadband |
| NFR-PERF-02 | Forecast data must be cached client-side and not re-fetched within a 10-minute window |
| NFR-PERF-03 | Severe alert check must occur at most every 5 minutes in the background |

### 3.2 Usability

| ID | Requirement |
|---|---|
| NFR-USE-01 | Temperature unit toggle (°C / °F) must persist across sessions |
| NFR-USE-02 | The dashboard must be usable at 1024×768 resolution |
| NFR-USE-03 | WCAG 2.1 AA compliance required |
| NFR-USE-04 | Colour-coded severity levels must also use icons or text labels (not colour alone) |

### 3.3 Reliability

| ID | Requirement |
|---|---|
| NFR-REL-01 | Cached forecast must be displayed if the API is unreachable, with a staleness warning |
| NFR-REL-02 | App must not crash if location permission is denied |

---

## 4. Use Cases

### UC-01: Check Morning Forecast

**Actor:** Jordan (Field Ops Lead)
**Precondition:** Jordan has saved their field site as a favourite location
**Main Flow:**
1. Jordan opens the Weather App
2. Default location loads automatically
3. Jordan sees current temperature, conditions, and the 7-day forecast
4. Jordan decides to reschedule outdoor crew work based on forecasted rain

### UC-02: Receive Severe Weather Alert

**Actor:** Sam (Logistics Coordinator)
**Precondition:** Sam has enabled browser push notifications
**Main Flow:**
1. NWS issues a severe thunderstorm warning for Sam's delivery area
2. App detects the alert within 5 minutes via background poll
3. Sam receives a push notification with the alert headline and expiry
4. Sam opens the app, reads the full alert, and reroutes afternoon deliveries

### UC-03: Compare Multiple Sites

**Actor:** Robin (Facilities Manager)
**Precondition:** Robin has saved 4 building locations to favourites
**Main Flow:**
1. Robin opens the app
2. Robin navigates to the "Multi-site" view
3. App displays current conditions side-by-side for all 4 saved locations
4. Robin identifies which building needs heating adjustment based on the coldest site
