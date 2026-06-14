# CLAUDE.md — MacBook Pro Keyboard

## Project overview

An interactive MacBook Pro keyboard simulator (`keyboard.html`). Single HTML file with CSS and JS inline. No build step, no package manager — open directly in a browser as `file://` or serve locally. External CDN dependencies: Tone.js (audio) and Google Inter font.

## Running the app

Open `keyboard.html` directly in a browser, or serve via the registered local server:

```
python3 -m http.server 8790
# then visit http://localhost:8790/keyboard.html
```

A `launch.json` in the Confetti project at `/Users/sebastien.lim/Vibe coding/Confetti project/.claude/launch.json` registers port 8790 for the preview tools.

---

## Architecture

Everything lives in `keyboard.html` — CSS, HTML, and JS in a single file.

**External CDN dependencies (loaded via `<head>`):**
- `https://fonts.googleapis.com/css2?family=Inter:wght@400;500` — Inter font for controls UI
- `https://cdn.jsdelivr.net/npm/tone@15.0.4/build/Tone.min.js` — Tone.js for key audio synthesis

### Scale system

All dimensions are multiplied by `--u: 1.8` (the global scale factor). Every `px` value in the CSS reads `calc(Npx * var(--u))`. Changing `--u` rescales the entire keyboard proportionally.

```css
:root {
  --u: 1.8;
  --gap: calc(4px * var(--u));   /* gap between keys */
  --kr:  calc(5px * var(--u));   /* key border-radius */
  --plate-a: 1;                  /* plate opacity (0 or 1, toggled by controls) */
}
```

### CSS custom properties for theming

All key appearance vars are defined in `:root` so JS can override them on `#keyboard-shell` and they cascade to every `.key` child:

| Variable | Default (Dark mode) | Light mode |
|---|---|---|
| `--key-bg` | `linear-gradient(175deg, #2A2A2D…)` | `rgba(255,255,255,0.10)` |
| `--key-color` | `#CECECE` | `rgba(255,255,255,0.92)` |
| `--key-sub-color` | `#888890` | `rgba(255,255,255,0.50)` |
| `--key-shadow` | multi-layer inset + drop shadow | `none` |

**Critical:** these vars must stay in `:root`, not inside `.key {}`. If defined on the element itself, inline style overrides from JS won't cascade correctly.

### Keyboard shell & plate

`.keyboard-shell` is a transparent container. The silver aluminium plate is rendered entirely in `::before` (background + box-shadow). The recessed bezel is in `::after`. Both fade via `opacity: var(--plate-a)`.

```css
.keyboard-shell::before {
  background: linear-gradient(170deg, #D6D6DC … #B8B8BF);
  box-shadow: …;
  opacity: var(--plate-a, 1);
  transition: opacity 0.08s linear;
}
```

**White mode plate override** (`.white-mode` class on `#keyboard-shell`):
```css
.white-mode.keyboard-shell::before {
  background: rgba(0, 0, 0, 0.10);  /* dark 10% tint instead of aluminium */
  box-shadow: none;
}
.white-mode.keyboard-shell::after { box-shadow: none; }
```

### Key layout data

Three layout arrays define the keyboard; JS builds DOM from them:

- `FN_ROW` — 14 keys (esc, F1–F12, power ⏻), all `fn-key` height (`28px * --u`)
- `MAIN_ROWS` — 4 rows (number row, QWERTY, ASDF, ZXCV)
- `BOTTOM_ROW` — fn, control, option, command, space, command, option

Each key object: `{ l: label, s: sub-label, c: event.code, f: flex, id?, mod?, modAlign? }`

Arrow keys (←, ↑, ↓, →) are appended manually in `buildKeyboard()`. ↑/↓ are wrapped in `.arrow-ud` which has `height: calc(42px * var(--u))` and `gap: var(--gap)`, with each half-key at `flex: 1` — this is what makes them fill exactly the same height as ← and →. **Do not remove the explicit height from `.arrow-ud` or the half-keys will render shorter.**

### Modifier key alignment

Bottom-row modifier keys (control, option, command) have directional content alignment:

