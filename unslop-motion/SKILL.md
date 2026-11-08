---
name: unslop-motion
description: Design and review UI motion with 20 concrete animation rules. Use when adding transitions, micro-interactions, drag gestures, page transitions, loading feedback, or when animation feels generic, floaty or overdone.
---

# Unslop Motion

A followable procedure for making interface motion feel intentional. It works on live pages, prototypes, component code or written interaction specs. It does not require a specific framework or model.

## Files

- `SKILL.md`: procedure, motion budget and the 20 rules.
- `checklist.md`: one-screen pass/fail checklist for a fast audit.
- `rules.md`: rationale, implementation notes and acceptance tests for each rule.
- `examples.md`: before/after snippets and a worked mini audit.

Start with this file. Read `checklist.md` for a quick review, `rules.md` for implementation detail, and `examples.md` when the user needs to see the fix.

## When to use

- Adding any transition or micro-interaction: menus, modals, sheets, lists, toggles, sliders, drag gestures, page changes.
- Reviewing motion that feels floaty, laggy, bouncy, mechanical or overdone.
- A user says "make it feel smoother", "add some motion", "this animation is weird", or "make it feel alive".
- Defining motion tokens for a design system.

## When not to use

- The user wants a deliberately playful, cartoon or game-like feel. Keep the timing and spatial rules, relax the restraint rules.
- The problem is backend latency rather than animation. Fix the latency first.

## Core principle

Motion must explain change.

Every animation should answer at least one of these questions:

1. Where did this come from?
2. What changed?
3. What can I do now?

If it answers none of them, remove it. Speed is part of the design: a beautiful animation that delays the next action is a bad animation.

## Motion budget

| Interaction | Duration |
|---|---|
| Press feedback | 80-120ms |
| Tooltip | 100-150ms |
| Icon swap | 120-180ms |
| Popover, dropdown, menu | 150-250ms |
| Modal, sheet, drawer | 200-350ms |
| Page or route transition | 250-450ms |
| List stagger | 20-40ms per item |
| Exit | 80% of the enter duration |

If an animation needs to be longer than these ranges to make sense, the transition is doing too much. Split it into two simpler changes.

## Easing tokens

| Token | Curve | Use |
|---|---|---|
| `ease-out` | `cubic-bezier(0.16, 1, 0.3, 1)` | Entrances, reveals, anything arriving |
| `ease-in` | `cubic-bezier(0.7, 0, 0.84, 0)` | Exits, anything leaving |
| `ease-in-out` | `cubic-bezier(0.65, 0, 0.35, 1)` | Moves, reorders, resizes, state changes in place |
| `spring` | damping 1.0, response 0.3-0.4s | Physical release, drag settle, success |
| `linear` | `linear` | Progress bars, marquees, continuous loops |

Browser defaults are weaker than these curves. Define the tokens once and reuse them. Two curves per product is usually enough: one for entrances, one for moves.

## The 40-second pass

Do not inspect everything. Scan in four phases and fix in order of impact.

| Phase | Rules | Look for |
|---|---|---|
| Timing (10s) | 1, 2, 3, 9, 15 | Linear entrances, 400ms menus, over-staggered lists, exits as long as entrances |
| Space (10s) | 7, 10, 11, 12, 16, 20 | Center-origin popovers, straight-line moves, cuts, empty frames between states |
| Feel (10s) | 4, 5, 8, 13, 14, 19 | No press feedback, laggy drags, bouncy menus, harsh bounds, endless pulses |
| Reveal (10s) | 6, 17, 18 | Details popping in as a block, animated width/height/colour, hard icon swaps |

## How to run

1. Classify the motion: entrance, exit, move, press, drag, reveal, state swap or shared element.
2. Pick the duration from the motion budget and the easing from the tokens.
3. Implement with `transform`, `opacity` or a mask. Never animate layout properties.
4. Make it interruptible, then test with reduced motion, slow input and rapid repeated actions.
5. Verify the handoff: press to drag, drag to release, open to close, trigger to popover.

## Context profiles

Give extra weight to these rules on these surfaces:

