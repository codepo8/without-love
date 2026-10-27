# Without love — soften

[Open the entry](without-love.html)

One switch, no JavaScript. Press **Add love** and the page warms up: the background
goes from a cold slate to cream, the type softens, the strikethroughs retract and
each line lifts into place in reading order. Press it again and all of that runs
backwards.

The text is untouched. Every line still says *"Duty without love is irritation"* —
only the typesetting changes what that feels like.

## The one idea

There is a single number, `--love`, between `0` and `1`. Everything on the page is
a function of it. Nothing is toggled with a class per element, and no property is
written twice.

```css
@property --love {
  syntax: "<number>";
  inherits: true;
  initial-value: 0;
}
```

`@property` is the important part. A plain custom property is an unparsed token
stream, so the browser cannot interpolate it — it would jump from `0` to `1`.
Declaring the syntax as `<number>` makes it a real, animatable value, which means
a single `transition: --love 1.15s` animates the entire page at once.

The switch itself is a checkbox, and `:has()` connects it to the page without any
script:

```css
body:has(#love:checked) { --love: 1; }
```

## What each technique is doing

**`color-mix()` for the palette.** Every colour is a blend between a cold end and
a warm end, mixed by `--love`:

```css
--bg:  color-mix(in oklab, #fff4e4 calc(var(--love) * 100%), #0b1015);
--ink: color-mix(in oklab, #241611 calc(var(--love) * 100%), #93a1ad);
```

Mixing in `oklab` rather than `srgb` keeps the midpoint from going muddy — a
straight sRGB blend between near-black and cream passes through a grey that looks
washed out, while oklab travels through a cleaner brown. The percentage is attached
to the colour, and the second colour takes the remainder, which is why only one
percentage is needed.

**Fraunces' `SOFT` and `WONK` axes for the emotional weight.** Fraunces ships two
unusual axes: `SOFT` (0–100) rounds the terminals, and `WONK` (0–1) exaggerates the
awkward letterforms. Both are driven by the same number:

```css
font-variation-settings:
  "SOFT" calc(var(--love) * 100),
  "WONK" calc(1 - var(--love)),
  "wght" calc(820 - var(--love) * 470);
```

At `--love: 0` the harsh words are heavy, wonky and hard-edged. At `1` they are
soft, even and light. Because the axes are fed by an animatable number, they
interpolate smoothly instead of snapping between two static settings — this is the
part I most wanted to try, and it is the reason the piece is set in Fraunces.

**A strikethrough that retracts.** `text-decoration` cannot be transitioned, so the
struck-through word is a pseudo-element hairline whose `scaleX` is the inverse of
`--love`:

```css
.harsh::after {
  content: "";
  position: absolute; inset-inline: 0; top: 56%; height: .07em;
  background: currentColor;
  transform: scaleX(calc(1 - var(--love)));
  transform-origin: left center;
}
```

The word is also `skewX`-ed by up to 9 degrees at `0`, so it straightens as it is
loved.

**`sibling-index()` for the stagger.** Each line's delay is its position in the
list, with no classes and no inline styles:

```css
@supports (transition-delay: calc(sibling-index() * 1ms)) {
  li { transition-delay: calc(sibling-index() * 34ms); }
}
```

The `@supports` guard means browsers without `sibling-index()` (Safari and Firefox
at the time of writing) drop the rule and animate all thirteen lines together
instead of one after another — a graceful fallback rather than a broken layout.

**The heart is inline SVG**, not an emoji, so it renders identically everywhere:
an outline that is always visible, plus a filled copy whose opacity is `--love`.

## Accessibility

- The control is a real `<input type="checkbox">`, visually hidden but focusable,
  with a `<label>` as its target. It works with the keyboard, and the visible text
  is the accessible name.
- `:focus-visible` draws a ring on the label, not the invisible input.
- `prefers-reduced-motion: reduce` removes every transition and the line lift, so
  the two states still exist and still change instantly.
- Both ends of the palette were checked for contrast: the cold end is light grey on
  near-black, the warm end is dark brown on cream.
- Nothing depends on JavaScript, so the entry also works with scripting off.

## Browser support

The entry is written for current Chromium (where `@property`, `color-mix()` and
`sibling-index()` are all supported). Elsewhere it degrades in this order:

| Feature | Without support |
| --- | --- |
| `@property` | `--love` still applies, but the change is instant instead of animated |
| `color-mix()` | colours fall back to the declared defaults |
| `sibling-index()` | the stagger is dropped, the rest is unaffected |

The layout and both states survive every one of those cases.

## Verifying it

1. Open the file, press **Add love**. The palette, the type axes, the strikethroughs
   and the line order all move together, over 1.15 seconds.
2. Tab to the switch and press Space — same result, no mouse needed.
3. Turn on "reduce motion" in the OS and toggle again: the state changes, nothing
   animates.