- **Left side** (control, option, command before spacebar): `modAlign: 'right'` → class `key-modifier-right` → `align-items: flex-end; justify-content: space-between` — symbol top-right, label bottom-right
- **Right side** (command, option after spacebar): `modAlign: 'left'` → class `key-modifier-left` → `align-items: flex-start; justify-content: space-between` — symbol top-left, label bottom-left

Modifier key labels:
- `.key-sub` (the symbol: ^, ⌥, ⌘): 13px, `var(--key-color)`, `font-weight: 300`
- `.key-main` (the word: control, option, command): 7.5px, `var(--key-sub-color)`

### Key press animation

**Press state** (`.key.active`):
- `transform: scale(0.90)` anchored to `transform-origin: center center`
- `filter: brightness(0.60) !important`
- Reduced box-shadow

**White mode press** (`.white-mode .key.active`) — overrides the above:
- Key goes to `rgba(255,255,255,1.0)` (solid white)
- `--key-color: #1A1A1A; --key-sub-color: #555555` (dark text)
- `filter: brightness(1) !important; box-shadow: none !important`

**White mode hover** (`.white-mode .key:hover`):
- `background: rgba(255,255,255,0.15)` (up from base 0.10)
- `filter: none` (suppresses the dark-mode brightness boost)

**Touch ripple** (spawned by `spawnTouchRipple(el)`):
- A `<span class="key-ripple">` is appended to the key on press
- Radial gradient: transparent centre → white ring edge
- Expands from `scale(0)` → `scale(1)` in 0.55s, then self-removes via `animationend`
- `overflow: hidden` on `.key` clips it to the key boundary

### Ripple wave engine

`fireRipple(srcEl)` spreads a wave outward from the pressed key to all others within `MAX_DIST` pixels:

| JS variable | Default | Control |
|---|---|---|
| `SPEED` | `1.1` ms/px | Wave Speed slider |
| `MAX_DIST` | `750` px | Wave Reach slider |
| `SWELL_MAX` | `0.045` (104.5%) | Key Max Size slider |
| `SWELL_MIN` | `0.0` (100%) | Key Min Size slider |
| `GLOW_OPACITY` | `0.08` (8%) | Glow Opacity slider |

Per-key CSS vars set on each element: `--r-delay`, `--r-dur`, `--r-max`, `--r-min`, `--r-bright`, `--r-flash`.

**White overlay flash** (`.key-wave-flash`): A `<span>` with `background: white` and `opacity` animation is also spawned alongside the ripple-wave class. This makes the wave visible in Light mode where `filter: brightness()` has no effect on translucent white keys. Flash opacity scales with proximity: `--r-flash = GLOW_OPACITY * t` (t = 1 at origin, 0 at max distance).

**Glow colour** is set via `#glow-color-pick` (a `<input type="color">`) and applied as the flash span's background. **Shimmer** mode (`shimmerOn` boolean) adds a holographic gradient overlay on the wave.

### Sound system (Tone.js)

Each key plays a unique musical note on press, synthesised in real time via Tone.js.

**Initialisation** — browsers require a user gesture before the Audio Context can start. `_ensureAudio()` is registered as a one-shot `pointerdown` / `keydown` listener; it calls `Tone.start()` then creates the synth.

**Synth** — `Tone.PolySynth(Tone.Synth)` with:
```js
oscillator: { type: 'triangle' },
envelope: { attack: 0.004, decay: 0.28, sustain: 0.04, release: 1.4 },
volume: -10
```

**Note assignment** — runs once after `buildKeyboard()`. All 78 key elements in `allKeyEls` are spread linearly across C major in 4 octaves (C3–B6), stored in `el.dataset.note`. Low keys (escape, fn row) get lower notes; high keys (bottom row) get higher notes.

**Playback** — `_playNote(el)` is called from `pressKey(el)`. It reads `el.dataset.note` and calls `synth.triggerAttackRelease(note, '8n')`.

### Theming — Light mode

When the "Light" key colour button is clicked:

1. CSS vars set on `#keyboard-shell`:
   - `--key-bg: rgba(255,255,255,0.10)`
   - `--key-color: rgba(255,255,255,0.92)`
   - `--key-sub-color: rgba(255,255,255,0.50)`
   - `--key-shadow: none`
2. `shell.classList.add('white-mode')` — triggers plate and active-key overrides

Reverting to Dark: `removeProperty()` each var, then `classList.remove('white-mode')`. The switch is instant (no opacity fade animation).

