---
name: motion-craft
description: The practitioner's motion bar — exact easing/spring/duration values, the frequency gate for when NOT to animate, ten non-negotiable animation standards, jank diagnosis, and a reverse-lookup glossary that names any motion effect. Distilled from Emil Kowalski's animation philosophy and skills repo. Use when building, improving, reviewing, or naming ANY UI motion — choosing easings/springs/durations, deciding whether an element should animate at all, finding animation opportunities, auditing or reviewing animation code, or fixing janky/toy-feeling motion. Layered system — `animation` supplies the recipe system, `apple-design` the Apple physics; this skill supplies the craft bar and exact values.
---

# Motion Craft — the bar, the values, the diagnosis

Attribution: the craft rules and exact values below are distilled from **Emil Kowalski's animation philosophy** (animations.dev) and his open skills repo. Layered system for this team: `animation` gives the recipe system (timing tiers, macro/micro choreography), `apple-design` gives Apple's physics values, `motion-craft` gives the craft bar and the exact numbers you cite.

## Values authority — cite these, never approximate

Strong custom curves (built-in CSS easings are too weak):

```css
--ease-out: cubic-bezier(0.23, 1, 0.32, 1);     /* strong ease-out for UI */
--ease-in-out: cubic-bezier(0.77, 0, 0.175, 1); /* strong ease-in-out for on-screen movement */
--ease-drawer: cubic-bezier(0.32, 0.72, 0, 1);  /* iOS-like drawer curve */
```

Easing by intent: entering/exiting → `ease-out` (strength-curve); moving/morphing on screen → `ease-in-out`; hover/color → `ease`; constant motion → `linear`. **Never `ease-in` on UI** — it delays the exact moment the user watches.

Durations (UI stays under 300ms — a 180ms dropdown feels more responsive than a 400ms one):

| Element | Duration |
| --- | --- |
| Button press feedback | 100–160ms |
| Tooltips, small popovers | 125–200ms |
| Dropdowns, selects | 150–250ms |
| Modals, drawers | 200–500ms |
| Marketing/explanatory | can be longer |

Springs (for drag with momentum, "alive" elements, interruptible gestures):

```js
{ type: "spring", duration: 0.5, bounce: 0.2 }              // Apple-style, recommended
{ type: "spring", mass: 1, stiffness: 100, damping: 10 }    // traditional physics
```

Bounce stays 0.1–0.3 and only for drag-to-dismiss/playful moments — most UI wants critically damped. Springs carry velocity through interrupts (keyframes restart from zero) — that's why they own gestures.

The numbers that appear in almost every fix: press scale **0.95–0.98** (`scale(0.97)` + `160ms ease-out` on `:active`); entry floors **`scale(0.9–0.97)` + opacity 0** (never `scale(0)`); stagger **30–80ms** (longer feels slow); velocity dismissal threshold **~0.11** (units/ms — a flick is enough); animated blur **< 20px** (2–5px masks imperfect crossfades).

## The Gate — should it animate at all?

| Frequency | Verdict |
| --- | --- |
| 100+/day (command palette, keyboard shortcuts) | **No animation. Ever.** (Raycast has no open/close — correct.) |
| Tens/day (hovers, list nav) | Remove or reduce to near-imperceptible |
| Occasional (modals, drawers, toasts) | Standard animation |
| Rare / first-time (onboarding, success, celebration) | The delight budget lives here |

Every animation names its purpose from this list: **spatial consistency / state indication / feedback / preventing a jarring change / explanation** (or delight, rare tier only). "It looks cool" is not a purpose on a frequently-seen element. Data the user is reading or acting on doesn't move for style.

## The ten non-negotiables

1. **Justified** — every animation answers "why does this animate?"
2. **Frequency-appropriate** — keyboard-initiated and 100+/day get none.
3. **Responsive easing** — `ease-out` (or strong custom curve) on entrances/exits; `ease-in` on UI is a block.
4. **Sub-300ms UI** — slower needs justification.
5. **Origin & physicality** — popovers/tooltips scale from their trigger (`transform-origin`, e.g. Base UI `var(--transform-origin)`); modals are exempt (centered). Never `scale(0)`.
6. **Interruptible** — rapidly-triggered/gesture motion uses transitions or springs that retarget mid-flight, never restarting keyframes.
7. **GPU-only** — `transform` + `opacity` only; layout properties are a performance finding (see below for shadow and the Framer-shorthand traps).
8. **Accessible** — `prefers-reduced-motion` honored (gentler, not zero: keep opacity/color, drop movement); hover motion gated behind `@media (hover: hover) and (pointer: fine)`.
9. **Asymmetric where deliberate** — slow the user's decision phase (hold-to-confirm fills over 2s linear), snap the system's response.
10. **Cohesion** — motion matches the product's personality (playful may bounce; a dashboard stays crisp). When unsure if motion feels right, deleting it is often the strongest move.

## Flag on sight (escalation triggers)

`transition: all` · `scale(0)` or pure fades with no initial transform · `ease-in` on any UI interaction · animation on keyboard/100+/day actions · UI >300ms unstated · `transform-origin: center` on trigger-anchored popovers · keyframes on toasts/toggles · layout properties (`width/height/margin/padding/top/left`) · Framer `x/y/scale` shorthands on motion that runs during page load/hydration · driving a child's transform via a parent's CSS variable (recalc storm) · missing reduced-motion · ungated `:hover` · symmetric press-and-release timing · everything-at-once entrance where a 30–80ms stagger belongs.

