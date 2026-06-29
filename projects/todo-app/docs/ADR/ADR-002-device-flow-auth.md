# ADR-002: Use GitHub OAuth Device Flow for Authentication

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-06-28 |
| Deciders | Platform Team |

---

## Context

The Todo App is a static SPA deployed to GitHub Pages. It needs to authenticate users against GitHub OAuth to make authenticated API calls. The standard OAuth Authorization Code Flow requires a server-side redirect endpoint to receive the callback and exchange the code for a token. We have no server.

Options:
1. **Authorization Code Flow** — requires a callback server
2. **OAuth Device Flow** — server-less, public-client flow
3. **Personal Access Tokens (PAT)** — user pastes their own token
4. **GitHub App** — requires server for installation and webhook handling

---

## Decision

Use the **GitHub OAuth Device Flow** (RFC 8628).

The flow:
1. App requests a `device_code` and `user_code` from GitHub
2. User is shown the `user_code` and a verification URL
3. User visits the URL, enters the code, and authorises the app
4. App polls for the access token until the user authorises or the code expires

Only the OAuth App **Client ID** is needed in the browser — no secret is exposed.

---

## Consequences

**Positive:**
- No backend server required — compatible with static hosting on GitHub Pages
- No client secret exposed in the browser
- Works identically for github.com and GitHub Enterprise Cloud
- Users experience is familiar (similar to GitHub CLI, VS Code)

**Negative:**
- UX is slightly more friction than a standard redirect flow — user must visit a separate URL
- Device codes expire after 15 minutes; if the user takes longer, they must restart
- Polling introduces minor network overhead during the auth window

**Mitigated by:**
- Clear UI showing the user code prominently with a "Copy code" button and direct link to the verification URL
- Expiry countdown displayed so users know how much time remains
