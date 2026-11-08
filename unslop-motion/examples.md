# Before / after

Concrete fixes for the most common motion problems.

## 1. Linear entrance (rule 1)

**Before**

```css
.modal { transition: all 400ms linear; }
```

**After**

```css
.modal {
  opacity: 0;
  transform: translateY(8px);
  transition:
    opacity 200ms cubic-bezier(0.16, 1, 0.3, 1),
    transform 200ms cubic-bezier(0.16, 1, 0.3, 1);
}
.modal.is-open { opacity: 1; transform: none; }
```

**Why.** Entrances should decelerate and settle. `linear` looks mechanical, and 400ms feels slow for a common surface.

## 2. Slow dropdown (rule 2)

**Before**

```css
.menu { transition: opacity 400ms ease, transform 400ms ease; }
```

**After**

```css
.menu {
  transition:
    opacity 180ms cubic-bezier(0.16, 1, 0.3, 1),
    transform 180ms cubic-bezier(0.16, 1, 0.3, 1);
}
.menu.is-closing {
  transition-duration: 140ms;
}
```

**Why.** Menus are opened constantly. 180ms feels instant but visible; 400ms feels laggy.

## 3. Stagger with no cap (rule 3)

**Before**

```js
row.style.transitionDelay = `${index * 30}ms`; // 50 rows = 1.5s
```

**After**

```js
row.style.transitionDelay = `${Math.min(index, 6) * 30}ms`;
```

**Why.** Stagger implies order, but the last row should not wait a second and a half.

## 4. Press scale too strong (rule 4)

**Before**

```css
.button:active { transform: scale(0.8); }
```

**After**

```css
.button {
  transition: transform 120ms cubic-bezier(0.16, 1, 0.3, 1);
}
.button:active { transform: scale(0.97); }
```

**Why.** 0.97 confirms the press. 0.8 looks like a toy and distorts the label.

## 5. Rigid knob (rule 5)

**Before**

```js
knob.style.transform = `translateX(${x}px)`;
```

**After**

```js
const stretch = Math.min(0.12, Math.abs(velocityX) * 0.0004);
knob.style.transform =
  `translateX(${x}px) scaleX(${1 + stretch}) scaleY(${1 - stretch * 0.5})`;
```

Reset both scales to 1 on release.

**Why.** A small stretch in the direction of travel makes the control feel physical without looking elastic.

## 6. Details arrive as a block (rule 6)

**Before**

```css
.card.is-open .detail { opacity: 1; }
```

**After**

```css
.card.is-open .detail {
  opacity: 1;
  transform: none;
  transition-delay: calc(var(--detail-index) * 30ms);
}
```

**Why.** A cascade in reading order explains the structure of the expanded card.

## 7. Straight-line move (rule 7)

**Before**

```css
@keyframes move {
  to { transform: translate(var(--dx), var(--dy)); }
}
```

**After**

```css
@keyframes move {
  50% {
    transform: translate(
      calc(var(--dx) / 2),
      calc(var(--dy) / 2 - 24px)
    );
  }
  to { transform: translate(var(--dx), var(--dy)); }
}
```

**Why.** A slight arc reads as placement. Keep it subtle and skip it for short distances.

## 8. Endless success pulse (rule 8)

**Before**

```css
.success { animation: pulse 1s infinite; }
```

**After**

```css
@keyframes confirm {
  0%   { transform: scale(1); }
  40%  { transform: scale(1.04); }
  100% { transform: scale(1); }
}
.success { animation: confirm 240ms cubic-bezier(0.16, 1, 0.3, 1) 1; }
```

**Why.** Confirmation should happen once and stop. Infinite motion becomes noise.

## 9. Move uses entrance easing (rule 9)

**Before**

```css
.item { transition: transform 240ms cubic-bezier(0.16, 1, 0.3, 1); }
```

**After**

```css
.item { transition: transform 240ms cubic-bezier(0.65, 0, 0.35, 1); }
```

**Why.** Entrances decelerate. Moves should accelerate and decelerate, so use `ease-in-out`.

## 10. Popover inflates from nothing (rule 10)

**Before**

```css
.popover { transform: scale(0.6); opacity: 0; }
.popover.is-open { transform: scale(1); opacity: 1; }
```

**After**

```css
.popover {
  transform: scale(0.95);
  opacity: 0;
  transform-origin: var(--origin-x) var(--origin-y);
}
.popover.is-open { transform: scale(1); opacity: 1; }
```

**Why.** 0.95 reads as emerging from the trigger. 0.6 looks like it is inflating.

## 11. Popover from the centre (rule 11)

**Before**

```css
.popover { transform-origin: center; }
```

**After**

```js
// placement: "bottom-start" or equivalent
popover.style.setProperty("--origin-x", "0%");
popover.style.setProperty("--origin-y", "0%");
```

