# Fingersweep v1.0.0

A first-player selector for board games. Players touch the screen, a timer drains, and one finger is chosen.

Open `index.html` in any browser — no server, no install, works offline.

![Demo](demo.gif)

## How It Works

1. **Screensaver** — Ghost fingerprints drift across the screen, inviting players to touch. This is a UI affordance, not decoration. Players don't need to read instructions.

2. **Collecting** — Every finger that touches gets a fingerprint with a color (from a palette of 10 distinct hues) and a sonar ring animation. A perimeter timer counts down from 4 seconds. Every new touch resets the timer so late joiners aren't locked out.

3. **Locked** — When the timer hits zero, the state machine locks. A lock sound plays, haptic feedback fires, and the selection animation begins.

4. **Selecting** — Slot-machine style cycling through all fingers, with exponential slowdown. The winner was already chosen by `Math.random()` before the animation started. The visual is theater, not real-time RNG.

5. **Winner** — The winning finger throbs, losers shrink to 8% opacity, and the overlay shows who won. Auto-resets after 5.5 seconds.

## Quick Start

Open `index.html` in any browser. That's it — no server, no install, works offline.

## Configuration

Want to tweak it? Open `index.html` in a text editor. At the top of the `<script>` block you'll find the `CONFIGURATION` section:

- **TIMER_MS** — milliseconds before lock-in (default 4000 = 4 seconds)
- **WINNER_MS** — how long winner screen displays (default 5500 = 5.5 seconds)
- **LOCK_PAUSE** — delay before selection animation (default 500ms)
- **MIN_STEPS** — slot-machine spins before final lap (default 24)
- **MAX_PLAYERS** — max simultaneous touches (default 10)
- **PLAYER_COLORS** — 10 distinct colors for players

In the `<style>` block, the `:root` section lets you adjust:
- `--accent` — main brand color (default phosphor green)
- `--bg` — background color
- `--font-t` / `--font-m` — title and UI fonts
- `--fp-w` / `--fp-h` — fingerprint dimensions
- `--ring-sz` — sonar ring size

All values have inline comments explaining what they do.

## State Machine

```
screensaver → collecting → locked → selecting → winner
                   ↑            ↓
                   └────────────┘ (reset)
```

## Touch Handling

- Uses `{passive:false}` to prevent browser scroll/zoom during touches
- Touch IDs tracked in a `Map<identifier, {el, color, num}>`
- Mouse fallback provided for desktop testing (`mDown`, tracked under `'mouse'` key)
- Hard cap at 10 players — matches the maximum simultaneous touch points on iPad/Android tablets
- Haptic feedback: `navigator.vibrate([30, 40, 30])` on new touches

## Timer

SVG `<rect>` around the viewport edge with `stroke-dashoffset` driving the countdown. Color shifts: green → amber at 50% → red flashing at 25%. Recalculated on every resize/orientation change.

```js
perimLen = 2 * ((W - 10) + (H - 10));
offset = perimLen * (1 - progress);
```

## Audio

Web Audio API — no external files. Every sound is synthesized:

- **touch**: unique pitch per player (660/780/520/880/600/720/560/820 Hz)
- **release**: 400 Hz sine, quick decay
- **ghost**: 360-640 Hz sine with random pitch, plays when ghost fades in
- **tick**: rising pitch during selection (170 + (step/total) * 330 Hz)
- **lock**: 72 Hz + 144 Hz chord — low mechanical thud
- **winner**: [220, 277, 330, 440, 554] Hz ascending arpeggio

All notes ramp frequency down to 36% of starting value over their duration, giving a descending-sonar quality.

## Ghosts

Screensaver decoration but also functional — they teach players what to do. Each `Ghost` instance is a lifecycle state machine:

```
in → hold → out → dead
```

- Fade in over 650-1150ms with sonar ring pop
- Breathe during hold (2400-5600ms) using sine wave on opacity
- Fade out over 580-980ms
- Always keep 3+ alive. Spawns 4-6 on screensaver entry.

Respects `prefers-reduced-motion` — ghosts still appear but at fixed opacity, no animations.

## Selection Algorithm

```js
const winIdx = Math.floor(Math.random() * n); // chosen before animation
const rand = MIN_STEPS + n * 3; // random-phase steps
const lap = [0,1,...,n-1] rotated until winner is last
seq = [...randomSteps, ...finalLap]
```

Timing: `38 + (progress^2.5) * 780` ms per step. The 2.5 exponent creates steep exponential slowdown — chaotic early, deliberate near the end.

## Device Detection

```js
const shorter = Math.min(screen.width, screen.height);
shorter < 600 ? 'phone' : 'tablet'
```

Set on `<html>` once at load, never re-detected on orientation change. Orientation media queries then fine-tune sizes on top.

## PWA

Inlined manifest as data URI. Enables "Add to Home Screen" on Android/iOS with fullscreen mode. Safari needs its own `apple-touch-icon` link since it ignores manifest icons.

```html
<link rel="manifest" href="data:application/manifest+json,{...}">
```

## CSS Architecture

Position:fixed/absolute stacking by z-index. No flexbox grids. No scrolling — full-screen touch canvas.

Z-index layers:
- 5: ghost fingerprints
- 10: logo
- 11: edge hints
- 20: live fingerprints
- 30: winner fingerprint (elevated)
- 50: touch layer (invisible, captures events)
- 60: timer SVG
- 65: lock flash
- 70: player count + status pills
- 75: color flash
- 80: winner overlay
- 200: control buttons
- 250: sound toast
- 300: iOS hint
- 9999: scanlines

## Reduced Motion

```css
@media (prefers-reduced-motion: reduce) {
  #radar-sweep { animation: none; opacity: .07; }
  .edge-hint { animation: none; opacity: .3; }
  .fp-live.active .fp-ring { animation: none; }
}
```

Functional UI stays visible, decorative animations killed. Ghosts still render (they're UI, not decoration).

## Edge Cases Handled

- **Visibility change**: clears fingers on background, re-anchors timer on return
- **Pointer cancel**: belt-and-suspenders for touch stream interruption
- **Window blur**: clears mouse finger on desktop focus loss
- **Solo player**: instant winner, shows "BOLD MOVE" instead of "CHOSEN"
- **Winner collision**: if winning finger is in top half, flip overlay to bottom half
- **Orientation**: debounced timer resize 220ms after rotation event

## Files

- `index.html` — single file, zero dependencies, works offline after load

---

**License: MIT** — Free to use, modify, distribute.