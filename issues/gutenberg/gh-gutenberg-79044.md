# #79044: Media Editor: Magnify the crop to fill the canvas

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Feature] Media`
- **Merged:** [`4b842bf`](https://github.com/WordPress/gutenberg/commit/4b842bf158c58e6ef064462edf820a7e5e1d6d3d)
- **Discussion:** [#79044](https://github.com/WordPress/gutenberg/pull/79044) · 17 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Media Editor now automatically magnifies the crop area to fill most of the canvas when the initial crop is small relative to the viewport. This prevents unusably tiny crops when editing large images in narrow windows by applying a presentational zoom that scales the entire scene without altering the actual exported image dimensions. The minimum crop size now adapts to the on-screen scale, keeping drag handles operable at smaller viewports.

## Impact

- **Plugin & theme developers / block authors:** No breaking changes or public API modifications. The cropper's internal rendering and interaction model changed, but the exported crop region and `cropRect` values remain identical.
- **Site owners / editors:** Improved usability when cropping large images in small browser windows. The crop overlay now scales up to fill the canvas, and drag handles remain grabbable without collapsing.
- **No action required.** Existing code relying on the Media Editor cropper will continue to function identically from a data and export perspective.

## Technical details

The diff introduces a presentational magnification layer in the Media Editor cropper. In `packages/media-editor/src/image-editor/core/camera.ts`, a new `getViewScale` function calculates a scale factor (capped by `MAX_VIEW_SCALE` and targeting `SETTLE_TARGET_CANVAS_FILL` at `0.8`) to scale the image and overlays so the crop fills the canvas. This scale is applied via CSS transforms in `packages/media-editor/src/image-editor/react/components/cropper.tsx` through `scaledVisualSize` and a prepended `scale(viewScale)` to the image matrix. During active resizing, the magnification is frozen via `frozenViewScale` to prevent the scene from jumping under the cursor. The minimum crop floor in `packages/media-editor/src/image-editor/core/stencil-math.ts` (`getMinCropPixels`) now incorporates `viewScale` into its `displayScale` calculation, ensuring the 24px source pixel floor and 44px screen pixel floor adapt correctly to the magnified view.

Before:
```ts
const displayScale = ( elementSize.width / naturalWidth ) * state.zoom;
```

After:
```ts
const displayScale = ( elementSize.width / naturalWidth ) * state.zoom * viewScale;
```

## Contribution

Opened by @ramonjd to address awkward thin/small crop creation in small viewports. @andrewserong provided early feedback, validating the approach while noting potential edge cases with very small images (smoothed rendering vs. nearest-neighbor, lack of pixel-snapping, and missing actual-size toggle). @ramonjd acknowledged the feedback, implemented nearest-neighbor rendering for zoomed states, and documented pixel-snapping and viewport zoom controls as future follow-ups rather than blocking changes. The PR merged after iterative refinement of the magnification constants and resize-handling behavior.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
