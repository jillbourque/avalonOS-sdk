# Getting started

This guide takes you from nothing to a talking avatar on your page.

## 1. Get a customer ID

The SDK is invite-only. Request access at
**[avalonos.com/access](https://avalonos.com/access/)**.

You'll receive:

- A **customer ID** that looks like `cus_yourcompany`. It's non-secret — safe to
  commit, safe to ship in front-end source.
- Your **domain(s) added to the allowlist** (`allowedOrigins`). The SDK only mints
  sessions from origins we've authorized for your customer ID, so a leaked ID can't
  be used from someone else's site.

There is **no API key to manage in the browser.** Auth is handled for you (see
[authentication.md](authentication.md)).

## 2. Drop in the script

Add a full-size container and the SDK script to your page:

```html
<!DOCTYPE html>
<html>
<body style="margin:0;background:#0f0f14;">
  <div id="app" style="width:100vw;height:100dvh;"></div>

  <script src="https://sdk.kinetar.app/v0/kinetar.js"></script>
  <script>
    const k = window.Kinetar('cus_your_customer_id', {
      container: '#app',
      character: 'phoebe-v2',
      voice: 'hd',
    });

    k.on('ready',  (d) => console.log('ready', d));
    k.on('error',  (e) => console.error(e.code, e));
    window.k = k; // handy for poking at it in the console
  </script>
</body>
</html>
```

`container` auto-mounts the avatar on construction — you don't need a separate
`mount()` call. The avatar appears, idles, and activates when the visitor taps the
chat overlay.

> **Browsers require a user gesture before audio can play.** The chat overlay's
> first tap doubles as that gesture, so the canonical flow Just Works. If you mount
> programmatically, make sure your first `say`/`chat` happens after a click/tap.

## 3. Wait for `ready`, then drive it

Gate any speech on the `ready` event — that's the single signal that the avatar is
loaded and drawable:

```js
k.on('ready', () => {
  k.say('Hi! Ask me anything.');     // speak a fixed line
});
```

The two ways to make the avatar talk:

| Method | What it does |
|---|---|
| `k.say(text)` | Speaks `text` through the LLM + TTS pipeline. |
| `k.chat(text)` | Sends a conversational turn — the avatar replies in character, with multi-turn memory within the session. |
| `k.stop()` | Barges in and stops the current line immediately. |

## 4. Listen to what's happening

```js
k.on('ready',        (d) => {});  // { characterIds, assemblyStatus }
k.on('speech-end',   (d) => {});  // a spoken line finished
k.on('usage',        (u) => {});  // a billable turn completed
k.on('error',     (e, ctx) => {}); // dispatch on e.code — see troubleshooting.md
```

Full event list and payloads: [api-reference.md](api-reference.md#events).

## 5. Tear down cleanly

When the avatar leaves the page (SPA route change, "end chat" button):

```js
k.unmount();   // releases the mic, disposes the renderer, ends the live session
```

`unmount()` is safe to call any time — before mount, twice, mid-mount. It preserves
the returning visitor's identity (their conversation resumes on a later re-mount)
while fully ending the *live* session. To force a brand-new visitor, also clear
`localStorage.kinetar_uid`.

## Your avatar's look and persona

The avatar's **appearance** and its **persona** (name, personality, goal, voice) are
tied to your customer ID and set when you onboard — you tell us what you want and we
configure it. Every embed of your `cus_…` then renders *your* character with *your*
personality, resolved server-side (so nothing sensitive ships in your page).

Today that's a configure-at-onboarding step. A **self-serve creator** (build your own
character from modular parts) and **self-serve persona authoring** are on the near-term
roadmap — see [what's coming](roadmap.md).

## Common options

| Option | Default | Notes |
|---|---|---|
| `container` | — | CSS selector or element. Set it to auto-mount on construction. |
| `character` | your configured avatar | Selects among the characters provisioned for your customer ID. Most integrations have one; ask us to add more. |
| `voice` | — | `'hd'` for the high-definition neural voice. |
| `scene` | — | A declarative stage (camera/lighting/background) applied at mount. See [api-reference.md](api-reference.md#the-scene-namespace). |

## Where to go next

- The complete surface → **[api-reference.md](api-reference.md)**
- How auth and visitor identity work → **[authentication.md](authentication.md)**
- See it in production → **[examples.md](examples.md)**
- Something not working → **[troubleshooting.md](troubleshooting.md)**
