# Authentication

The SDK uses **managed authentication**. You ship a non-secret customer ID; the SDK
handles everything else. There is no secret key in your browser code.

## The customer ID (`cus_…`)

When you request access at [avalonos.com/access](https://avalonos.com/access/),
you receive a customer ID like `cus_yourcompany`. It is:

- **Non-secret** — safe to commit to source, safe to ship in a public bundle.
- **Origin-scoped** — it only works from domains we've added to your `allowedOrigins`
  list. A leaked ID can't be used from another site.

```js
const k = window.Kinetar('cus_yourcompany', { container: '#app' });
```

## How a session is established

You never see any of this — it's described so you can reason about behavior and errors:

1. On mount, the SDK asks our backend for a short-lived **session token**, sending
   your customer ID and the visitor's identity.
2. The backend checks that the **page origin is in your `allowedOrigins`**. If not,
   it refuses before issuing anything → you get an `ORIGIN_REJECTED` error.
3. The session token is held internally and attached to every subsequent request.
   It is **never** exposed on the instance, in `getSession()`, or in telemetry.
4. When the token nears expiry (or a request returns `TOKEN_EXPIRED`), the SDK
   **mints a fresh one and retries automatically**. You don't handle refresh.

## The `allowedOrigins` gate

This is the single most important thing to get right at onboarding.

- Every origin where you embed the SDK (`https://yourapp.com`, your staging domain,
  `http://localhost:3000` for dev) must be on your allowlist.
- An origin we haven't seeded returns **`ORIGIN_REJECTED` (403)** from the session
  call — the avatar never mounts.
- To add an origin, send us the **exact** origin string (scheme + host + port). There
  is no self-service editor yet; we add it for you.

```js
k.on('error', (e) => {
  if (e.code === 'ORIGIN_REJECTED') {
    // This page's origin isn't allowlisted. Send us window.location.origin.
    console.error('Origin not allowlisted:', window.location.origin);
  }
});
```

## Visitor identity

Each visitor gets a stable ID stored in their browser as `localStorage.kinetar_uid`.

- A returning visitor with the same `kinetar_uid` **resumes their conversation
  history**.
- Clearing `localStorage` (or private-browsing) makes them a **new visitor**.
- `kinetar_uid` is per-browser, not a cross-device server identity. It survives
  `unmount()` so an "end chat → start again" flow resumes the same visitor.

You can read the current visitor's ID for your own analytics:

```js
const { userId, expiresAt, customerId } = k.getSession();
// userId === localStorage.kinetar_uid — log it, correlate it. No token is exposed.
```

### A benign cookie warning

Cross-site SDK consumers see **one** browser-emitted warning in the DevTools
**Issues** tab — Chrome declining to set a `SameSite=Lax` identity cookie. This is
expected and harmless: `localStorage` is the primary identity channel and the cookie
is only best-effort redundancy. No action needed; don't change `SameSite` settings.

## Ending a session vs. forgetting a visitor

| Goal | Call |
|---|---|
| Stop the current spoken line | `k.stop()` |
| End the live session (release mic, dispose), keep the visitor's history | `k.unmount()` |
| Treat the visitor as brand-new next time | `k.unmount()` **and** `localStorage.removeItem('kinetar_uid')` |

## Server-to-server / non-browser callers

The browser path above is the recommended one for all web embeds. If you need a
non-browser caller (CI, build-time banking), that uses a secret API key over a
server-side endpoint — **never** put that key in browser code. Ask us if you have a
server-side use case.
