# What's coming

Where the SDK is today, and what's landing next. We build in small, shipped
increments — this is the near-term direction, not a dated promise.

---

## Today

What you can rely on right now:

- **Drop-in embed** — one `<script>` tag, managed `cus_…` auth, an avatar that
  listens, talks, lip-syncs, and emotes. See [getting-started.md](getting-started.md).
- **Your avatar, configured at onboarding** — when you request access you tell us
  your avatar's **look** and its **persona** (name, personality, goal, voice). We
  set it on your customer ID; every embed of that ID renders *your* character with
  *your* personality. It resolves server-side, so nothing sensitive ships in your
  page.
- **A declarative stage** — `k.scene.{camera, lighting, background, music}`.
- **Banked playback** — pre-bake performances and replay them with `k.perform()`.

The two things you can't yet do **yourself, without us** are: build/customize the
avatar's look, and edit its persona. Both engines exist and run in production today —
what's coming is putting them directly in your hands.

---

## Coming soon

### 1. Your own character — the Assembly creator

This is the big one. The avatar isn't a fixed cast you pick from — it's **assembled**
from modular parts (face, skin, eyes, hair, clothing, colors), composed live in the
browser onto a fully rigged, performing body.

> **Other avatar creators stop at a static dress-up doll. We build a character that performs.**
> Pick a face → pick hair → pick clothes → and it's alive, ready to talk back.

The assembly pipeline already powers every avatar we ship. What's landing is the
**developer-facing creator**: a UI (and a programmatic recipe) that lets you build
*your* character — not ours — and have it lip-sync, emote, and react on the full
performance stack. Custom looks become yours; no static GLB export, no losing the
performance layer.

**What you'll be able to do:**
- Compose a character from face / skin tone / hair / clothing / color slots.
- Hot-swap individual slots live (change the hair, keep the conversation going).
- Save a recipe and load it by reference from the SDK.

### 2. Self-serve persona authoring

Edit your avatar's **prompt and personality yourself** — name, personality, goal,
and voice — without going through us. The persona override system runs in production
today (it's how we configure your avatar at onboarding); what's coming is the
self-serve door so you author and update it directly.

Persona stays **server-side by design** — your system prompt is never shipped to the
browser, which protects it from inspection and injection. You'll author it through a
developer dashboard / authenticated endpoint, not as a browser parameter.

### 3. Multi-character scenes, generally available

The SDK already carries a multi-character **cast** surface — `k.scene.cast.*`,
per-character routing (`perform(bundle, { as })`), named arrangements and framings.
Today it's exercised mainly by our own story apps. As the creator and persona tools
land, driving a full multi-character scene becomes a first-class developer feature.

---

## How we'll roll it out

Access is invite-only while we onboard the first developers by hand — that's
deliberate, so each integration gets attention. As the self-serve creator and persona
tools ship, onboarding shifts from "tell us and we configure it" to "do it yourself in
a dashboard."

Want early access to any of this, or to shape what we build? Tell us at
[avalonos.com/access](https://avalonos.com/access/) — say which of the above matters
most to you.
