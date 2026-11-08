# The 20 rules: expanded reference

Use this file when the quick pass is not enough: edge cases, implementation notes and acceptance tests.

## Working with motion tokens

Define durations and easings once, then use them everywhere.

```css
:root {
  --motion-press: 100ms;
  --motion-fast: 150ms;
  --motion-base: 200ms;
  --motion-slow: 300ms;

  --ease-out: cubic-bezier(0.16, 1, 0.3, 1);
  --ease-in: cubic-bezier(0.7, 0, 0.84, 0);
  --ease-in-out: cubic-bezier(0.65, 0, 0.35, 1);
}
```

Animate `transform`, `opacity`, `clip-path` and `filter`. Do not animate layout properties.

---

## 1. Ease-out for entrances

**Why.** An element arriving on screen should decelerate, as if it has momentum and settles. `ease-in` makes it look like it is being pulled into the page; `linear` looks mechanical.

**Do**
- Use `cubic-bezier(0.16, 1, 0.3, 1)` or `(0.22, 1, 0.36, 1)`.
- 150-300ms depending on travel distance.
- Pair opacity with 4-12px of translate or scale from 0.96.
- Keep the travel short; long distances make the interface feel slow.

```css
.panel {
  opacity: 0;
  transform: translateY(8px);
  transition:
    opacity 180ms var(--ease-out),
    transform 180ms var(--ease-out);
}
.panel.is-open {
  opacity: 1;
  transform: none;
}
```

**Don't**
- Use `ease-in` or `linear` for entrances.
- Translate 40-100px just to show movement.
- Run 500ms on a control the user sees every day.

**Test.** The element settles into its final position. It does not stop dead, overshoot or speed up.

---

## 2. Dropdowns at 150-250ms

**Why.** Menus are opened constantly. Latency is felt more here than anywhere else, but a 100ms animation is invisible enough to feel like a teleport.

**Do**
- 180ms is a safe default. Use 160-200ms for enter.
- Exit at 120-150ms.
- Animate opacity and a 4-8px translate or 0.98 scale.
- Interrupt correctly: reopening while closing should continue from the current state, not restart.

**Don't**
- 300-400ms menus.
- Stagger every menu item. The menu itself is the animation.
- Animate height or `display` with a transition.

**Test.** Open and close ten times. It feels immediate but visible, and rapid clicking never queues.

---

## 3. Stagger list rows by 30ms

**Why.** A small delay between rows suggests order and causality. The list reads as a cascade instead of a block appearing at once.

**Do**
- 20-40ms per row. 30ms is the default.
- Cap the total delay. With many rows, use `min(index, 6) * 30ms`.
- Start from the top, or from the interaction point when the user clicked a specific row.
- Animate opacity and 4-8px of translate.

```css
.row {
  opacity: 0;
  transform: translateY(6px);
  transition:
    opacity 180ms var(--ease-out),
    transform 180ms var(--ease-out);
  transition-delay: calc(var(--row-index, 0) * 30ms);
}
```

**Don't**
- Stagger by 100ms. Ten rows take a second.
- Stagger 50 rows without a cap.
- Stagger every child element in a complex card.

**Test.** The last row appears within roughly 300ms of the first. The list never feels like it is loading in sequence forever.

---

## 4. Scale to 0.97 on press

**Why.** A small scale change confirms that the press registered without moving layout or changing colour. It is the cheapest tactile feedback there is.

**Do**
- Scale to 0.97, never below about 0.95.
- 80-120ms down, 120-180ms back.
- Apply to the interactive surface, not the whole card when the card contains text.
- Reset on pointer cancel, pointer leave and blur.

```css
.button {
  transition: transform 120ms var(--ease-out);
}
.button:active {
  transform: scale(0.97);
}
```

For draggable or long-press controls, drive the scale from pointer events so a drag-off resets it.

**Don't**
- Scale to 0.8 or 0.9. It looks like a toy.
- Animate width, padding or font size.
- Leave the button stuck at the pressed size.

**Test.** Press and hold, drag off, release. The control returns cleanly every time.

---

## 5. Stretch moving knobs 12%

**Why.** Squash and stretch is how physical objects communicate speed and mass. A slider handle that stretches slightly in the direction of travel feels alive; one that stays rigid feels like a static rectangle.

**Do**
- Stretch up to 12% along the axis of motion while dragging or flicking.
- Shrink the perpendicular axis by roughly half the stretch.
- Base the amount on velocity and clamp it.
- Reset to 1 on release, before the settle animation.

```js
const stretch = Math.min(0.12, Math.abs(velocityX) * 0.0004);
knob.style.transform =
  `scaleX(${1 + stretch}) scaleY(${1 - stretch * 0.5})`;
```

**Don't**
- Stretch text or icons.
- Exceed about 15%.
- Leave a knob stretched at rest.
- Stretch on a slow drag where there is no velocity.

**Test.** Flick the knob. It stretches in the direction of travel, settles, and looks perfectly round at rest.

---

## 6. Stagger as details settle