## Remedial preference order (fixes climb this ladder)

1. **Delete** the animation (high-frequency / no purpose) 2. **Reduce** (shorter, smaller, fewer properties) 3. **Fix easing** (`ease-in` → strong `ease-out`) 4. **Fix origin** (`transform-origin` at trigger; `scale(0)` → `scale(0.95)`+opacity) 5. **Make interruptible** (keyframes → transitions/springs) 6. **Move to GPU** (layout props → transform; shorthand → full `transform` string; WAAPI) 7. **Asymmetric timing** 8. **Polish** (blur-masked crossfades, stagger, `@starting-style`) 9. **A11y + cohesion** (reduced-motion, hover gating, personality match).

## Jank diagnosis — symptom table

| Symptom | Cause → fix |
| --- | --- |
| Animation stutters | Layout properties → `transform`/`opacity` composite-only |
| Choppy shadow animation | `box-shadow` repaints per frame → `filter: drop-shadow()` |
| Jank only during navigation/loading | JS/rAF animation on busy main thread → CSS/WAAPI (off-main-thread) |
| Framer `x/y` drops frames | Shorthands = animated CSS variables = not accelerated → full `animate={{ transform: "translateX(100px)" }}` |
| Hand-rolled React animation drops frames | `setState` per frame → `ref.current.style.transform = …` or motion values |
| Drag lags as list grows | Parent CSS-variable read by children → set `transform` on the element directly |
| Blur transition lags (Safari) | Animated `blur()` over 20px → stay ≤ 20px, 2–5px for crossfades |
| Unintended properties animate | `transition: all` → name the exact properties |
| 1px shift at motion start/end | Layer promotion hand-off → `will-change: transform`, only once seen |

Frame budget: 16.7ms at 60fps, **8ms at 120Hz** — design against the smaller number. Never conclude "smooth" from a fast laptop; the bar is a mid-range phone and it needs a recording, not a feeling.

## Working glossary — reverse-lookup (name what the user describes)

**Entrances/exits**: fade/slide/scale in · **pop in** (slight overshoot landing) · reveal (clip-path/mask uncovers) · enter/exit.
**Sequencing**: keyframes · interpolation/tween · **stagger** (one-after-another cascade) · orchestration · stepped animation · fill mode.
**Transforms**: translate/scale/rotate/skew · 3D tilt/flip · perspective · transform-origin · **origin-aware animation** (grows from its trigger, e.g. popover from the button — the polish that makes popovers feel physical).
**State transitions**: crossfade · continuity transition · **shared element transition** (travels between positions, thumbnail→card) · layout animation (element animates to its new spot) · morph (one shape becomes another — Dynamic Island) · accordion/collapse · direction-aware transition · **view transition** (browser-morphed pages).
**Scroll**: scroll reveal · scroll-driven animation · parallax · page transition.
**Feedback**: hover effect · **press/tap feedback** (subtle physical scale-down) · hold-to-confirm (progress fill) · drag (with momentum on release) · drag-to-reorder · swipe-to-dismiss · **rubber-banding** (resist + snap-back past an edge — iOS overscroll) · shake/wiggle (error signal) · ripple.
**Springs/physics**: spring · stiffness/tension (snappier) · damping (settle speed; lower = bouncier) · mass · bounce · perceptual duration · momentum · velocity (carried through interrupts) · **interruptible animation**.
**Loops/ambient**: marquee · loop · alternate/yoyo · orbit · pulse · float · idle animation.
**Polish**: blur (softens + masks imperfections) · clip-path · mask (soft-edge clip) · line drawing (SVG self-draws) · text morph · **skeleton/shimmer** · number ticker (+ **tabular numbers** so digits don't jitter) · typewriter.
**Principles**: purposeful animation · anticipation (wind-up before the move) · follow-through (settle after the main motion) · squash & stretch · **perceived performance** (the right motion makes UI *feel* faster, even when it isn't) · frequency of use (seen more → shorter/subtler) · spatial consistency · reduced motion.

Use: quote glossary descriptions as-is when naming; if no term fits, say so plainly — closest term + "that's an X of Y" (e.g. "a stagger of scale-in entrances").

## Output formats

**Reviewing motion** (one table, then verdict): findings as `Before | After | Why` — exact values cited from this file, never approximated — then verdict tiers (feel-breaking regressions → missed simplifications → performance → interruptibility/timing → origin/cohesion → accessibility) and an explicit call: **Block** (feel-breaking, keyboard-action animation, `scale(0)`, `ease-in` on UI, GPU-fixable non-GPU animation) or **Approve**. Cite `file:line`. Uncertain feel → prescribe the slow-motion/frame-by-frame check, don't guess.

**Finding opportunities** (the gate as a filter): sweep the seams — pressables with no `:active`, teleporting conditionals (`{isOpen &&`, `display:none` toggles), accordions that snap, trigger-disconnected popovers, dismissable surfaces exiting wrong-edged, group entrances without stagger, snaps without physics, rare moments rendered flat. Cap at 5–7 suggestions, each with exact values, ranked by leverage — and **list the rejected candidates with the gate question that killed them**. An interface can be already-right; saying so is a success.

Building new motion: reach for `animation` (recipes/tiers) + `apple-design` (Apple physics) and implement inside these values; CSS transitions/`@starting-style`/WAAPI for predetermined motion, springs for dynamic/interruptible/gesture motion.
