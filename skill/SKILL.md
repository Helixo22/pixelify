---
name: pixelify
description: Builds websites in a pixel and voxel style with pleate. Voxel 3D objects drawn as large pixels that assemble, come apart and burst as the page scrolls, plus 2D pixel art with dithering, all made in code with no image or model files. Use when the user types /pixelify <description>, asks for a pixel art, voxel, 8 bit or blocky 3D site, or picks the pixelify style when asked how a new site should look.
argument-hint: "<what the site is for>"
---

# /pixelify

Site: `$ARGUMENTS`

A pixelify site is a normal, readable site with one extraordinary thing in it: the product itself, built from voxels or
pixels, that changes as the reader scrolls. Text, navigation, buttons and forms stay calm and come from pleate. The
picture carries the color.

## Stack

- Next.js App Router, React 19, Tailwind CSS v4, TypeScript.
- UI and engine: [pleate](https://github.com/Helixo22/pleate). The voxel and pixel modules are part of it, under
  `pleate/effects/`. Never write another engine, and never use a component pleate already has.
- `three` (0.187 or later) for the voxel modules.
- Backend, only if the site needs one: `pleate-backend` and `pleate-client` from the same repository.

| Import | What it gives |
| --- | --- |
| `pleate/effects/voxel-scene` | `VoxelScene`, `VoxelView`, `Volume`, `solids`, `volumeFromPixels`, `voxelModel`, `loadVox`, `decodeVolume` |
| `pleate/effects/voxel-gallery` | 12 models in parts, and `voxelDrawing(id)` |
| `pleate/effects/pixel-art` | `PixelArt`, `PixelFrame` (server safe), `PixelCanvas`, `PixelSteps`, `PixelBuffer`, `shapes` |
| `pleate/effects/pixel-gallery` | 40 drawings, `pixelRamps`, `PIXEL_INK` |
| `pleate/effects/scroll-scene` | `ScrollScene`, `ScrollBeat`, `createTimeline`, `range`, `useSceneProgress` |

Details: [references/voxel.md](references/voxel.md) to model and animate, [references/pixel.md](references/pixel.md) to
draw, [references/page.md](references/page.md) for the page around them. Read the ones you need before writing code.

## 1. Ask first, once

Ask everything in one round, in the user's language, with a proposed answer for each. Skip what the user already said.
Do not write code before the answer.

1. What the site is for, its pages and real texts (name, prices, hours, contacts).
2. The object of the page: what should be built as the reader scrolls. Propose one concrete idea with 3 to 6 steps.
3. Technique: voxel 3D (default), pixel art 2D (lighter, good for small sites), or both.
4. Brand color, and light, dark or both.
5. Language of the site, and where it will be published.
6. Backend functions, if any.

## 2. Set up

pleate is not on npm yet:

```bash
git clone --depth 1 https://github.com/Helixo22/pleate.git <temporary folder>/pleate
cd <temporary folder>/pleate && npm install && npm run build:lib
cd packages/pleate && npm pack
```

Copy the tarball to `vendor/` in the project and run `npm install ./vendor/pleate-<version>.tgz three` and
`npm install -D @types/three`. Then, as in the pleate README:

```css
/* app/globals.css */
@import "tailwindcss";
@import "pleate/styles.css";
@source "../node_modules/pleate/dist";
```

`app/layout.tsx` gets `ThemeScript` in the head and `ThemeProvider` around the body, with `suppressHydrationWarning` on
`<html>`. Load the pixel font with `next/font/google` (`Pixelify_Sans`, variable `--font-pixel`).

Read the `README.md` of pleate and the demos in `apps/docs/demos` for real props. The complete example is
`apps/docs/app/examples/voxel3d`. Never invent a prop: if it is not in the source, it does not exist.

## 3. Plan the scene

Write it down before coding, as a short table the user can read:

| Progress | What the reader sees | Text |
| --- | --- | --- |
| 0 to 0.25 | The dough drops on the table and flattens | "The dough" + one sentence |
| ... | ... | ... |

- One object, one story. Each step changes something the reader can name.
- Steps follow the real process of the product (how it is made, what is inside, how it arrives).
- Every scroll position has one picture. Scrolling back plays it backwards.
- One surprise at most: a burst when things meet, a lid that opens, a light that turns on.

## 4. Build

1. **Model the object.** Start from the gallery if something fits. Otherwise write the parts with `Volume` and `solids`,
   each part its own `VoxelPart`, 24 to 40 voxels on the longest side. Three tones per material, from `pixelRamps`.
2. **Look at it.** Render it, take a screenshot from two angles, fix it. Do this for every model and drawing: never ship
   a picture you have not seen. If it is not recognizable without its label, model it again.
3. **Hero.** The finished object, on a disc in the accent color, slowly turning.
4. **Scroll scene.** `ScrollScene` with `VoxelScene` as the stage, `ScrollBeat` for the texts, `PixelSteps` for the
   position.
5. **The rest of the page** with pleate components. `PixelFrame` goes around the main buttons only.

## 5. Rules of the style

- Headings in the pixel font, **words only**. Prices, hours, phone numbers and dates stay in the text font: in pixel
  fonts a 5 reads like an S and a 7 like a 9.
- Body text is never in a pixel font.
- One accent color. The disc, the main button and the current step use it. Everything else is the neutral theme.
- Contour `#2a1720` on objects, never pure black. Light from the top left everywhere.
- One size of pixel per scene, 3 to 5 CSS pixels. Never scale a pixel drawing to a size that is not a whole multiple of
  it when it has to look sharp, and never rotate a 2D drawing.
- No scanlines, no CRT glow, no neon, no gradients behind the text, no emoji.
- No navigable rooms and no free flying camera: the reader scrolls, the scene answers.
- Make original objects. Do not copy characters, logos or the look of an existing game, film or brand, and do not model
  real people, unless the user owns it and gives the material.

## 6. Check before finishing

- `npm run build` and `npm run lint` pass.
- Screenshots at 1440 and 390 px wide, light and dark, at the top and at every step of the scene. No text over the
  object, no sideways scroll.
- Reduced motion: the scene still follows the scroll, nothing moves by itself.
- Every scene has a `label` and a `fallback`. Keyboard reaches every link and button, focus is visible.
- The page reads and works with the canvas missing.
- Weight: a scene adds no images and no model files. Load `VoxelScene` with `next/dynamic` when it is below the fold.

If the user has other instructions for UI work (anti slop rules, accessibility, writing), apply them on top of this.
