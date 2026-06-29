# ADR-001: Protect Weather API Key via Cloudflare Worker Proxy

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-06-28 |
| Deciders | Platform Team |

---

## Context

The Weather App is a static SPA hosted on GitHub Pages. It needs to call the OpenWeatherMap API, which requires an API key. Embedding the API key in the client-side JavaScript bundle would expose it to any user who opens DevTools, allowing them (or bots) to consume our quota and generate unbounded charges.

Options:
1. **Embed key in bundle** — exposes key, risk of quota abuse
2. **Cloudflare Worker proxy** — free tier, edge-cached, keeps key server-side
3. **AWS Lambda / GCP Cloud Function** — works but adds infrastructure cost and setup complexity
4. **Backend for Frontend (BFF) on Render/Railway** — introduces a persistent server to operate

---

## Decision

Use a **Cloudflare Worker** as a stateless API proxy.

The worker:
- Accepts requests from the SPA's `Origin` (allowlist enforced)
- Appends the OWM `appid` from a Cloudflare Worker Secret
- Returns the response with `Cache-Control: public, max-age=600`
- Leverages Cloudflare's edge cache — identical requests within 10 minutes are served from cache at ~1ms with zero cost

---

## Consequences

**Positive:**
- API key never appears in browser network traffic or JavaScript bundle
- Cloudflare Workers free tier covers 100,000 requests/day — sufficient for our user base
- Edge caching dramatically reduces OWM API call count and cost
- No persistent server to operate or scale

**Negative:**
- Introduces a dependency on Cloudflare (though easy to migrate to another Worker/Lambda platform)
- The proxy must be deployed and managed separately from the GitHub Pages SPA
- `ALLOWED_ORIGIN` must be updated if the Pages URL changes (e.g. custom domain)

**Alternatives rejected:**
- AWS Lambda: requires AWS account setup, IAM roles, API Gateway — over-engineered for a simple proxy
- Embedded key: unacceptable security risk