**Why.** When a card expands, the details inside carry the structure. Arriving in reading order makes the layout easier to parse than a single block fade.

**Do**
- Animate the container first, 150-250ms.
- Start the children after 40-60% of the container animation.
- Stagger 20-40ms in reading order.
- Animate opacity plus 4-8px of translate.
- Keep the primary controls available within 300ms.

**Don't**
- Animate every nested element, including dividers and icons.
- Delay the main action behind the cascade.
- Stagger in a random or reverse order.

**Test.** Expand the card. Details cascade in the order a person would read them.

---

## 7. Arcs for object movement

**Why.** Objects in the real world rarely move in perfectly straight lines. A slight arc reads as natural placement and makes a transition feel authored rather than mechanical.

**Do**
- Offset the path perpendicular to the straight line by 10-20% of the distance.
- Use `offset-path` with a quadratic curve, or a midpoint keyframe.
- Keep the arc subtle. It should be felt more than seen.

```css
@keyframes place {
  from { transform: translate(0, 0); }
  50% { transform: translate(calc(var(--dx) / 2), calc(var(--dy) / 2 - 24px)); }
  to { transform: translate(var(--dx), var(--dy)); }
}
```

**Don't**
- Use large semicircles for UI controls.
- Arc short distances under about 40px.
- Arc text or icons that should feel anchored.

**Test.** Follow the object with your eye. It feels placed or thrown, not teleported.

---

## 8. Pulse to confirm success

**Why.** A single pulse is a clear acknowledgment that does not block the next action or steal focus the way a modal does.

**Do**
- Scale 1 to 1.04 and back, or animate a ring once.
- 200-300ms total.
- Pair with a check icon or a short label.
- Use the success colour, not the accent colour.

**Don't**
- Pulse forever.
- Bounce the element.
- Pulse on errors or warnings.
- Pulse every row in a list.

**Test.** It confirms once and stops. The interface is stable immediately afterward.

---

## 9. Ease-in-out for moves

**Why.** An element already on screen has to accelerate from rest and decelerate into a new position. Symmetric easing reads as a deliberate move with weight at both ends.

**Do**
- Use `cubic-bezier(0.65, 0, 0.35, 1)` at 200-300ms.
- Use it for reorder, reposition, resize and in-place state changes.
- Keep the path short; long moves need more time, not a slower curve.

**Don't**
- Use entrance easing for moves.
- Use linear for anything the user is watching.
- Chain several moves that could happen together.

**Test.** The move starts and stops with weight. It does not snap out of one state and into another.

---

## 10. Start popovers at 0.95

**Why.** A popover that starts at 0.6 looks like it inflates from nothing. Starting at 0.95 keeps the scale change subtle and reads as the surface emerging from the trigger.

**Do**
- Scale 0.95 to 1, opacity 0 to 1.
- 120-180ms.
- Set the origin at the trigger (see rule 11).
- Use a small translate if it helps direction.

**Don't**
- Scale from 0.
- Animate width or height.
- Combine scale, rotation and a long translate in the same entrance.

**Test.** The popover feels attached to the trigger and arrives without inflating.

---

## 11. Enter from the trigger

**Why.** Spatial continuity tells the user where the new surface came from. That reduces the cognitive cost of a new layer appearing on screen.

**Do**
- Compute `transform-origin` from the trigger and the popover placement.
- For a menu below a button, origin at the top edge; above, bottom edge.
- Animate scale and translate from that origin.
- Use anchor positioning, floating-ui or an equivalent placement system.

```css
.popover {
  transform-origin: var(--origin-x) var(--origin-y);
}
```

**Don't**
- Use centre origin for anchored surfaces.
- Fade in from the middle of the screen.
- Animate from the top-left of the viewport.

**Test.** Cover the trigger. You can still tell where the surface came from.

---

## 12. Exit to the trigger

**Why.** Reversible motion completes the story. If something came from the trigger, it should return there; otherwise the user has to re-learn the spatial model.

**Do**
- Reverse the transform toward the trigger.
- Keep the same origin.
- Make the exit faster than the entrance (see rule 15).

**Don't**
- Fade out in place when the entrance came from a trigger.
- Exit toward the screen centre.
- Use a different curve family from the entrance.

**Test.** Close the surface. It retraces the entrance, and the trigger looks like the source and destination.

---

## 13. No bounce on everyday UI

**Why.** Overshoot means play. On menus, forms, navigation and modals it feels toy-like, delays reading and makes the product less trustworthy. Bounce works when it is rare.

**Do**
- Use critically damped springs or `ease-out`.
- Reserve bounce for drag release, success confirmation and deliberate playful moments.
- Keep one bouncy moment per flow at most.

**Don't**
- Bounce dropdowns, modals, page transitions or form validation.
- Use the same spring for everything.
- Let bounce push content past its final position.

**Test.** No routine action overshoots. If everything bounces, nothing feels special.

---

## 14. Don't tween while held

**Why.** During a drag, the control is an extension of the pointer. Any easing introduces lag, makes the interface feel broken and breaks the user's sense of direct manipulation.

