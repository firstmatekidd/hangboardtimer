# Hangboard Timer — Build Plan

## Overviews

A single-file (`index.html`) web app for hangboard interval training. All CSS, JS, fonts (via CDN), and inline SVG/base64 assets live in one file to minimize latency on mobile connections.

---

## Aesthetic Direction

**Industrial Minimalism meets Climbing Wall.**

- **Palette**: Near-black background (`#0f0f0f`), chalk-white text (`#f0ede8`), with a single punchy accent — carabiner orange (`#e8621a`) for active/hang states and a cool slate-blue (`#4a7fa5`) for rest states.
- **Typography**: System sans-serif stack to eliminate font CDN latency — `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Ubuntu, Cantarell, 'Helvetica Neue', Arial, sans-serif`. Covers macOS/iOS → Windows → Android → Linux → legacy fallbacks → generic. Bold/heavy weights used for the countdown to retain athletic character.
- **Texture**: Subtle grain overlay via an inline SVG `<feTurbulence>` filter to evoke chalk dust and raw wood.
- **Motion**: The timer digit transitions with a fast CSS scale pulse on each second tick. State transitions (hang → rest) trigger a full background color shift with a 300ms ease.
- **Differentiator**: The countdown circle is a large SVG `stroke-dashoffset` progress ring — visceral and clear at a glance from across a room.

---

## File Structure

Single file: `index.html`

```
index.html
  └── <head>
        └── <style> — all CSS (no external font CDN)
  └── <body>
        ├── #app-wrapper
        │   ├── <header>         — App title (<h1>)
        │   ├── <section> #timer-section
        │   │     ├── <h2> "Timer"            — visually hidden, for screen reader nav
        │   │     ├── #timer-display          — SVG progress ring + text countdown + state label
        │   │     └── #hang-counter            — "Hang X of Y"
        │   ├── <section> #controls-section
        │   │     ├── <h2> "Controls"         — visually hidden, for screen reader nav
        │   │     └── #controls               — Start/Pause + Reset buttons
        │   ├── <section> #settings-section
        │   │     ├── <h2> "Settings"         — visible heading
        │   │     └── #settings               — Hang time / Rest time / Repeats / Sound toggle / Haptic toggle
        │   ├── <section> #help-section
        │   │     ├── <h2> "Help"              — visually hidden, for screen reader nav
        │   │     └── #help-disclosure      — <button> + collapsible <div> (inline, no popup)
        │   ├── <section> #affiliate-section
        │   │     ├── <h2> "Recommended Gear" — visible heading
        │   │     └── #affiliate              — Hangboard product link block
        │   └── <section> #ad-section
        │         ├── <h2> "Advertisement"    — visually hidden, for screen reader nav
        │         └── Ad placeholder banner
        └── <script> — all JS (inline)
```

Section headings use a `.visually-hidden` CSS class (positioned off-screen, not `display:none`) where they don't suit the visual design but are needed for screen reader landmark navigation.

---

## Component Breakdown

### 1. Timer Display (`#timer-display`)
- Large SVG circle as a progress ring using `stroke-dasharray` / `stroke-dashoffset`. SVG ring marked `aria-hidden="true"` since it is decorative — screen readers use the text countdown instead.
- A visible text countdown (`<div role="timer" aria-live="off">`) displayed prominently below (or centered over) the ring, sized with `clamp()` / `vw` units. Both the ring and text count down in sync — the ring is a visual reinforcement, not a replacement.
- A separate visually-hidden `<div aria-live="polite" aria-atomic="true">` used exclusively for state-transition announcements (e.g. "Hang — number 2 of 5", "Rest", "Workout complete!"). This fires only on state changes, not every second, to avoid overwhelming screen reader users. The per-second digit countdown is intentionally silent to assistive tech (`aria-live="off"` on the timer element).
- State label below: "HANG" (orange) or "REST" (blue).

### 2. Repeat Counter (`#hang-counter`)
- Text display: `Hang 2 of 5`
- Updates after each full hang+rest cycle.

### 3. Controls (`#controls`)
- All interactive elements use semantic `<button>` elements — no `<div>` or `<span>` click handlers.
- **Start/Pause** `<button>` — large tap target (min 8vh height), toggles label.
- **Reset** `<button>` — returns to initial state without changing settings.
- Keyboard focus indicators via `outline` styling (never `outline: none`).
- `aria-label` attributes on all buttons; `role="timer"` on countdown element.

