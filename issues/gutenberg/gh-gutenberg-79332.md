# #79332: Media Editor: Fix crop canvas pinch zoom

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Feature] Media`
- **Merged:** [`02cc069`](https://github.com/WordPress/gutenberg/commit/02cc069fcc715da6fcbe8615b1ae78b1a3a8774b)
- **Discussion:** [#79332](https://github.com/WordPress/gutenberg/pull/79332) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a touch gesture bug in the Gutenberg media editor crop modal where pinch-to-zoom caused visual drift and conflicted with crop resize handles. The patch ensures pinch zoom anchors to the initial pinch midpoint regardless of prior pan/zoom state, properly disambiguates pointer vs. touch event ownership, and disables crop handle interactions during active pinches to prevent gesture interference.

## Impact

- **Media editor users & mobile developers:** Improved pinch-to-zoom accuracy and gesture stability on touch devices; no code changes required.
- **Plugin/theme developers:** No public API changes or breaking updates. Internal crop component behavior is refined, but external consumers relying on the standard media modal are unaffected.
- **Platform/Host teams:** Touch rendering in the block editor's media workflows now follows proper event ownership rules, reducing edge-case conflicts between pointer and touch paths.

## Technical details

- `packages/media-editor/src/image-editor/core/interaction-controller.ts`: Added early return for `e.pointerType === 'touch'` in pointer down handlers to prevent drag UI flashing before pinch ownership is established. Updated focal-point zoom correction to calculate from `touch.startZoom` and the initial midpoint (`startFocalNormX/Y`) instead of the current reducer state, preventing cumulative drift across successive `touchmove` frames. Added `moveEvent.preventDefault()` in touch move handling.
- `packages/media-editor/src/image-editor/react/components/cropper.tsx`: Introduced `isTouchPinching` state tracked via `onTouchStartCapture`, `onTouchMoveCapture`, etc. Guards `handlePointerDown`, `handleResizeStart`, and `handleCropChange` to bail out or cancel active resizes when a pinch begins. On resize end, conditionally clears the settle timer and resets the viewport if the gesture was overridden by a pinch, merging undo states cleanly.
- `packages/media-editor/src/image-editor/react/components/stencils/rectangle-stencil.tsx`: Added `isResizeDisabled` prop to `StencilProps`. When true, prevents pointer/touch start on handles, cancels active pointer resizes via `activePointerResizeRef.current?.cancel()`, ignores keyboard resize steps, and adjusts focus restoration. The stencil now passes this flag through `RectangleStencilProps`.

## Contribution

Opened and implemented by @ramonjd with testing and patch refinement from @andrewserong. During smoke testing on mobile Safari/Chrome, a one-frame drag UI flicker was reported when transitioning into pinch mode; the fix applied an early return for touch `pointerdown` events in the interaction controller to resolve it. Merged as commit `02cc069`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
