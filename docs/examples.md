# Examples

Two production apps built entirely on the avalonOS SDK. Open them, talk to them,
then read what each one demonstrates.

---

## avalonOS demo — the canonical drop-in

**→ [avalonos.com/demo](https://avalonos.com/demo)**

The simplest possible integration: one container, the SDK script, one factory call.
An avatar greets you and holds a conversation.

**What it shows**

- The minimal drop-in shape — `window.Kinetar('cus_…', { container, character, voice })`.
- Auto-mount on construction and the tap-to-activate audio flow.
- The default `ready` → chat → `speech-end` loop.

This is the integration in [getting-started.md](getting-started.md), running live.

---

## Convincing — interview practice

**→ [convincing.app](https://convincing.app)**

An interview-practice app: the avatar plays an interviewer for a role you choose,
asks questions, listens to your spoken answers, scores them, and produces a
downloadable report.

**What it shows**

- A full, opinionated product built on the same public SDK — strict SDK-consumer
  parity, no privileged access.
- Driving the conversation with `chat()` and reacting to `speech-end` to pace turns.
- Combining the avatar with the app's own logic (scoring, reporting) **outside** the
  SDK — the SDK renders and converses; the product owns everything else.

A good reference for how much of your app stays *yours*: the SDK is the avatar and
the voice; your branching, scoring, persistence, and UI live in your own code.

---

## Want your app listed here?

If you ship something on the SDK we'd love to feature it. Reach out via
[avalonos.com/access](https://avalonos.com/access/).