### Controls panel

A frosted-glass bar (`background: rgba(16,16,20,0.72); backdrop-filter: blur(16px)`) below the keyboard. Hidden by default (`opacity: 0; transform: scale(0.97); pointer-events: none`); revealed on `#controls-zone:hover` with a scale-in transition.

**Row 1:** Wave Speed · Wave Reach · Key Min Size · Key Max Size

**Row 2:** Glow Colour (colour picker + Shimmer button) · Glow Opacity · Key Colour (Dark/Light pills) · Keyboard (plate toggle) · Background (circle swatches)

- Sliders: `ctrl-group` with `ctrl-header` (label + live value) + `<input type=range>`. Track fill is updated by `updateTrackFill(sl)` which sets a linear-gradient background on the slider element.
- Glow Colour: `<input type="color" id="glow-color-pick">` for custom wave colour; Shimmer button toggles `shimmerOn`.
- Key Colour: pill buttons (`ctrl-opt`) toggling dark/light — switch is instant, no animation.
- Keyboard Plate: CSS toggle switch (`ctrl-toggle` → `ctrl-toggle-track` → `ctrl-toggle-thumb`); checkbox `#plate-toggle` drives `--plate-a` (1 = on, 0 = off).
- Background: circle swatches (`.ctrl-bg-swatch`). Selected swatch has `outline: 2px solid rgba(255,255,255,0.72)`. Hover shows tooltip via `::after { content: attr(data-label) }`.

All controls text uses **Inter** font (`font-family: 'Inter', -apple-system, …`), set on `.controls-panel` and inherited/explicit on `.ctrl-label`, `.ctrl-value`, `.ctrl-opt`, tooltip.

### Background options

`BG_OPTIONS` array in JS — add objects here to add new background swatches:

```js
const BG_OPTIONS = [
  { id: 'space-grey', label: 'Space Grey', value: 'linear-gradient(135deg, #CECECE 0%, #8E8E94 100%)' },
  { id: 'malibu',     label: 'Malibu',     value: 'linear-gradient(to bottom, #80C8F0 0%, … #D84838 100%)' },
  { id: 'soft-dawn',  label: 'Soft Dawn',  value: 'linear-gradient(to bottom, #90B8F8 0%, … #F4B0B0 100%)' },
  { id: 'blue-skies', label: 'Blue Skies', value: 'linear-gradient(160deg, #58A8E0 0%, … #C6E8FF 100%)' },
  { id: 'rio',        label: 'Rio',        value: 'radial-gradient(150% 80% at 50% 100%, …)' },
  { id: 'midnight',   label: 'Midnight',   value: 'linear-gradient(135deg, #1A1848 0%, … #B088D8 100%)' },
  // ← add future options here
];
```

**Crossfade transition:** switching backgrounds uses a `#bg-fade` overlay (fixed, full-page, `z-index: -1`). Old gradient is set on `#bg-fade` at `opacity: 1`; body background updates to new value; then `#bg-fade` fades to `opacity: 0` over 0.55s, revealing the new background underneath.

### Hint text

`#hint-text` is `position: absolute` inside `#controls-zone` (which is `position: relative`), centred with `top: 50%; left: 50%; transform: translate(-50%,-50%)`. It overlays the controls zone without adding vertical height to the page layout.

**State machine** — five rules govern visibility:

| Trigger | Effect |
|---|---|
| Page load (after keyboard reveal) | Fades in with shimmer sweep |
| Key pressed or physical key | Permanently dismissed; idle timer cleared |
| Mouse enters controls zone | Temporarily hidden; idle timer paused |
| Mouse leaves controls zone | Reappears (if no key interacted); idle timer restarts |
| 30 s idle (controls not open, no key interacted) | Reappears with shimmer |

Key variables: `_keyDone` (permanent dismiss flag), `_ctrlsOpen`, `_hintShown`, `_idleTimer` (30 000 ms).

**Shimmer animation** (`hint-shimmer` keyframe): background-clip text gradient sweeps left→right over 1.1 s. Applied by adding class `shimmer-run`; the class is removed and re-added (with a forced reflow via `el.offsetWidth`) to replay the animation on subsequent appearances.

### Boot sequence