### 4. Settings (`#settings`)
- Three labeled number inputs with large tap targets:
  - **Hang** — label reads "Hang / seconds", centered above input, default: 10
  - **Rest** — label reads "Rest / seconds", centered above input, default: 30
  - **Repeats** — label reads "Repeats / count", centered above input, default: 5. ("Repeats" replaces "Sets" to make clear this is the number of hang/rest cycles that will repeat.)
- Labels use "seconds" in full (not "(s)") to avoid ambiguity between the unit and plurality.
- Labels are centered above their respective inputs; label text is visibly sized (13px) for readability.
- Inputs are clearly editable fields — styled with a visible border and background.
- Inputs are `type="number"` with `inputmode="numeric"` for mobile keyboards.
- All number inputs have `min="1"` to prevent zero or negative values; JS validation also clamps any manually typed value to a minimum of 1 on blur.
- Maximum values enforced via `max` attribute and JS blur clamping: hang ≤ 300s, rest ≤ 600s, repeats ≤ 99. Prevents accidental multi-hour sessions from a typo.
- **Settings remain editable while the timer is running.** Changes take effect as follows:
  - Hang/Rest time changes: applied at the start of the next phase (not mid-phase, to avoid jarring jumps). The current phase completes at its original duration.
  - Repeats change: applied immediately — if the user lowers the repeat count below `currentHang`, the session ends at the current hang's completion.
- **Focus indicators**: Settings inputs and checkboxes use a white `outline` with 2px offset (`outline: 2px solid #fff; outline-offset: 2px`) so the ring sits visibly clear of the control edge.
- **Toggles row**: The "Sounds" checkbox, "Vibration" checkbox, and the "? Help" disclosure button sit on the same row. Sounds and Vibration checkboxes are grouped on the left; Help button on the right.
- **Sound toggle**: A semantic `<input type="checkbox">` paired with a `<label>`, labeled "Sounds". Checked by default. When unchecked, all Web Audio API tones are suppressed. Default is to be toggled on.
- **Haptic toggle**: A semantic `<input type="checkbox">` paired with a `<label>`, labeled "Vibration". Checked by default. When unchecked, all `navigator.vibrate()` calls are suppressed. Default is to be toggled off.
- **Persistence**: All settings saved to `localStorage` on change: `hbt_hangTime`, `hbt_restTime`, `hbt_repeats`, `hbt_soundEnabled`, `hbt_hapticEnabled`. Loaded on page load with fallback to defaults.

### 5. Help Disclosure (`#help-disclosure`)
- A `<button>` labeled "? Help" that toggles an inline `<div>` panel — no popup, no modal.
- Positioned on the same row as the "Sounds" checkbox, aligned to the right, so both sit at the bottom of the settings section.
- Implemented with `aria-expanded` on the button and `aria-controls` pointing to the panel `id`.
- Panel uses `hidden` attribute (not `display:none` via CSS) so assistive technologies respect the collapsed state correctly.
- Panel expands directly below the settings section, above the affiliate card.
- Panel contains placeholder text explaining how to use the app; content to be filled in before launch.

### 6. Sound & Haptics
- All tones generated via the **Web Audio API** (no external audio files needed):
  - **Tick tone**: soft, neutral click (~1200 Hz, 40ms, low gain ~0.15) — used for the 3-second get-ready countdown and for the final 3 seconds of every hang and rest phase so users know the transition is imminent.
  - **Hang tone**: high-pitched short beep (~880 Hz, 150ms) — signals start of hang.
  - **Rest tone**: lower, longer tone (~440 Hz, 300ms) — signals start of rest.
  - **Done tone**: a short descending two-note chime (~660 Hz then ~440 Hz, ~200ms each) — signals workout complete. Distinct from the hang/rest tones so users know the session has ended.
