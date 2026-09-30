# #79058: Image: Fix lightbox using thumbnail aspect ratio instead of original image ratio

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @SainathPoojary
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Image`
- **Merged:** [`62a159a`](https://github.com/WordPress/gutenberg/commit/62a159ac1af3af4a1476f242f6bebd513cbf0b53)
- **Discussion:** [#79058](https://github.com/WordPress/gutenberg/pull/79058) · 10 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Image block lightbox previously displayed images at the thumbnail's cropped aspect ratio rather than the original image's ratio when "Enlarge on click" was enabled. This fix captures the full-size ratio before any mutation in `setOverlayStyles()`, removes the old ratio-recalculation block that was overwriting it, and introduces a `clip-path`-based animation that smoothly reveals the uncropped image during the zoom transition. The lightbox container is now always sized to the full image dimensions, with the thumbnail crop handled visually via `clip-path` inset values.

## Impact

- **Site owners / editors:** When "Enlarge on click" is enabled on an Image or Gallery block (especially with "Crop images to fit"), the lightbox now shows the image at its original proportions with a smooth clip-path reveal animation instead of a jarring shape jump. No configuration change required.
- **Theme developers:** The lightbox container div gains a new `lightbox-thumbnail-container` class. New CSS custom properties are set on the overlay: `--wp--lightbox-initial-clip`, `--wp--lightbox-thumbnail-width`, `--wp--lightbox-thumbnail-height`. The existing `--wp--lightbox-image-width` and `--wp--lightbox-image-height` now always equal the full container dimensions (previously they could be smaller when the thumbnail ratio differed). If you override or read these variables in custom CSS/JS, audit your assumptions.
- **Plugin & block developers:** No public API, hook, or REST schema changes. The change is internal to the Image block's `view.js` and its SCSS.
- **No action required** for most developers unless you have custom lightbox styling that depends on the old `--wp--lightbox-image-width`/`--wp--lightbox-image-height` semantics.

## Technical details

In `packages/block-library/src/image/view.js` (`setOverlayStyles`):

- `originalRatio` is now `const` (was `let`, reassigned in the removed block).
- `imgRatio` is renamed to `fullSizeRatio` and made `const`; the entire `if (naturalRatio.toFixed(2) !== imgRatio.toFixed(2))` block that recalculated `imgMaxWidth`/`imgMaxHeight` and mutated `imgRatio` is deleted.
- Container sizing now uses `fullSizeRatio` instead of `originalRatio`:

```js
// Before
if ( originalRatio > targetContainerRatio ) {
    containerWidth = targetMaxWidth;
    containerHeight = containerWidth / originalRatio;
}

// After
if ( fullSizeRatio > targetContainerRatio ) {
    containerWidth = targetMaxWidth;
    containerHeight = containerWidth / fullSizeRatio;
}
```

- A new `hasCroppedSource` flag and `sourceRatio` variable determine whether the thumbnail file itself is cropped relative to the full image.
- `containerScale` is now computed as `Math.max()` of four expressions to account for the crop offset, and `cropX`/`cropY` are derived to center the thumbnail within the full-size container.
- `screenPosX` and `screenPosY` are adjusted by `cropX * containerScale` and `cropY * containerScale` so the zoom origin aligns with the visible thumbnail area.
- New CSS custom properties emitted on the overlay element: `--wp--lightbox-initial-clip` (an `inset()` value), `--wp--lightbox-thumbnail-width`, `--wp--lightbox-thumbnail-height`.

In `packages/block-library/src/image/style.scss`:

- New `.lightbox-thumbnail-container` rule sets `--wp--lightbox-image-width`/`--wp--lightbox-image-height` to the thumbnail dimensions (overriding the full-size values set inline).
- `@keyframes lightbox-zoom-in` and `lightbox-zoom-out` gain `clip-path` at their start/end frames, transitioning from `var(--wp--lightbox-initial-clip)` to `inset(0)` (and vice versa), so the crop is animated alongside the existing `transform` zoom.

In `packages/block-library/src/image/index.php`:

- The container `<div>` gains the `lightbox-thumbnail-container` class alongside the existing `lightbox-image-container`.

## Contribution

Opened by @SainathPoojary closing #79040. @jasmussen initially leaned toward closing the issue as expected behavior, noting the shape jump would be jarring, and proposed animating `aspect-ratio` as a middle ground. @t-hamano supplied demo videos clarifying the problem. @lbwtaylor advocated for the fix from a site-hosting perspective. @SainathPoojary ultimately implemented a `clip-path`-based transition (rather than transitioning `aspect-ratio` directly, which @jasmussen had suggested as pseudo-code) and provided a recording showing the smooth reveal. Merged as `62a159a` targeting the 7.2 release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