| Surface | Extra weight |
|---|---|
| Dropdown, popover, menu | 2, 10, 11, 12, 15 |
| List or grid | 3, 6, 16 |
| Slider, knob, drag surface | 5, 14, 19 |
| Page or route transition | 1, 9, 16, 20 |
| Icon toggle | 4, 18 |
| Success confirmation | 8, 13 |
| Modal or sheet | 1, 12, 13, 16 |

## Severity order

Fix in this order. A lower item never justifies leaving a higher one broken.

1. **Safety and input**: reduced motion, no motion sickness, no blocked input, no tween while held.
2. **Spatial correctness**: origin, direction, overlap, shared elements.
3. **Timing**: durations, easing, exit budget, stagger caps.
4. **Interaction feel**: press, stretch, pulse, rubber band, bounce restraint.
5. **Technique**: masks, icon crossfades, rendering performance.

## The 20 rules

Each rule has an acceptance test. A check either passes or fails; "close enough" is a fail.

### 1. Ease-out for entrances

Elements should arrive with momentum and settle, not accelerate into the screen. Use `ease-out` at 150-300ms, combine opacity with a small translate (4-12px) or scale (0.96 to 1), and keep the distance short.

**Test:** the element settles into place instead of stopping dead or speeding up.

### 2. Dropdowns at 150-250ms

Menus are opened constantly, so latency is felt. Aim for 180ms on enter and roughly 140ms on exit. Under 150ms reads as a teleport; over 250ms reads as lag.

**Test:** open and close ten times. It should feel instant but still visible.

### 3. Stagger list rows by 30ms

A 30ms offset implies order and causality. Use 20-40ms per row, cap the total delay at about 200-300ms, and start from the top or from the point of interaction.

**Test:** the last row appears within roughly 300ms of the first. Fifty rows do not take 1.5 seconds.

### 4. Scale to 0.97 on press

Press feedback confirms input without moving layout. Scale to 0.97 over 80-120ms, release over 120-180ms, and reset on pointer cancel or leave.

**Test:** press, drag off, release. The control returns cleanly and never sticks at the pressed size. 0.8 is a toy; 0.97 is a button.

### 5. Stretch moving knobs 12%

Squash and stretch sells physicality. Stretch up to 12% along the axis of motion while dragging or flicking, reduce the perpendicular axis slightly, and reset on release.

**Test:** flick the knob. It stretches in the direction of travel, then settles without wobbling at rest.

### 6. Stagger as details settle

When a card or panel expands, its details should cascade in reading order. Animate the container first, then stagger children by 20-40ms with opacity and a 4-8px translate.

**Test:** expand the card. Details arrive in sequence, and critical controls are usable within 300ms.

### 7. Arcs for object movement

Objects that travel across a surface should follow a slight arc, not a straight line. Offset the path perpendicular by 10-20% of the distance.

**Test:** the object feels placed or thrown rather than teleported. Skip the arc for distances under about 40px.

### 8. Pulse to confirm success

A single pulse is a clear, non-blocking acknowledgment. Scale 1 to 1.04 and back, or animate a ring once, over 200-300ms. Pair it with a check or a label.

**Test:** it pulses once and stops. No infinite pulse, no bounce, no pulse on errors.

### 9. Ease-in-out for moves

Elements already on screen should accelerate and decelerate. Use `ease-in-out` at 200-300ms for reposition, reorder, resize and in-place state changes.

**Test:** the move has weight at both ends instead of snapping out of one state into another.

### 10. Start popovers at 0.95

A popover should emerge from its trigger. Start at scale 0.95 with opacity 0 and animate to 1 over 120-180ms. Starting at 0.6 looks like it inflates from nothing.

**Test:** the popover reads as attached to the trigger, not as a new layer appearing from the centre.

### 11. Enter from the trigger

Spatial continuity is the whole trick. Set `transform-origin` to the trigger edge or corner, and animate scale and translation from that point.

**Test:** cover the trigger with your hand. You should still be able to tell where the popover came from.

### 12. Exit to the trigger

Closing should retrace the entrance. Reverse the transform toward the trigger, keep the same origin, and make the exit faster than the entrance.