- Tick logic: on every tick, if `timeRemaining <= 3` AND current state is `get_ready`, `hanging`, or `resting`, play the tick tone instead of (or in addition to) any transition tone.
- Sounds triggered on each state transition.
- Audio context initialized on first user interaction (to comply with autoplay policy).
- **Haptic feedback**: `navigator.vibrate()` called alongside each audio cue as a fallback for silent-mode users:
  - Tick: short pulse (30ms).
  - Hang tone: sharp double pulse (80ms on, 40ms off, 80ms on).
  - Rest tone: single medium pulse (150ms).
  - Done tone: long-short-long pattern (200ms on, 80ms off, 100ms on).
  - Haptics are suppressed when the haptic toggle is off.
  - `navigator.vibrate` is not supported on iOS — this is a known limitation; no workaround exists in the browser. The app degrades silently (no error shown).
- **Haptic toggle**: A semantic `<input type="checkbox">` paired with a `<label>`, labeled "Vibration". Checked by default. Saved to `localStorage` under key `hbt_hapticEnabled`.

### 7. Affiliate Link (`#affiliate`)
- A styled card with:
  - Placeholder image (inline SVG icon of a hangboard).
  - Text: "Train on the same board I use →"
  - `<a>` tag linking to Amazon affiliate URL (placeholder `#` initially).
  - `rel="noopener sponsored"` on the link.

### 8. Ad Section (`#ad-section`)
- A clearly demarcated banner at the bottom.
- Placeholder text: "Advertisement" with a fixed-height container (e.g. 90px) ready to accept a Google AdSense script or image ad.

---

## Timer Logic (JavaScript)

```
State machine:
  IDLE → (start) → GET_READY (3s countdown) → HANGING → (hang timer hits 0, play rest tone) → RESTING
       → (rest timer hits 0, play hang tone) → HANGING
       → (after N repeats complete, play done tone) → DONE
       → (user presses Reset) → IDLE

GET_READY state behaviour:
  - Fixed 3-second countdown before the first hang only (not between hangs).
  - State label reads "GET READY" in a neutral white/grey.
  - SVG ring drains over 3 seconds (uses the same ring/tick mechanism).
  - Tick sound plays on each of the 3 seconds (see Sound section).
  - `aria-live` region announces "Get ready" once on entry.
  - Cannot be paused (too short) — Pause during GET_READY is ignored or immediately cancels back to IDLE.

DONE state behaviour:
  - SVG ring fills completely (dashoffset = 0) and stays filled.
  - State label changes to "DONE" in a neutral white/grey (neither orange nor blue).
  - Countdown display shows "0".
  - `aria-live` region announces "Workout complete!" once (injected as a one-shot text update).
  - Start/Pause button is hidden or disabled; only Reset is active.

Variables:
  - currentState: 'idle' | 'get_ready' | 'hanging' | 'resting' | 'done'
  - currentHang: number
  - timeRemaining: number (seconds)
  - intervalId: reference to hangInterval
  - phaseStartTime: number (Date.now() snapshot taken at each state transition)
  - phaseDuration: number (total seconds for the current phase)

On each tick (1s):
  - Compute timeRemaining from wall clock: Math.ceil(phaseDuration - (Date.now() - phaseStartTime) / 1000)
    — do NOT decrement a counter; derive from elapsed time to prevent drift over long sessions.
  - Update countdown display
  - Update SVG ring offset
  - If timeRemaining <= 0: transition state

Note: hangInterval alone drifts. Always recompute remaining time from Date.now() rather than
decrementing a variable, so a 5-hang session with long rests stays accurate to wall-clock time.
```

---

## CSS Architecture

- CSS custom properties (variables) on `:root` for all colors, font sizes, spacing.
- Mobile-first layout using flexbox, single-column.
- Breakpoint at `min-width: 480px` for slight padding/sizing expansion on tablets.
- Sizes expressed in viewport-relative units (`vw`, `vh`, `svh`) or percentages wherever practical — fixed `px` values only for borders and minimum thresholds (e.g. `min-width: 44px` for tap targets).
- `clamp()` for fluid font sizes on the countdown (e.g. `clamp(3rem, 18vw, 9rem)`).
- Timer ring sized as a percentage of viewport width (e.g. `width: min(60vw, 60vh)`).
- `.hang-state` / `.rest-state` classes toggled on `#app-wrapper` to drive color transitions globally.
- No external font CDN — system sans-serif stack only.

---

## Accessibility Checklist

