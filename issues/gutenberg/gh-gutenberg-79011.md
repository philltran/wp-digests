# #79011: Media Editor: Keep crop handles operable on large images

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Feature] Media`
- **Merged:** [`dd59ec2`](https://github.com/WordPress/gutenberg/commit/dd59ec2f05b163dfec5ecd3f45cbe4fbb3eaad88)
- **Discussion:** [#79011](https://github.com/WordPress/gutenberg/pull/79011) · 3 comments · 0 reactions

## Summary

The Media Editor now dynamically scales the minimum crop area based on display zoom to prevent crop handles from overlapping when large images are rendered at a small canvas size. Previously, a hardcoded 24 source-pixel floor would collapse to only a few CSS pixels on high-DPI or zoomed-out views, making the crop region impossible to drag. This update introduces a display-aware floor that keeps touch targets operable until the image is zoomed in enough for the hard 24px limit to take effect.

## Impact

- **Block Editor & Media Editor Developers**: No breaking changes. The `minCropSize` calculation logic inside `@wordpress/media-editor` was updated to be scale-aware.
- **Plugin & Theme Developers**: No action required. This change is confined to the client-side crop component and does not affect server-side image processing, REST schemas, or block attributes.
- **Headless/REST Consumers**: Unaffected. Image cropping here is purely a browser-based UI interaction within the Gutenberg editor.

## Technical details

- **`packages/media-editor/src/image-editor/core/constants.ts`**: Exports a new constant `MIN_CROP_SCREEN_PX = 44`, aligned with the CSS handle touch target size (`$handle-touch-target-size`).
- **`packages/media-editor/src/image-editor/core/stencil-math.ts`**: Adds `getMinCropPixels(displayScale: number): number`. Returns `Math.max(MIN_CROP_PIXELS, MIN_CROP_SCREEN_PX / displayScale)`, effectively binding the usability floor at low zoom and yielding to the source-pixel hard floor as `displayScale` increases.
- **`packages/media-editor/src/image-editor/react/components/cropper.tsx`**: Replaces the hardcoded `MIN_CROP_PIXELS` reference in `CropperInner` with the new utility. Derives `displayScale` via `( elementSize.width / naturalWidth ) * state.zoom`. Updates the `useMemo` dependency array to include `elementSize.width`, `naturalWidth`, `naturalHeight`, `state.rotation`, and `state.zoom`.

**Before/After pattern change:**
```diff
// Before (hardcoded source pixel floor regardless of scale)
const minCropSize: Size | undefined = useMemo( () => {
    // ... bounding box calculations ...
    return {
        width: Math.min( 1, ( MIN_CROP_PIXELS * state.zoom ) / bbox.width ),
        height: Math.min( 1, ( MIN_CROP_PIXELS * state.zoom ) / bbox.height ),
    };
}, [ naturalWidth, naturalHeight, state.rotation, state.zoom ] );

// After (dynamic floor based on display scale)
const displayScale = ( elementSize.width / naturalWidth ) * state.zoom;
const minPixels = getMinCropPixels( displayScale );
return {
    width: Math.min( 1, ( minPixels * state.zoom ) / bbox.width ),
    height: Math.min( 1, ( minPixels * state.zoom ) / bbox.height ),
};
}, [ naturalWidth, naturalHeight, state.rotation, state.zoom, elementSize.width ] );
```

## Contribution

PR #79011 was authored and merged by @ramonjd. The diff focuses strictly on adapting the crop minimum to UI scale rather than source pixels alone. Manual testing in the PR verified behavior across large/small images, zoom levels, and handle drag interactions. No design alternatives or rejections were documented in the available discussion; the solution was accepted as a targeted math-level fix within `stencil-math.ts` with minimal footprint in `cropper.tsx`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
