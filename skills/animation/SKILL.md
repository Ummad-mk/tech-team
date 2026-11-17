---
name: animation
description: Motion design system — vocabulary, timings, easings, and recipes for macro animations (entrances, transitions, choreography) and micro animations (press feedback, hovers, toasts, loaders) across platforms (CSS/JS, React, SwiftUI, Android). Use when adding or reviewing any UI motion — entrance animations, transitions, hover/press states, skeletons, toasts, sheets, page loads — or fixing janky/toy-feeling animation.
---

# Animation — macro & micro motion system

Motion is behavior, not decoration. Any animation you add must answer: **what does it tell the user** (what moved, where focus went, what just happened) — motion that answers nothing gets cut.

## The timing vocabulary
| Tier | Duration | Use for |
| --- | --- | --- |
| Instant | 80–120ms | Press/active feedback, hover state settles, focus ring appear |
| Quick | 180–240ms | Micro transitions: toggles, tooltips, small reveals, checkbox checks |
| Standard | 300–360ms | Toasts, menus, color/theme changes, small sheets |
| Deliberate | 400–480ms | Modals, large sheets, page-level transitions, shared elements |

- **Easing by intent**: entrances → `cubic-out`-family (fast start, gentle landing); exits → `cubic-in` (clearing out); loops → `ease-in-out` or linear-for-continuous. **Springs for anything positional**: critically damped by default (no overshoot) — bounce ONLY for momentum-driven gestures (a flicked sheet settling), never for button presses.
- Interruptibility is a law: animation always starts FROM the current state (interpolated), never from keyframe 0 — users should be able to re-click/re-toggle mid-flight without visual snapping.
- Stagger rhythm for choreography: **40–80ms between siblings** (small lists faster, hero sections slower); total choreography ≤ ~900ms or the page feels slow.

## Micro animations (the texture of quality)
- **Press feedback**: on pointer-down scale to ~0.96–0.98 (transform only), release springs back; sometimes opacity drop + scale reads better on dense UI. Never wait for the release to react — the pointer-down is the feedback moment.
- **Hover**: translate/lift ≤ 4px or shadow intensify — never rescale text-heavy rows (layout shimmer is beginner). Focus rings: animate width/opacity, then settle — keyboard users deserve the same fluidity.
- **Toasts**: entrance from edge with standard tier + slight translate; exit Early (they feel faster going out than coming in — that's correct). Auto-dismiss pauses on hover. Success = brief checkmark morph; error = shrink+shake at most once, never looping.
- **Toggles/thumbs**: spring physics; thumb leads, track color transitions with it; haptics on native platforms where available.
- **Loaders**: skeleton shimmer ≤ 0.5% surface brightness delta, subtle loop (1.2–1.6s); button loading morph (label crossfade → spinner inline) beats replacing the button wholesale.
- **Number/progress changes**: count-up for ≥ 1s displays only, duration proportional to delta; instant for small rapid updates.

## Macro animations (structure & choreography)
- **Page/section entrance**: opacity 0→1 + translateY 8–12px (NOT 50px — small distance, big polish); one wave top-to-bottom or by importance; decor first, content lands last; run once per visit, not on every scroll-back.
- **Route transitions**: fade+subtle scale (0.98→1) or shared-element where the platform supports it (matched geometry); direction consistency — forward pushes left, back reveals right.
- **Modals/sheets**: enter with settle (slight overshoot ≤ 2% or spring), dismiss with LESS travel than entrance (exits are underplayed); drag-to-dismiss with velocity threshold (~500px/s) OR displacement ratio (>40%) — whichever fires first.
- **Lists (add/remove)**: FLIP technique — measure, reorder, invert, play; removal animates height/opacity collapse, siblings close the gap; insertion slides in from the occupied slot.
- **Skeleton → content**: don't shimmer-jarringly swap; crossfade skeletons into real content when data arrives fast (<300ms: skip animation entirely).

## Physics & interruption rules (non-negotiable)
- Animate only **transform + opacity** in web (compositor-friendly, 60fps); animating layout properties (width/top/margin) = jank. `will-change` as a temporary local override, otherwise removed.
- Reduced-motion contract: every nonessential entrance/parallax/loop is suppressed under `prefers-reduced-motion`; opacity-only stand-ins keep meaning intact.
- No uniform "animate everything": motion creates hierarchy — the important thing moves more/first; five elements animating equally = noise.
- Performance: measure frame drops under simulated load (scrolling during animations); a janky 300ms feels worse than a crisp instant.

## Platform cheat sheets
- **CSS**: durations/easings as tokens (`--dur-quick`, `--ease-out-soft`); FLIP via two rAF reads + transform; `:where()` overrides for reduced-motion; never JS-trigger transitions on :hover.
- **React/JS**: Web Animations API or spring solver (no jQuery chains); unmount animation via two-phase state (exit class → transitionend); FLIP via getBoundingClientRect pairs.
- **Swift/iOS**: `matchedGeometryEffect` for shared elements; `.spring(response: 0.3–0.5, dampingFraction: 0.8–1.0)` — 1.0 for UI chrome, below for momentum gestures; `withTransaction({$0.disablesActions = true})` to suppress implicit animations during data updates.
- **Android**: physics-based spring (DynamicAnimation) overInterpolator timing; MotionLayout for coordinated sequences; predictive back gestures animate the real thing.
- **Rich/lotties**: Lottie for hand-off designer animations (brand moments, loaders); never for basic UI chrome — vector code animates cheaper and smoother.
