# The page around the picture

## Order that works

1. **Navbar** (pleate `Navbar`): the name in the pixel font, two to four links. Quiet.
2. **Hero**: one sentence that says what the place or product is, one line of facts, two actions at most. The finished
   object beside it on wide screens, below it on phones.
3. **The scroll scene**: 3 to 6 steps.
4. **What people came for**: menu, prices, rooms, plans. A pleate list or a plain grid with dashed rules, one `PixelArt`
   per item. Not cards inside cards.
5. **Hours, place, contacts**, with a real open or closed state when the hours are known.
6. **Footer**.

## Layout

- Wide: text in a column of 24 rem at most, picture on the other side. Never text on top of the object.
- Phone: picture in the upper 60 percent of the screen, text under it, steps at the bottom.
- The stage of a `ScrollScene` fills the screen. Use `offset` when the navbar covers the top.
- Use one flat band of accent color on the page, not more.

## Color

Start from a pleate palette or `setCustomColor(brand)`. A page can be pinned with `data-palette="orange"` on its root.
The gallery ramps work on every palette. If the accent is close to the color of the object, change the disc
(`disc={{ color: "var(--pl-fg)" }}`) so the silhouette stays clear.

## Type

```tsx
const PIXEL = "font-(family-name:--font-pixel)";
<h1 className={`${PIXEL} text-[clamp(2.75rem,6vw,5rem)] leading-none text-balance`}>...</h1>
```

Display sizes only, from 24 px up. Sentence case. Numbers in the text font, `tabular-nums`.

## Performance

- Below the fold: `const Scene = dynamic(() => import("./scene"), { ssr: false })` with a placeholder of the same height.
- Keep `loop="demand"` (the default) for anything driven by the scroll.
- One large scene and at most a few `VoxelView`. Scenes off screen do not draw.
- `PixelArt` renders on the server: use it for everything that does not move.
