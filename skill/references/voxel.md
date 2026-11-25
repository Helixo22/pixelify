# Voxel models and scenes

## A model

A model is a list of parts. Parts are what comes apart, lands, opens or turns.

```ts
import { pixelRamps as r } from "pleate/effects/pixel-gallery";
import { solids, Volume, type VoxelModel } from "pleate/effects/voxel-scene";

const { box, cylinder, ellipsoid, and, not } = solids;

export const lantern: VoxelModel = {
  id: "lantern",
  name: "Lantern",
  parts: [
    // `at` is the bottom center of the part, in voxels from the bottom center of the model.
    { id: "base", volume: () => new Volume(20, 4, 20).fill(cylinder(9.5, 9.5, 0, 4, 9.6, 8), r.dark[1]) },
    {
      id: "glass",
      at: [0, 4, 0],
      away: [0, 8, 0], // where it goes when the model comes apart
      volume: () => new Volume(16, 16, 16)
        .fill(cylinder(7.5, 7.5, 0, 16, 6.8), r.yellow[0])
        .paint((x, _y, z) => x === 7 || z === 7, r.dark[2]), // only recolors what is there
    },
    { id: "roof", at: [0, 20, 0], away: [0, 20, 0], volume: () => new Volume(22, 8, 22).fill(cylinder(10.5, 10.5, 0, 8, 10.6, 2), r.red[1]) },
  ],
};
```

- Axes: x to the right, y up, z toward the viewer. A part is centered on x and z and stands on y = 0.
- `fill` adds, `carve` removes, `paint` recolors, `speckle(color, density, solid, seed)` scatters a second tone. A solid is
  any function `(x, y, z) => boolean`, so waves, leaves and roofs are one line of math.
- The center of a volume of even size n is `(n - 1) / 2`.
- Materials: base color, then `speckle` with the light or dark tone at 0.15 to 0.35. Smooth things (metal, porcelain) get
  no speckle.
- Keep a model under 24,000 triangles. Speckle costs triangles: fewer speckles on large parts.
- Colors are any CSS color. `var(--pl-accent)` follows the theme.

Other sources: `volumeFromPixels(drawing, depth, rim)`, `voxelDrawing("gem", 6)`, `await loadVox("/model.vox")` for
MagicaVoxel, `decodeVolume(text)` for a saved model (make the text with `encodeVolume`).

## One model

```tsx
<div className="h-96">
  <VoxelView model={lantern} build spin={4} explode={open ? 1 : 0} disc label="A paper lantern" />
</div>
```

The parent needs a height. `pose={(actor, { time }) => { actor.parts.roof.rotation.y = time }}` moves parts by hand.
At most 8 WebGL canvases on a page: for grids of small pictures use `PixelArt`.

## A scene on the scroll

```tsx
"use client";

import { range, ScrollBeat, ScrollScene } from "pleate/effects/scroll-scene";
import { VoxelScene, type VoxelContext } from "pleate/effects/voxel-scene";

function setup(ctx: VoxelContext) {
  const { camera, disc } = ctx;
  const object = ctx.add(lantern, 2.4); // the longest side becomes 2.4 units
  object.group.position.y = -1.2; // models stand on their origin
  const sparks = ctx.burst({ count: 30, colors: ["#ffe27a", "#f7b733"], cube: 0.08, spread: 1.6 });
  return {
    frame({ progress }: { progress: number }) {
      object.explode(1 - range(progress, 0.1, 0.6));
      sparks.set((progress - 0.6) / 0.25);
      object.group.rotation.y = -0.5 + progress * 2;
      // Tall screens put the picture above the text, wide ones beside it.
      const tall = camera.aspect < 1;
      disc.center = tall ? [0.5, 0.36] : [0.68, 0.5];
      disc.radius = tall ? 0.42 : 0.38;
      camera.position.set(0, 1.8, tall ? 14 : 9);
      camera.lookAt(0, 0.1, 0);
    },
  };
}

<ScrollScene length={4} label="How the lantern is made" stage={<VoxelScene setup={setup} disc label="A lantern assembling" />}>
  <ScrollBeat from={0} to={0.33} className="items-end pb-28 md:items-center md:pb-12">...</ScrollBeat>
</ScrollScene>
```

- `disc.center` moves the models and the disc together. Do not call `camera.setViewOffset`: the scene uses it.
- `frame` must give the same picture for the same progress. No random numbers, no state that piles up. The clock
  (`time`, with `loop="always"`) is only for waves, smoke and idle turning.
- A landing: `position.y = rest + (1 - t * t) * 34` on a part, hidden while `t === 0`. Read `rest` once in setup.
- `actor.build(t)` raises the model from the ground voxel by voxel.
- `burst.set(t)` runs from 0 to 1. Put it right after two things meet.
- `createTimeline` blends many values between named points, for camera moves.
- Texts sit on the empty side. On a 1440 px wide page the disc keeps to the right half.

## The look

| Prop | Default | When to change it |
| --- | --- | --- |
| `pixel` | 4 | 3 for small views, 5 for a chunkier look. The whole page uses the same one |
| `bands` | 4 | 3 for flatter, 5 for rounder |
| `dither` | 0.55 | 0 for clean bands |
| `grain` | 0.3 | 0 for smooth products, 0.6 for food and stone |
| `outline` | `#2a1720` | A dark tone of the brand color also works. `false` removes it |