**Do**
- Set the transform directly from pointer delta while the pointer is down.
- Disable transitions for the duration of the drag.
- Capture the pointer so the drag continues outside the element.
- On release, hand off velocity to a spring or an ease-out settle.

```js
function onPointerMove(event) {
  knob.style.transition = "none";
  knob.style.transform = `translateX(${event.clientX - startX}px)`;
}

function onPointerUp() {
  knob.style.transition = "transform 220ms var(--ease-out)";
  knob.style.transform = `translateX(${snapToNearest()}px)`;
}
```

**Don't**
- Transition `transform` during the drag.
- Animate toward a target while the finger is still down.
- Reset to the start position on release unless the gesture was cancelled.

**Test.** Drag fast and reverse direction. The control stays under the pointer with no rubbery lag.

---

## 15. Make exits 20% shorter

**Why.** The exit is not the point; the next action is. A faster exit keeps the interface responsive and makes closing feel decisive.

**Do**
- Exit duration = 0.8 × enter duration.
- Simplify the exit: opacity plus a small scale or translate is usually enough.
- Skip the full reverse path on complex transitions.

**Don't**
- Use the same duration for enter and exit.
- Make exits longer than entrances.
- Block the next action until the exit completes.

**Test.** Close and immediately reopen. Closing feels decisive and reopening never waits.

---

## 16. Overlap old & new states

**Why.** A sequential exit-then-enter produces a blank frame and a visible cut. Overlapping keeps continuity and hides the swap.

**Do**
- Start the entering state before the exit finishes.
- Overlap by 30-50% of the total transition.
- Keep both states absolutely positioned during the cross so layout does not jump.
- Use shared elements when the same object appears in both states.

**Don't**
- Wait for the exit to finish before entering.
- Stack both states in normal flow.
- Crossfade two different objects as if they were the same.

**Test.** Step through the transition. There is never an empty frame or a layout jump.

---

## 17. Masks for reveals & colour

**Why.** `clip-path` and `mask-image` are GPU-friendly, avoid layout reflow and can reveal content or wipe colour in a controlled shape. Animating height or background colour is slower and harder to keep smooth.

**Do**
- Reveal with `clip-path: inset(...)`.
- Use a moving gradient as a mask for soft reveals.
- Animate a CSS custom property that feeds the mask.
- For text colour, overlay a coloured copy and reveal it with a mask.

```css
.reveal {
  clip-path: inset(0 100% 0 0);
  transition: clip-path 300ms var(--ease-out);
}
.reveal.is-in {
  clip-path: inset(0);
}
```

**Don't**
- Animate `height: auto`.
- Tween `background-color` across large surfaces.
- Use a mask when a simple opacity fade is enough.

**Test.** The reveal is smooth, avoids layout shift and does not animate layout properties.

---

## 18. Crossfade icons + blur

**Why.** A hard swap between play and pause, or between chevrons, is jarring because the shape changes instantly. A crossfade with a slight scale and blur hides the swap and reads as one icon changing.

**Do**
- Overlap the two icons in one slot.
- 150-200ms.
- Crossfade opacity, scale 0.8 to 1 and 1 to 0.8, blur 0 to 2px at the midpoint.
- Keep the hit target stable.

**Don't**
- Show both icons at full opacity.
- Rotate 180 degrees when a simple swap is enough.
- Bounce the icon.

**Test.** Toggle play and pause. It feels like one icon changing, not two icons trading places.

---

## 19. Rubber band past bounds

**Why.** A hard stop feels broken. Resistance communicates the limit while preserving the user's intent, and the spring back confirms where the boundary is.

**Do**
- Past the bound, apply diminishing resistance, for example 30% of the drag delta.
- Use a formula such as `bound + (over * 100) / (over + 100)` for a natural curve.
- Spring back to the bound over 200-300ms.
- Use it for sheets, carousels, pull-to-refresh and draggable controls.

**Don't**
- Allow unlimited drag with no resistance.
- Snap back linearly.
- Rubber band inside native scroll areas where the platform already handles it.

**Test.** Drag past the edge. It resists, communicates the limit and returns smoothly without a hard snap.

---

## 20. Reuse the same layoutId

**Why.** A shared element transition tells the user that the same object is moving, not that one object disappeared and another appeared. It is the difference between a cut and a camera move.

**Do**
- Keep the same identity across states: `layoutId` in Framer Motion, `view-transition-name` in the View Transitions API.
- Keep the DOM structure stable enough for the transition to match.
- Animate transform and opacity, not layout.

```jsx
{isOpen ? (
  <motion.img layoutId="hero" src={large} />
) : (
  <motion.img layoutId="hero" src={thumb} />
)}
```

```css
.card { view-transition-name: card-1; }
```

**Don't**
- Duplicate the element with different IDs and crossfade.
- Use a shared element for unrelated content.
- Animate layout properties to force the match.

**Test.** Move between list and detail. The image, title or card travels. Two unrelated copies do not crossfade.