- [ ] `role="timer"` and `aria-live="off"` on the text countdown element (per-second digits are silent to AT).
- [ ] Separate visually-hidden `aria-live="polite"` region for state-transition announcements only ("Get ready", "Hang — repeat X of Y", "Rest", "Workout complete!").
- [ ] SVG progress ring has `aria-hidden="true"` (decorative — screen readers use text countdown).
- [ ] All interactive elements are semantic `<button>` or `<input>` elements.
- [ ] Sound toggle uses `<input type="checkbox">` with an associated `<label>`.
- [ ] Haptic toggle uses `<input type="checkbox">` with an associated `<label>`.
- [ ] `aria-label` on Start/Pause (dynamic: reflects current state).
- [ ] `aria-label` on Reset button.
- [ ] All inputs have associated `<label>` elements.
- [ ] Page sections wrapped in `<section>` elements each with an `<h2>` heading.
- [ ] Help disclosure uses `<button aria-expanded>` + `aria-controls` pointing to panel `id`.
- [ ] Help panel uses `hidden` attribute (not CSS `display:none`) for correct AT collapsed-state handling.
- [ ] Visually-hidden headings use `.visually-hidden` CSS (off-screen, not `display:none`) so they remain in the accessibility tree.
- [ ] Focus ring visible on all interactive elements (`:focus-visible` styling).
- [ ] Settings inputs and sound checkbox use white `outline` with `outline-offset: 2px` so the ring clears the control edge clearly.
- [ ] Color is not the only differentiator between hang/rest states (also label text + tone).
- [ ] Minimum tap target size 44×44px (buttons exceed this).

---

## Responsive Design

- Layout stacks vertically on mobile (default, portrait).
- Settings inputs displayed in a 3-column grid on wider screens.
- Timer ring scales with `vw` units for visibility across screen sizes.
- Bottom sections (affiliate + ad) are full-width, comfortable thumb-tap targets.

### Landscape Mode
- Triggered via `@media (orientation: landscape) and (max-height: 500px)` to target phones in landscape specifically (not wide desktop windows).
- Layout switches to a two-column grid: timer ring + state label on the left, controls + settings on the right.
- Timer ring shrinks to fit the reduced viewport height (e.g. `width: min(42vh, 42vh)`).
- Affiliate and ad sections are hidden in landscape to keep the workout UI uncluttered; they reappear in portrait.
- Font sizes re-clamped for the reduced height (e.g. countdown uses `clamp(2rem, 10vh, 5rem)`).

### WakeLock
- On `Start`, request the Screen Wake Lock API: `navigator.wakeLock.request('screen')`.
- Store the lock handle; release it on `Reset`, `DONE`, and `visibilitychange` (page hidden).
- Re-acquire the lock on `visibilitychange` when the page becomes visible again and the timer is running (the OS releases wake locks when the page is backgrounded).
- Wake Lock is not supported in all browsers (notably iOS Safari < 16.4). Degrade silently — do not show an error. The help panel should note that users on unsupported browsers may need to adjust their screen timeout settings manually.

---

## Deliverable

- **File**: `index.html`
- **Dependencies**: None — no external CDN required. Works fully offline.
- **No build step** required — open directly in browser.

---

## Build Order

1. HTML skeleton + meta tags (viewport, charset, theme-color)
2. CSS variables + base reset + system font stack
3. Timer display (SVG ring with `aria-hidden` + separate text countdown)
4. Timer JS state machine + tick logic (including GET_READY state)
5. Web Audio API tones (tick, hang, rest, done)
6. Haptic feedback (`navigator.vibrate()` calls alongside each tone)
7. Settings inputs (semantic `<label>` + `<input>`) + live-update wiring to running timer
8. localStorage save/load for all settings including sound + haptic toggles
9. Start/Pause + Reset `<button>` logic
10. Repeat counter display
11. Hang/Rest/Get-Ready visual state transitions (color, label, ring color)
12. DONE state (visual treatment, aria announcement, button states)
13. WakeLock (acquire on start, release on reset/done, re-acquire on visibility restore)
14. Landscape layout (`@media orientation: landscape`)
15. Help disclosure button + inline panel (`aria-expanded`, `aria-controls`, `hidden`)
16. Affiliate link card
17. Ad section placeholder
18. Accessibility audit (aria, focus, labels, aria-hidden)
19. Responsive polish — verify viewport-relative sizing in both orientations