# Unslop Motion checklist

Twenty pass/fail checks. Ten seconds per phase. Score one point per rule.

Before scoring, confirm that `prefers-reduced-motion` is handled and that input is never blocked.

## Phase 1: Timing (10s)

- [ ] **1. Entrances:** ease-out, 150-300ms, short travel.
- [ ] **2. Dropdowns:** 150-250ms enter, exits shorter.
- [ ] **3. List stagger:** about 30ms per row, total delay capped.
- [ ] **9. Moves:** ease-in-out, 200-300ms.
- [ ] **15. Exits:** 20% shorter than entrances.

## Phase 2: Space (10s)

- [ ] **7. Arcs:** moving objects follow a slight curve, not a straight line.
- [ ] **10. Popovers:** start at scale 0.95, not 0.6.
- [ ] **11. Enter:** transform-origin sits at the trigger.
- [ ] **12. Exit:** returns to the trigger.
- [ ] **16. Overlap:** new state enters before old state leaves.
- [ ] **20. Shared elements:** same `layoutId` or view-transition name.

## Phase 3: Feel (10s)

- [ ] **4. Press:** scales to 0.97 and resets cleanly.
- [ ] **5. Knobs:** stretch up to 12% while moving, rest at 1.
- [ ] **8. Success:** one pulse, then stop.
- [ ] **13. Bounce:** none on everyday UI.
- [ ] **14. Drag:** 1:1 while held, no easing during the drag.
- [ ] **19. Bounds:** rubber band past limits, spring back on release.

## Phase 4: Reveal (10s)

- [ ] **6. Details:** children cascade in reading order.
- [ ] **17. Masks:** clip-path or mask reveals, no height or colour tween.
- [ ] **18. Icons:** crossfade with a small scale and blur.

## Motion budget

| Interaction | Duration |
|---|---|
| Press | 80-120ms |
| Tooltip | 100-150ms |
| Icon swap | 120-180ms |
| Dropdown | 150-250ms |
| Modal or sheet | 200-350ms |
| Page transition | 250-450ms |
| Stagger | 20-40ms per item |
| Exit | 80% of enter |

## Verdict

| Score | Verdict | Action |
|---|---|---|
| 0-9 | Floaty | Fix safety and spatial logic first |
| 10-13 | Generic | Fix timing and origin |
| 14-17 | Almost | Close the feel and reveal gaps |
| 18-20 | Clean | Ship |

**Minimum to ship: 18/20. Critical rules: 11, 14 and 15.**

## Three questions that catch most bad motion

1. Does this explain where something came from, what changed, or what to do next?
2. Does it delay the next action or block input?
3. Would the interaction still make sense without it?
