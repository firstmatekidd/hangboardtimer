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
        │   │     └── #set-counter            — "Set X of Y"
        │   ├── <section> #controls-section
        │   │     ├── <h2> "Controls"         — visually hidden, for screen reader nav
        │   │     └── #controls               — Start/Pause + Reset buttons
        │   ├── <section> #settings-section
        │   │     ├── <h2> "Settings"         — visible heading
        │   │     └── #settings               — Hang time / Rest time / Sets / Sound toggle
        │   ├── <section> #help-section
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
- A visible text countdown (`<div role="timer" aria-live="assertive">`) displayed prominently below (or centered over) the ring, sized with `clamp()` / `vw` units. Both the ring and text count down in sync — the ring is a visual reinforcement, not a replacement.
- State label below: "HANG" (orange) or "REST" (blue).

### 2. Set Counter (`#set-counter`)
- Text display: `Set 2 of 5`
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
  - **Sets** — label reads "Sets / count", centered above input, default: 5
- Labels use "seconds" in full (not "(s)") to avoid ambiguity between the unit and plurality.
- Labels are centered above their respective inputs; label text is visibly sized (13px) for readability.
- Inputs are clearly editable fields — styled with a visible border and background.
- Inputs are `type="number"` with `inputmode="numeric"` for mobile keyboards.
- All number inputs have `min="1"` to prevent zero or negative values; JS validation also clamps any manually typed value to a minimum of 1 on blur.
- **Focus indicators**: Settings inputs and the sound checkbox use a white `outline` with 2px offset (`outline: 2px solid #fff; outline-offset: 2px`) so the ring sits visibly clear of the control edge. This overrides the browser default for these elements only — the rest of the app uses the standard accent-color focus ring.
- **Sound toggle + Help button row**: The "Enable sounds" checkbox and the "? Help" disclosure button sit on the same row, space-between, so they balance each other visually.
- **Sound toggle**: A semantic `<input type="checkbox">` paired with a `<label>`, labeled "Enable sounds". Checked by default. When unchecked, all Web Audio API tones are suppressed.
- Settings are disabled while timer is running; re-enabled on reset.
- **Persistence**: All three time/set settings plus the sound toggle saved to `localStorage` on change under keys `hbt_hangTime`, `hbt_restTime`, `hbt_sets`, `hbt_soundEnabled`. Values are loaded from `localStorage` on page load, falling back to defaults if not set.

### 5. Help Disclosure (`#help-disclosure`)
- A `<button>` labeled "? Help" that toggles an inline `<div>` panel — no popup, no modal.
- Positioned on the same row as the "Enable sounds" checkbox, aligned to the right, so both sit at the bottom of the settings section.
- Implemented with `aria-expanded` on the button and `aria-controls` pointing to the panel `id`.
- Panel uses `hidden` attribute (not `display:none` via CSS) so assistive technologies respect the collapsed state correctly.
- Panel expands directly below the settings section, above the affiliate card.
- Panel contains placeholder text explaining how to use the app; content to be filled in before launch.

### 6. Sound
- Two short tones generated via the **Web Audio API** (no external audio files needed):
  - **Hang tone**: high-pitched short beep (~880 Hz, 150ms) — signals start of hang.
  - **Rest tone**: lower, longer tone (~440 Hz, 300ms) — signals start of rest.
- Sounds triggered on each state transition.
- Audio context initialized on first user interaction (to comply with autoplay policy).

### 6. Affiliate Link (`#affiliate`)
- A styled card with:
  - Placeholder image (inline SVG icon of a hangboard).
  - Text: "Train on the same board I use →"
  - `<a>` tag linking to Amazon affiliate URL (placeholder `#` initially).
  - `rel="noopener sponsored"` on the link.

### 7. Ad Section (`#ad-section`)
- A clearly demarcated banner at the bottom.
- Placeholder text: "Advertisement" with a fixed-height container (e.g. 90px) ready to accept a Google AdSense script or image ad.

---

## Timer Logic (JavaScript)

```
State machine:
  IDLE → (start) → HANGING → (hang timer hits 0, play rest tone) → RESTING
       → (rest timer hits 0, play hang tone) → HANGING
       → (after N sets complete) → DONE → IDLE

Variables:
  - currentState: 'idle' | 'hanging' | 'resting' | 'done'
  - currentSet: number
  - timeRemaining: number (seconds)
  - intervalId: reference to setInterval

On each tick (1s):
  - Decrement timeRemaining
  - Update countdown display
  - Update SVG ring offset
  - If timeRemaining === 0: transition state
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

- [ ] `role="timer"` and `aria-live="assertive"` on the text countdown element.
- [ ] SVG progress ring has `aria-hidden="true"` (decorative — screen readers use text countdown).
- [ ] All interactive elements are semantic `<button>` or `<input>` elements.
- [ ] Sound toggle uses `<input type="checkbox">` with an associated `<label>`.
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

- Layout stacks vertically on mobile (default).
- Settings inputs displayed in a 3-column grid on wider screens.
- Timer ring scales with `vw` units for visibility across screen sizes.
- Bottom sections (affiliate + ad) are full-width, comfortable thumb-tap targets.

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
4. Timer JS state machine + tick logic
5. Web Audio API tones
6. Settings inputs (semantic `<label>` + `<input>`) + wiring to timer
7. localStorage save/load for settings + sound toggle (`hbt_soundEnabled`)
8. Start/Pause + Reset `<button>` logic
9. Set counter display
10. Hang/Rest visual state transitions (color, label, ring color)
11. Help disclosure button + inline panel (`aria-expanded`, `aria-controls`, `hidden`)
12. Affiliate link card
12. Ad section placeholder
13. Accessibility audit (aria, focus, labels, aria-hidden)
14. Responsive polish — verify viewport-relative sizing throughout