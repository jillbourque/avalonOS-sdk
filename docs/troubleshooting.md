# Troubleshooting

Always start from `err.code`. Subscribe early:

```js
k.on('error', (e, ctx) => console.error(e.code, e.status, e.requestId, ctx));
```

When you contact support, include `err.code`, `err.status`, and `err.requestId` —
the request ID maps directly to our server-side trace.

---

## Integration symptoms

| Symptom | Likely cause | First check |
|---|---|---|
| `window.Kinetar is not a function` | The bundle was served through a dev bundler that rewrote it, or it didn't load. | `typeof window.Kinetar` in the console (`"function"` = good). Confirm the `<script>` loaded the CDN URL, not a re-bundled copy. Serve it as a plain static asset, not through a module dev server. |
| `ORIGIN_REJECTED` | Your page origin isn't allowlisted for this customer ID. | Log `window.location.origin` and send us that exact string to add. |
| Mount never fires `ready` / hangs | Avatar assets 404, backend unreachable, or the 30s mount timeout fired. | Network tab — look for red rows in the first 30s. |
| `401` errors right after mount | Wrong or unrecognized customer ID. | Confirm the `cus_…` string is exactly what we issued (`UNKNOWN_CUSTOMER` = typo). |
| Audio is silent / `Failed to construct 'Worker'` in console | An old SDK build loaded cross-origin. | Make sure you load from `https://sdk.kinetar.app/v0/` (or a pinned build ≥ v0.4.0). |
| Audio plays but no audio at all on first interaction | Browser autoplay policy — audio needs a user gesture. | Ensure the first `say`/`chat` follows a click/tap. The chat overlay's first tap handles this automatically. |
| Avatar speaks but mouth doesn't move | The turn returned no lip-sync data, or a transient sync issue. | Reproduce and report with `requestId`. |
| `speech-start` never fires | Known: not reliably emitted on the streaming path. | Use `ready` to gate speech and `speech-end` to detect completion instead. |
| `localStorage.kinetar_uid` resets between visits | Visitor cleared storage, is in private mode, or browser ITP cleared it after inactivity. | Expected. `kinetar_uid` is per-browser, not a server identity. |
| One unexplained DevTools **Issues**-tab warning | Chrome blocking the best-effort `SameSite=Lax` identity cookie. | Benign by design — `localStorage` is the primary identity channel. No fix. |
| JWT/token shows up somewhere | The token must never be exposed. `getSession()` returns only `{ userId, expiresAt, customerId }`. | If you ever see a token on the instance, report it as a bug. Scrub `Authorization` headers before sending Network logs to a tracker. |

---

## Error-code reference

| `err.code` | Meaning | Retry? | Action |
|---|---|---|---|
| `ORIGIN_REJECTED` | Origin not allowlisted | No | Send us the exact origin. |
| `TOKEN_EXPIRED` | Session token expired | Auto | SDK refreshes + retries; surfaced only if refresh fails. |
| `TOKEN_INVALID` | Token failed validation | Auto once | SDK retries once, then surfaces. |
| `CUSTOMER_REVOKED` | Customer paused/revoked | No | Contact us to reactivate. |
| `UNKNOWN_CUSTOMER` | ID not recognized | No | Check for a typo in the `cus_…` string. |
| `QUOTA_EXCEEDED` | Monthly minutes used up | No | Upgrade / contact us. |
| `RATE_LIMITED` | Too many requests | After `err.retryAfter`s | Back off and retry. |
| `CHARACTER_NOT_FOUND` | Unknown character | No | Check the `character` value. |
| `BAD_REQUEST` | Malformed request | No | Check the text/character you passed. |
| `SERVICE_UNAVAILABLE` | Dependency down | Yes | Transient — retry shortly. |
| `INTERNAL_ERROR` | Unexpected server error | Maybe | Report with `requestId`. |
| `AUDIO_INIT_FAILED` | Audio couldn't start | — | Ensure a user gesture preceded audio; mount continues. |

---

## Still stuck?

Use the contact form at **[avalonos.com/contact](https://avalonos.com/contact/)**. It
has fields for everything we need to resolve it fast:

- The `err.code`, `err.status`, and `err.requestId`.
- Your customer ID and the page origin.
- The SDK URL you're loading (`/v0/` or a pinned version).

Filling those in up front saves a round-trip — `err.requestId` maps straight to our
server-side trace.
