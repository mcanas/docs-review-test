# ADR-003: Use Service Worker for Background Alert Polling

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-06-28 |
| Deciders | Platform Team |

---

## Context

The Weather App must deliver severe weather alerts within 5 minutes of issuance, even when the user is not actively looking at the app tab. Standard SPA polling (`setInterval`) only runs while the tab is visible and active — on mobile, background tabs are frequently suspended by the OS.

Options:
1. **Main-thread `setInterval`** — stops when tab hidden/backgrounded
2. **Service Worker with periodic background sync** — runs independently of tab
3. **Server-sent events / WebSocket from a backend** — requires persistent server
4. **Web Push from the weather API provider** — OWM does not offer push webhooks

---

## Decision

Use a **Service Worker** registered on first app load. The worker polls the NWS GeoJSON alert feed every 5 minutes using `setInterval` within the worker's own execution context. On detecting a new alert, it calls `self.registration.showNotification()`.

Alert state (known alert IDs) is persisted to IndexedDB within the Service Worker so that already-seen alerts are not re-notified after the worker restarts.

---

## Consequences

**Positive:**
- Alert polling continues on mobile even when the tab is backgrounded or the screen is locked
- No backend infrastructure required — the NWS alert feed is a public, unauthenticated endpoint
- Push notifications integrate naturally with OS notification centres (Android, macOS, Windows)

**Negative:**
- Service Workers require HTTPS — enforced by GitHub Pages, so no issue in production; developers must use `localhost` (allowed by browsers) or a self-signed cert
- User must grant notification permission — if denied, alerts degrade to in-app banner only
- Service Worker lifecycle (install, activate, wait) adds complexity during development
- IndexedDB in service workers is asynchronous and requires careful error handling

**Fallback:** If the Service Worker is unavailable (browser doesn't support it, or permission denied), the SPA falls back to a visible in-app alert banner that polls on page focus using `visibilitychange`.
