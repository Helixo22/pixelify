# Pixel drawings

## A drawing

```ts
import { PixelBuffer, shapes } from "pleate/effects/pixel-art";
import { PIXEL_INK, pixelRamps as r } from "pleate/effects/pixel-gallery";

const { ellipse, polygon, rect, taper, and, or, not } = shapes;

export const kite = {
  size: 32,
  draw: (p: PixelBuffer) => {
    p.shade(polygon([[16, 2], [27, 13], [16, 26], [5, 13]]), r.red); // three tones, lit from the top left
    p.shade(polygon([[16, 2], [27, 13], [16, 13]]), r.yellow);
    p.line(16, 26, 12, 31, r.brown[2]);
    p.outline(PIXEL_INK); // always last
  },
};

<PixelArt art={kite} label="A kite" className="size-32" />
```

1. Build the silhouette from large shapes. Two or three are enough.
2. `shade` each one with a ramp. `dither` 0.5 is the default, 0.9 for round and soft things. `bias` above 0 pushes a part
   into the shade: use it on the parts behind.
3. A few details by hand: a highlight of 3 to 5 pixels at the top left, one pattern, one spot of a second color.
4. `outline` once at the end. Steam, light and water come after it, without a contour.

Sizes: 32 for icons and lists, 48 to 64 for a feature, 96 for a scene. Show 32 px drawings at 64, 96 or 128 CSS pixels.
`colors={{ "#e5402f": "var(--pl-accent)" }}` ties a tone to the theme.

## A scene

`PixelCanvas` redraws a 96 by 96 picture for each point of the scroll:

```tsx
<ScrollScene length={4} label="How it grows" stage={
  <div className="grid size-full place-items-center pb-48">
    <PixelCanvas draw={draw} animate label="A plant growing" className="h-auto w-[min(26rem,80vw,52dvh)]" />
  </div>
}>...</ScrollScene>

function draw(p: PixelBuffer, { progress, time }: PixelFrameState) {
  p.shade(ellipse(48, 46, 41, 41), DISC); // the disc goes first
  const grown = range(progress, 0.1, 0.6);
  // shapes whose size and place depend on `grown`
  p.outline(PIXEL_INK);
}
```

Round positions of moving things so they step from pixel to pixel. Use `time` for steam, bubbles and sway only, at 12
frames per second.

## Around the page

- `PixelFrame` around a pleate `Button` or a picture: stepped corners and a hard shadow. Main actions only.
- `PixelSteps` inside a `ScrollScene` shows the step. Give it the step names, screen readers read them.
