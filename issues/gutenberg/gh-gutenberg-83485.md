# #83485: Notes: Refactor floating board measurement

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Feature] Notes`
- **Merged:** [`9ddd97f`](https://github.com/WordPress/gutenberg/commit/9ddd97f4c8c457194a11a3ace43a56aa4fe8e161)
- **Discussion:** [#83485](https://github.com/WordPress/gutenberg/pull/83485) · 11 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Rewrites the floating notes board measurement in `@wordpress/editor` so all DOM reads happen inside a single `ResizeObserver` callback, React consumes a value-compared snapshot via `useSyncExternalStore`, and note positions are derived during render instead of being scheduled through `requestAnimationFrame`. The change eliminates the extra rAF that caused two paints per resize, and fixes several alignment bugs: threads now follow blocks during move animations, stay aligned when editor notices or device preview shift the canvas, handle closed Details blocks, and respect `prefers-reduced-motion` for the ease-in transition.

## Impact

- **Site owners / editors using the floating notes sidebar:** Threads will stay correctly aligned with their blocks during block moves, when editor notices appear above the canvas, when switching device preview, and when a Details block is collapsed. No configuration or code changes required.
- **Plugin & theme developers:** No public API, hook, or REST route changes. The refactored symbols (`createBoardStore`, `useFloatingBoard`, `calculateNotePositions`, `getNoteAnchorRect`) are internal to `packages/editor/src/components/collab-sidebar/`. No action required.
- **Hosting & platform:** Bundle size increases by 730 B (+0.01%) in `build/scripts/editor/index.min.js`. No DB or REST changes.
- **Headless & REST consumers:** No impact.

## Technical details

The core rewrite is in `packages/editor/src/components/collab-sidebar/board-store.js`. The old store held only `heights` and a shallow snapshot; the new store holds `rootEl`, `canvas`, `frameEl`, and a snapshot of shape `{ heights, anchorRects, canvas, frameOffset }`. All DOM reads are consolidated into a single `measure()` function called from the `ResizeObserver` callback (`onResize`). A new `MutationObserver` watches `style` attributes under the root to catch block move animations (which offset blocks via `transform` and therefore trigger no resize); it calls `requestMeasure()` only once the transform clears, not on every animation frame.

`requestMeasure()` works by un-observing and re-observing the root element, exploiting the fact that a new observation always fires one callback before the next paint.

In `hooks.js`, `useFloatingBoard` no longer schedules a `requestAnimationFrame` or stores positions in React state. Instead it subscribes via `useSyncExternalStore` and derives positions with:

```js
// Before (simplified)
useEffect( () => {
  const id = requestAnimationFrame( () => {
    setNotePositions( calculateNotePositions( { …, blockRects: store.getAnchorRects(), … } ) );
  } );
  return () => cancelAnimationFrame( id );
}, [ threads, heights, selectedNoteId, … ] );

// After
const notePositions = useMemo( () => calculateNotePositions( { threads, selectedNoteId, …snapshot } ), [ threads, selectedNoteId, snapshot ] );
```

Anchors are stored in canvas content-space (`rect.top + scrollTop`), so scrolling never changes the snapshot. A new `--canvas-offset` CSS variable (alongside the existing `--canvas-scroll`) translates threads to account for the gap between the canvas frame and the threads' `offsetParent` (e.g. an editor notice above the canvas).

`getNoteAnchorRect(noteId, blockEl)` (moved from `hooks.js` to `utils.js`) now also handles collapsed content: if the anchor element fails `checkVisibility()` (e.g. inside a closed Details block), it climbs to the closest visible block.

The `FloatingContainer` adds a `top` CSS transition for selection-change reflows, gated behind `prefers-reduced-motion`, and a thread's first positioning starts from `top: auto` to avoid an initial animation.

`add-note.jsx` drops the `opacity: 0` hack that hid the "New note" button before its first position was computed.

## Contribution

Opened by @Mamaduka as a follow-up to #79877 (specifically a review thread on that PR) and to fix #83508. @adamsilverstein ran a Playwright stress pass against the branch and identified three remaining edge cases: a growing post title leaving threads behind (pre-existing on trunk), duplicating a noted block sharing the same `noteId` (pre-existing, related to a deeper issue), and the per-frame cost of the new `MutationObserver` during move animations with many notes (~0.60 s script time and 185 layouts for 20 notes vs. ~0.20 s and 40 on trunk). The first two were deferred as separate issues; the MutationObserver cost was noted as a potential follow-up (coalescing callbacks per frame or gating on active transforms). Co-authored with jasmussen, t-hamano, adamsilverstein, and i-am-chitti.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
