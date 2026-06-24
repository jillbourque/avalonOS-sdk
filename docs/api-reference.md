# API reference

The SDK exposes exactly one global: `window.Kinetar`, a **factory function**.

```js
const k = window.Kinetar(customerId, options);
```

- Call it **without `new`** — it returns a ready instance.
- `customerId` is your `cus_…` string from [avalonos.com/access](https://avalonos.com/access/).
- It's the only global. Detect an instance by duck-typing (`typeof k.on === 'function'`), not `instanceof`.

---

## Constructor options

| Option | Type | Default | Description |
|---|---|---|---|
| `container` | `string \| Element` | — | CSS selector or DOM element. When set, the avatar **auto-mounts on construction** — no separate `mount()` needed. |
| `character` | `string` | your configured avatar | Selects among the characters provisioned for your customer ID. Look + persona are configured at onboarding (self-serve creator coming — see [roadmap](roadmap.md)). |
| `voice` | `string` | — | `'hd'` selects the high-definition neural voice. |
| `scene` | `object` | — | A declarative stage applied at mount — see [the scene namespace](#the-scene-namespace). |
| `workerBaseUrl` | `string` | auto | Override the audio-decoder worker path. Only needed under strict CSP where you self-host the worker files. Auto-derived from the script URL otherwise. |

---

## Methods

### Lifecycle

| Method | Returns | Description |
|---|---|---|
| `mount(containerOrSelector)` | `Promise` | Mounts the avatar into the container. Resolves when `ready` fires (30s timeout). Skip this if you passed `container` to the factory. |
| `unmount()` | — | Tears down the live session: releases the mic, disposes the renderer, removes SDK DOM, ends the session token. Null-safe at any time. Preserves visitor identity for a later re-mount. |

A given instance mounts **once**. After `unmount()`, construct a new instance to mount again.

### Conversation

| Method | Returns | Description |
|---|---|---|
| `say(text)` | `Promise` | Speak a line through the LLM + TTS pipeline. |
| `chat(text, opts?)` | `Promise` | Send a conversational turn; the avatar replies in character with within-session memory. `opts.characterId` addresses a specific cast member. |
| `stop()` | — | Barge in — stop the current spoken line immediately. The mic and session stay live. |

### Events

| Method | Description |
|---|---|
| `on(event, handler)` | Subscribe. |
| `off(event, handler)` | Unsubscribe. |

### Introspection

| Method | Returns | Description |
|---|---|---|
| `getCharacters()` | `string[]` | The character IDs currently loaded. |
| `getUsage()` | `object` | Session totals: `{ totalSpeechSeconds, interactionCount, estimatedCost, voiceQuality }`. `estimatedCost` is a display estimate; authoritative billing is server-side. |
| `getSession()` | `object \| null` | `{ userId, expiresAt, customerId }`. Never exposes the session token. Returns `null` after `unmount()`. |

---

## Events

Subscribe with `k.on(name, handler)`.

| Event | Payload | Fires when |
|---|---|---|
| `ready` | `{ characterIds, assemblyStatus }` | Avatar mounted, loaded, and drawable. Gate `say`/`chat` on this. |
| `speech-end` | `{ characterId }` | A spoken line (and its lip-sync) finished. |
| `usage` | `{ speechDurationMs, cumulativeSeconds, estimatedCost, voiceQuality, source }` | After each billable turn. `source` is `'visemes-ended'` (natural end) or `'barge-in'` (interrupted). |
| `error` | `(err, ctx?)` | Any auth, network, mount, or audio error. **Dispatch on `err.code`**, never on the message string. |
| `perform:start` / `perform:end` / `perform:error` | `{ bundleId? }` / `(err)` | Banked playback boundaries (see [perform](#banked-playback-perform)). |
| `perform:pause` / `perform:resume` | `{}` | Mid-line pause/resume. |
| `action:start` / `action:end` | `{}` | A cast-member animation (`cast.action`) started/ended. |

> **Note on `speech-start`:** a `speech-start` event exists but is not reliably
> emitted on the streaming (production) audio path today. Use `ready` to know the
> avatar can speak and `speech-end` to know a line finished. Don't build critical
> logic on `speech-start`.

---

## Error codes

Customer code should `switch (err.code)` — never match `err.message`. Every error
carries `.code`, `.status`, and (for server errors) `.requestId`.

| `err.code` | HTTP | What to do |
|---|---|---|
| `ORIGIN_REJECTED` | 403 | Your page's origin isn't on the allowlist for this customer ID. Send us the exact origin to add it. The most common onboarding error. |
| `TOKEN_EXPIRED` | 401 | Session token aged out. The SDK refreshes and retries automatically — you only see this if the refresh itself fails. |
| `TOKEN_INVALID` | 401 | Token failed validation. SDK retries once, then surfaces this. |
| `CUSTOMER_REVOKED` | 403/423 | The customer ID is paused or revoked. Contact us to reactivate. |
| `UNKNOWN_CUSTOMER` | 404 | The `cus_…` ID isn't recognized — usually a typo. |
| `QUOTA_EXCEEDED` | 429 | Monthly speech minutes exhausted. Show an upgrade path. |
| `RATE_LIMITED` | 429 | Too many requests too fast. `err.retryAfter` is the back-off in seconds. |
| `CHARACTER_NOT_FOUND` | 404 | Unknown `character` value. |
| `BAD_REQUEST` | 400 | Malformed request — missing text/character. |
| `SERVICE_UNAVAILABLE` | 503 | A backend dependency is down. Transient. |
| `INTERNAL_ERROR` | 500 | Unexpected server error. Include `err.requestId` when reporting. |
| `AUDIO_INIT_FAILED` | — | Audio couldn't start — usually the SDK ran before a user gesture. |

More on each in [troubleshooting.md](troubleshooting.md).

---

## The `scene` namespace

`k.scene.*` declaratively controls the stage. Each verb can be called at any time —
calls supersede whatever the stage is currently doing.

| Call | Description |
|---|---|
| `k.scene.camera.to / orbit / dolly / frame / reset` | Move or frame the camera. `frame({ targets:[ids] })` frames specific cast members. |
| `k.scene.lighting(preset)` | `'studio'` / `'warm'` / `'moody'` / `'candlelit'`. |
| `k.scene.background(spec)` | `'gradient'` / `'image'` / `'hdr'`. |
| `k.scene.music(spec)` | Ambient music that ducks under speech. |
| `k.scene.set(scene)` | Apply a whole stage schema (camera + lighting + background + cast) in one call. |

### Multiple characters — the cast

> **Availability:** the cast surface ships in the bundle and powers our own
> multi-character apps today, but most integrations are provisioned with a single
> character. Multi-character scenes become a first-class developer feature as the
> Assembly creator lands — see [what's coming](roadmap.md). Talk to us if you want
> more than one character on your customer ID now.

| Call | Description |
|---|---|
| `k.scene.cast.add / remove / focus / show / hide / list` | Manage a resident cast — preload bodies once, toggle visibility. |
| `k.scene.cast.action(id, name, opts?)` | Play a named animation on one character (rides on top of automatic staging). |
| `k.scene.cast.express(id, key)` | Play a named expression (face + gesture + mocap together) on one character. |
| `k.scene.cast.arrange(name, opts?)` | Animate the cast into a named arrangement (`row` / `arc` / `face_off` / …). |
| `k.perform(bundle, { as: id })` | Route a banked performance to a specific cast member; focus follows the speaker. |

---

## Banked playback (`perform`)

Pre-bake a performance at build time and replay it with no LLM round-trip and no
marginal cost. (Banking happens server-side at build time; the browser only plays.)

| Method | Description |
|---|---|
| `k.perform(bundle, opts?)` | Play a banked bundle. Fires `perform:start` → `perform:end`. |
| `k.stopPerform()` | Interrupt the current performance (no resume). |
| `k.pausePerform()` / `k.resumePerform()` | Hold and continue mid-line (survives screen-lock / backgrounding). |

Sequence whole stories by chaining off the `perform:end` event.

---

## Versioning

- `https://sdk.kinetar.app/v0/kinetar.js` — rolling latest stable (~5-min cache).
- `https://sdk.kinetar.app/v<x.y.z>/kinetar.js` — an immutable pinned build.

Pin a version in production if you want byte-for-byte stability; use `/v0/` to
always get the latest fixes. Current release: **v0.4.0**.
