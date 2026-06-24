# avalonOS SDK

Embed a real-time, voice-driven 3D avatar on any web page with one `<script>` tag.
The avatar listens, talks back with low-latency neural TTS, lip-syncs, and animates —
all rendered client-side, all driven by your prompts.

> **Access is currently invite-only.** Request a customer ID at
> **[avalonos.com/access](https://avalonos.com/access/)**. You'll receive a `cus_…`
> ID and we'll add your domain to the allowlist — that's all you need to go live.

---

## Quick start

```html
<div id="app" style="width:100vw;height:100dvh;"></div>

<script src="https://sdk.kinetar.app/v0/kinetar.js"></script>
<script>
  const k = window.Kinetar('cus_your_customer_id', {
    container: '#app',
    character: 'phoebe-v2',
    voice: 'hd',
  });

  k.on('ready', (d) => console.log('avatar ready', d));
  k.on('error', (e) => console.error(e.code, e));
</script>
```

That's the whole integration. The avatar mounts on page load, idles, and starts
interacting when the visitor taps the chat overlay. No build step, no npm install,
no API key in your front-end — the customer ID is non-secret and safe in source.

→ Full walkthrough: **[docs/getting-started.md](docs/getting-started.md)**

---

## What you get

- **One global, one factory.** `window.Kinetar(customerId, options)` returns an
  instance with a small, stable surface: `mount` / `chat` / `say` / `stop` / `on` /
  `off` / `unmount`.
- **Managed auth.** The SDK mints and refreshes a short-lived session token behind
  the scenes. You never handle a secret in the browser.
- **A real engine, not a video.** Three.js avatar, streaming Opus audio, viseme
  lip-sync, and a Director that drives posture and gesture from the conversation.
- **A declarative stage.** `k.scene.{camera, lighting, background, music}` and a
  multi-character cast for richer scenes (`k.scene.cast.*`).
- **Banked playback.** Pre-bake performances at build time and replay them with
  `k.perform()` — zero marginal cost, no LLM round-trip.

---

## Documentation

| Doc | What's in it |
|---|---|
| [Getting started](docs/getting-started.md) | Drop-in script, the managed `cus_…` flow, your first mount, talking to the avatar |
| [API reference](docs/api-reference.md) | The full `Kinetar` surface — factory, options, methods, events, the `scene`/`perform`/`cast` namespaces |
| [Authentication](docs/authentication.md) | How `cus_…` managed mode works, the `allowedOrigins` gate, session lifecycle, visitor identity |
| [What's coming](docs/roadmap.md) | The near-term roadmap — your own character via the Assembly creator, self-serve persona authoring, multi-character scenes |
| [Examples](docs/examples.md) | Live apps built on the SDK you can try right now |
| [Troubleshooting](docs/troubleshooting.md) | Error-code catalog and the common integration gotchas |

---

## Examples

Two production apps built entirely on this SDK:

- **[avalonos.com/demo](https://avalonos.com/demo)** — the canonical drop-in: an
  avatar that greets you and chats.
- **[convincing.app](https://convincing.app)** — Convincing, an interview-practice
  app where the avatar interviews you for a role, scores each spoken answer, and
  hands you a report.

See [docs/examples.md](docs/examples.md) for what each one demonstrates.

---

## Current version

The CDN serves the rolling **`/v0/`** channel (latest stable, ~5-minute cache).
Latest release: **v0.4.0**. To pin an exact build, load
`https://sdk.kinetar.app/v<x.y.z>/kinetar.js` instead of `/v0/`.

---

## Support

- **Request access / a customer ID** → **[avalonos.com/access](https://avalonos.com/access/)**
- **Questions, a domain to allowlist, or a bug** → **[avalonos.com/contact](https://avalonos.com/contact/)** —
  a form that captures the details we need (error code, request ID, customer ID,
  origin, SDK version) so we can resolve it fast.

---

avalonOS is a product of **[Avalon Spatial Inc.](https://avalonspatial.com)**
© 2026 Avalon Spatial Inc. All rights reserved.
