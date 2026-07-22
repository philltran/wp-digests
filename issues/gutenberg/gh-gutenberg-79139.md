# #79139: Media editor: Snap crop handles to source pixels

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Feature] Media`
- **Merged:** [`0199a9d`](https://github.com/WordPress/gutenberg/commit/0199a9d76c375bdb26c1744fa8fdc30c5aaaf34f)
- **Discussion:** [#79139](https://github.com/WordPress/gutenberg/pull/79139) · 10 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The media editor now snaps freeform crop handles to whole source pixels when the image is displayed at 1:1 scale or larger. This prevents fractional pixel edges during high-zoom crops, making exported regions predictable and precise. The change reduces the minimum crop floor to 16 source pixels and gates snapping to the `PIXEL_SNAP_DISPLAY_SCALE` threshold.

## Impact

- **Block & theme developers**: No breaking changes. The `StencilProps` interface now accepts optional `snapCropRect` and `keyboardResizeStep` props for hosts that want to customize or override the snapping behavior.
- **Media editor users**: Freeform cropping at high zoom now aligns to the source pixel grid. Large/downscaled images remain unaffected.
- **No action required** for existing implementations unless you rely on the previous fractional crop behavior or need to override the new snapping logic.

## Technical details

The diff introduces `snapCropRectToSourcePixels` and `snapCropRectToSourcePixelGrid` in `packages/media-editor/src/image-editor/core/source-region.ts`, which round source-region coordinates to integers and map them back to normalized crop rects via `getCropRectFromSourceRegion`. Snapping is gated by `PIXEL_SNAP_DISPLAY_SCALE = 1` in `constants.ts` and only applies when rotation is at a cardinal stop (checked via `isCardinalRotation`). The `CropperInner` component calculates `displayScale` from `elementSize`, `state.zoom`, and `viewScale`, then passes a `snapCropRect` callback to the stencil. Keyboard resizing now uses a dynamic `keyboardResizeStep` object instead of the hardcoded `0.01` normalized step, ensuring each keypress advances by exactly one source pixel. The `StencilProps` interface in `types.ts` was extended to expose `snapCropRect` and `keyboardResizeStep`.

## Contribution

Opened by @ramonjd as a follow-up to PR #79044. During testing, @fcoveram reported that keyboard resizing on the left/right axes stepped incorrectly and that aspect-ratio keyboard resizing bypassed the snap logic. @ramonjd identified that the keyboard handler still used the legacy `0.01` normalized step and that aspect-ratio crops were explicitly skipped. The keyboard step calculation was patched in a subsequent commit (`8e35704`), while the aspect-ratio keyboard fix was deferred to PR #79207. Co-authored with @andrewserong and @fcoveram.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