**Why.** The surface should appear to come out of the button, not the middle of the screen.

## 12. Exit away from the trigger (rule 12)

**Before**

```css
.popover.is-closing { transform: scale(0.9); opacity: 0; }
```

**After**

```css
.popover.is-closing {
  transform: scale(0.95);
  transform-origin: var(--origin-x) var(--origin-y);
  opacity: 0;
}
```

**Why.** If it came from the trigger, it should go back to the trigger. Reversible motion completes the spatial story.

## 13. Bouncy menu (rule 13)

**Before**

```js
{ type: "spring", stiffness: 500, damping: 12 }
```

**After**

```js
{ type: "spring", stiffness: 400, damping: 40 }
```

Or use `ease-out` for everyday surfaces.

**Why.** Overshoot is playful. It belongs on drag release and success, not on a menu the user opens ten times a minute.

## 14. Tweening during a drag (rule 14)

**Before**

```js
knob.style.transition = "transform 150ms ease-out";
knob.style.transform = `translateX(${x}px)`;
```

**After**

```js
knob.style.transition = "none";
knob.style.transform = `translateX(${x}px)`;

// on release
knob.style.transition = "transform 220ms cubic-bezier(0.16, 1, 0.3, 1)";
knob.style.transform = `translateX(${snapTarget}px)`;
```

**Why.** While held, the control must track the pointer 1:1. Easing during a drag reads as lag.

## 15. Exit as long as the entrance (rule 15)

**Before**

```css
.menu { transition-duration: 200ms; }
```

**After**

```css
.menu { transition-duration: 200ms; }
.menu.is-closing { transition-duration: 160ms; } /* 80% */
```

**Why.** Exits are less important than the next action. Faster exits keep the interface responsive.

## 16. Blank frame between states (rule 16)

**Before**

```js
await fadeOut(oldView);
fadeIn(newView);
```

**After**

```js
fadeOut(oldView);   // starts
fadeIn(newView);    // starts before the exit finishes
```

Keep both layers absolutely positioned during the cross.

**Why.** Sequential transitions produce a visible empty frame. Overlap keeps continuity.

## 17. Animating height and colour (rule 17)

**Before**

```css
.panel {
  height: 0;
  background: transparent;
  transition: height 300ms ease, background 300ms ease;
}
```

**After**

```css
.panel {
  clip-path: inset(0 0 100% 0);
  transition: clip-path 300ms cubic-bezier(0.16, 1, 0.3, 1);
}
.panel.is-open { clip-path: inset(0 0 0 0); }
```

**Why.** Height and background animations cause layout and paint work. A clip-path reveal is cheaper and cleaner.

## 18. Hard icon swap (rule 18)

**Before**

```js
playIcon.hidden = true;
pauseIcon.hidden = false;
```

**After**

```css
.icon { transition: opacity 160ms ease, transform 160ms ease, filter 160ms ease; }
.icon.is-hidden { opacity: 0; transform: scale(0.8); filter: blur(2px); }
```

Keep both icons in the same grid slot and toggle a class.

**Why.** A crossfade with a small scale and blur reads as one icon changing instead of two icons trading places.

## 19. Hard clamp at the edge (rule 19)

**Before**

```js
x = Math.min(x, max);
```

**After**

```js
const over = Math.max(0, x - max);
x = max + (over * 100) / (over + 100); // diminishing resistance
```

On release, spring back to `max`.

**Why.** Resistance communicates the limit and keeps the gesture from feeling broken.

## 20. Two copies crossfading (rule 20)

**Before**

```jsx
<img className="thumb" src={thumb} />
<img className="hero" src={large} />
```

**After**

```jsx
<motion.img layoutId="cover" src={isOpen ? large : thumb} />
```

Or with the View Transitions API:

```css
.cover { view-transition-name: cover; }
```

**Why.** A shared element tells the user the same object is moving. Two crossfading copies look like a cut.

## Mini audit

**Input:** settings drawer that slides in from the left in 500ms with a bouncy spring, fades the backdrop in 500ms, and closes in 500ms.

```text
Motion score: 9/20 (Floaty)

1. Rule 13: the drawer overshoots on open.
   Fix: critically damped spring or 240ms ease-out.
2. Rule 2: 500ms is slow for a drawer used often.
   Fix: 280ms enter, 220ms exit.
3. Rule 15: exit matches the entrance.
   Fix: 80% of the enter duration.
4. Rule 12: closing fades in place instead of exiting toward the trigger.
   Fix: reverse the transform toward the settings button.
5. Rule 16: the backdrop and drawer do not overlap.
   Fix: start both together, with the drawer slightly ahead.
```

Fix order: safety and spatial logic, then timing, then feel.
