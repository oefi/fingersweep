![altar](https://github.com/user-attachments/assets/5214cc3b-cde6-412a-bdfc-c8d860eab615)

# ALTAR — Who Goes First?

A browser-based first-player selector for board games. Everyone places a finger on the screen; one is chosen.

**[Live demo →](https://oefi.github.io/finger_altar/)**

---

## How it works

1. **Screensaver** — ghost fingerprints drift across the screen. No instructions needed.
2. **Collecting** — each touch registers a colored fingerprint and a sonar ring. A perimeter countdown starts; it resets on every new touch so late players can join.
3. **Locked** — timer hits zero, no more entries.
4. **Selecting** — slot-machine animation cycles through all fingers with exponential slowdown. The winner is determined by `Math.random()` before the animation begins.
5. **Winner** — chosen finger pulses; losers shrink away. Auto-resets after 5.5 s or tap **▲ AGAIN**.

## Features

- Up to 10 simultaneous players
- Synthesized audio (Web Audio API — no sound files)
- Haptic feedback on lock-in and winner reveal (Android)
- Perimeter countdown timer with color-coded urgency (green → amber → red)
- Status messages mirrored to all four screen edges — readable from any seat
- Fullscreen API (Android/Chrome); Add to Home Screen prompt (iOS)
- `prefers-reduced-motion` respected
- PWA manifest included — installable

## Technical

| | |
|---|---|
| Dependencies | None |
| External requests | None (fonts inlined as base64 woff2) |
| Deployment | Drop a single HTML file anywhere |
| Offline | Works from first load |
| Audio | Fully synthesized via Web Audio API |
| State machine | `screensaver → collecting → locked → selecting → winner` |

## Tunable constants

At the top of the `<script>` block:

```js
const TIMER_MS   = 4000;  // ms before lock-in (resets on each new touch)
const WINNER_MS  = 5500;  // ms winner screen displays before auto-reset
const LOCK_PAUSE = 500;   // ms between lock flash and selection animation
const MIN_STEPS  = 24;    // minimum slot-machine steps before final lap
```

## Browser support

Any browser with multi-touch support. Tested on Chrome/Android and Safari/iOS. Mouse fallback available for desktop testing (one simulated finger).

## License

MIT
