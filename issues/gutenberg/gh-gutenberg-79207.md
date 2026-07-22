# #79207: Media editor modal: Fix keyboard resizing for locked aspect-ratio crops

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Feature] Media`
- **Merged:** [`7d8e021`](https://github.com/WordPress/gutenberg/commit/7d8e02165f201f251835683e73aed07c600ce230)
- **Discussion:** [#79207](https://github.com/WordPress/gutenberg/pull/79207) · 5 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

Fixes a bug in the media editor modal where crop handles would fail to resize when using keyboard arrow keys with a locked aspect ratio. Previously, the resize logic calculated the driving axis based on resulting crop dimensions, which caused single-axis keyboard steps to be cancelled out. The fix explicitly passes the input axis to the resize calculation, ensuring arrow keys correctly expand or shrink the crop area while preserving the selected ratio.

## Impact

- **Plugin & theme developers:** No action required. This is an internal media editor behavior fix.
- **Site owners & editors:** Resolves a usability issue where keyboard navigation in the image cropper appeared unresponsive when an aspect ratio was locked.
- **Headless & REST consumers:** No impact. The change is confined to the Gutenberg media editor UI.
- **No breaking changes or deprecations.**

## Technical details

The diff modifies `packages/media-editor/src/image-editor/core/stencil-math.ts` and `packages/media-editor/src/image-editor/react/components/stencils/rectangle-stencil.tsx`. It introduces a new `ResizeDriverAxis` type (`'width' | 'height'`) and adds an optional `driverAxis` parameter to `computeLockedResizeRect` and the internal `computeLockedRect` callback in `RectangleStencil`. 

Previously, the driver axis was determined dynamically by comparing pixel distances:
```ts
const pixelDistW = distW * imageSize.width;
const pixelDistH = distH * imageSize.height;
if ( pixelDistW / pixelDistH > pixelRatio ) { ... }
```
The updated logic prioritizes an explicit `driverAxis` when provided (used for keyboard input), falling back to the pixel-distance comparison only for pointer drags:
```ts
const isWidthDriver =
    driverAxis === 'width' ||
    ( ! driverAxis &&
        ( distW * imageSize.width ) / ( distH * imageSize.height ) >
            pixelRatio );
```
In `rectangle-stencil.tsx`, keyboard handlers now derive `keyboardDriverAxis` from the active arrow key (`dx !== 0 ? 'width' : 'height'`) and pass it to `computeLockedRect`, preventing the zero-distance axis from overriding the intended resize step.

## Contribution

Opened to address a keyboard accessibility gap in the cropper. During review, a potential mouse-drag size jump near the pixel-motion threshold was identified; the author noted it only reproduced with the original aspect ratio, and the final merged diff focuses strictly on the keyboard driver-axis fix without including a separate patch for the drag behavior.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