**Test:** close the popover. It returns to the trigger instead of fading out in the middle of the screen.

### 13. No bounce on everyday UI

Overshoot implies physical play. On menus, forms, navigation and modals it feels toy-like and delays reading. Use critically damped springs or `ease-out`; reserve bounce for drag release, success and deliberate play.

**Test:** no routine action overshoots its target. If the whole product is bouncy, nothing feels special.

### 14. Don't tween while held

While the pointer is down, the control must track it 1:1. Any easing during a drag feels broken. Apply transforms directly from pointer delta, then hand off velocity to a spring on release.

**Test:** drag fast and reverse direction. The control stays under the pointer with no rubbery lag.

### 15. Make exits 20% shorter

Exits matter less than what comes next. Compute exit duration as 80% of enter, and simplify the exit to opacity plus a small transform instead of a full reverse path.

**Test:** closing feels decisive. Reopening never waits for the previous close to finish.

### 16. Overlap old & new states

A sequential exit-then-enter leaves a blank frame and a visible cut. Start the entering state before the exit finishes, overlap by 30-50%, and keep both states out of the layout flow while they cross.

**Test:** switch states frame by frame. There is never an empty moment.

### 17. Masks for reveals & colour

Reveal with `clip-path` or `mask-image` instead of animating height or background colour. Masks are GPU-friendly, avoid layout shift and can wipe colour across text or images.

**Test:** the reveal is smooth, does not reflow the page, and does not animate `height: auto`.

### 18. Crossfade icons + blur

A hard icon swap jars. Crossfade the two states in one slot over 150-200ms, with a small scale change and 0-2px blur at the midpoint.

**Test:** play to pause feels like one icon changing, not two icons trading places.

### 19. Rubber band past bounds

Hard stops feel broken. Past a boundary, apply diminishing resistance (for example 30% of the drag delta), then spring back to the bound on release.

**Test:** drag past the edge. It resists, communicates the limit, and returns smoothly without a snap.

### 20. Reuse the same layoutId

Shared element transitions make state changes legible. Keep the same identity across states: `layoutId` in Framer Motion or `view-transition-name` in the View Transitions API.

**Test:** move between list and detail. The image, title or card travels; two unrelated copies do not crossfade.

## Audit output format

When reviewing existing motion, respond in this shape:

```text
Motion score: 14/20 (Almost)

1. Rule 2: the dropdown runs 400ms.
   Fix: 180ms enter, 140ms exit.
2. Rule 11: the popover scales from the centre.
   Fix: set transform-origin at the trigger and start at 0.95.
...
```

- Lead with the score and verdict: `Floaty` (0-9), `Generic` (10-13), `Almost` (14-17), `Clean` (18-20).
- List violations by impact. One line of diagnosis, one line of fix.
- Fix spatial logic and timing before technique.
- Make the smallest change that passes the rule. Do not rebuild the interaction unless asked.

When adding motion to a new screen, apply the rules silently, then add a short note:

```text
Motion pass: 20/20. Ease-out entrances, 180ms menu, 30ms row stagger, press at 0.97, exits at 80%, reduced motion respected.
```

## Definition of done

- Score 18/20 or higher, with every critical item passing: rules 11, 14 and 15.
- Animations run on `transform`, `opacity` or masks, never on layout properties.
- Every animation can be interrupted, and rapid repeated input never queues or stutters.
- `prefers-reduced-motion` is handled by removing movement, not by hiding content.
- The motion still fits the brand. Not every product should move the same way.

## Guardrails

- Respect `prefers-reduced-motion`. Replace movement with a short fade or an instant change.
- Never animate `width`, `height`, `top`, `left`, `margin` or `padding` for UI transitions.
- Never block input, scroll or navigation while an animation plays.
- Avoid parallax, large zooms, rapid flashes, long spins and looping motion near text. Motion sickness is real.
- Do not slow an interaction down to show off the animation.
- Bounce is seasoning, not a default.
- Match platform conventions: a native-feeling control beats a clever one.
- When a rule conflicts with a deliberate art direction, say so and keep the safety and spatial rules.