```js
buildKeyboard();          // builds DOM, assigns notes to allKeyEls
// audio first-gesture listeners registered
// note assignment: allKeyEls spread across C3–B6
requestAnimationFrame(() => {
  scaleToFit();
  requestAnimationFrame(() => {
    // keyboard fades in: shell opacity 0→1 (0.7s), inner scale 0.96→1 (0.65s)
    // after 450ms: fireRipple from spacebar
    // after 900ms: _initHint() — hint fades in with shimmer
  });
});
```

### Viewport scaling

`scaleToFit()` runs on load and resize. It reads the keyboard shell's natural width, computes `scale = min(1, viewportWidth / shellWidth)`, and applies `transform: scale(scale)` to both the keyboard shell and controls panel. `marginBottom` is adjusted to collapse the extra whitespace left by the transform.

---

## Key CSS classes / element IDs reference

| Selector | Purpose |
|---|---|
| `#bg-fade` | Fixed full-page overlay used for background crossfade transitions |
| `#hint-text` | "Press any on-screen key…" hint; absolutely positioned inside `#controls-zone` |
| `.keyboard-shell` | Outer container; transparent; holds `::before` plate and `::after` bezel |
| `.white-mode` | Added to `#keyboard-shell` in Light mode; triggers plate + active-key overrides |
| `.key` | Individual key; height `42px * --u`; `overflow: hidden` for ripple clipping |
| `.key.fn-key` | Function row keys; height `28px * --u` |
| `.key.half-key` | ↑ and ↓ arrow keys; `flex: 1` inside `.arrow-ud` fills the space |
| `.key.active` | Pressed state — scale + brightness |
| `.key.ripple-wave` | Wave animation class; applied then removed per-press |
| `.key-modifier` | Bottom-row modifier key (control/option/command) |
| `.key-modifier-right` | Left-side modifiers — content right-aligned |
| `.key-modifier-left` | Right-side modifiers — content left-aligned |
| `.key-sub` | Symbol or secondary character label |
| `.key-main` | Primary character label |
| `.key-ripple` | Touch ripple span (spawned on press, self-removes) |
| `.key-wave-flash` | White overlay for wave visibility in Light mode (self-removes) |
| `.arrow-ud` | Column flex container for ↑/↓; explicit `height: calc(42px * --u)` |
| `.ctrl-bg-swatch` | Background circle swatch button |
| `#hint-text.shimmer-run` | Triggers the left→right shimmer animation on hint text |

---

## Known design decisions & gotchas

- **`--key-shadow` must be in `:root`** — if moved into `.key {}`, JS `setProperty` on the shell won't override it (element-level specificity wins over inherited values).
- **`.arrow-ud` needs explicit `height`** — without it, `flex: 1` on half-keys has no container height to fill against and they shrink to their CSS `height` only (which is shorter than regular keys).
- **White mode ripple** — `filter: brightness()` is invisible on `rgba(255,255,255,0.10)` keys, so `.key-wave-flash` (opacity animation of a white overlay) is spawned in addition to the scale animation.
- **White mode hover uses `filter: none`** — the dark-mode `brightness(1.45)` hover filter has no visible effect on translucent white keys; instead background opacity is bumped from 0.10 → 0.15 and filter is suppressed.
- **Power key colour** — hardcoded to `#A0A0A8` in dark mode; `.white-mode #power-key .key-main { color: var(--key-color) }` overrides it in Light mode.
- **Modifier key colours** must use `var(--key-color)` / `var(--key-sub-color)` (not hardcoded hex) so they theme correctly with the rest of the key in Light mode.
- **Tone.js audio context** — browsers block audio until a user gesture. `_ensureAudio()` is registered as a one-shot listener and must be called before any `triggerAttackRelease`. The first key press will trigger init; the note plays from the second press onward if the context wasn't ready instantly.
- **`#hint-text` z-index** — set to `10` so it renders above the controls panel (which is inside the same `#controls-zone` container). When controls are revealed on hover, they naturally sit behind the hint since the hint has higher z-index; the hint fades out via the state machine before controls need to be interacted with.
- **Background crossfade direction** — `#bg-fade` holds the *old* gradient at opacity 1 on top of the *new* body background, then fades away. This means the body background must be updated *before* `#bg-fade` starts its fade-out transition.
