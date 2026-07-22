# #79877: Notes: align floating threads with their inline marker

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Package] Editor`, `Backported to WP Core`, `[Feature] Notes`
- **Merged:** [`389f4d8`](https://github.com/WordPress/gutenberg/commit/389f4d8c5abb24d65d40f3915d33ea92b898182f)
- **Discussion:** [#79877](https://github.com/WordPress/gutenberg/pull/79877) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Floating collaborative note threads in the block editor now vertically align with their inline text markers instead of the top of the containing block. This resolves a UX regression introduced in #78218 where notes attached to mid-paragraph selections rendered far from their anchor text. Block-level notes continue to align with the block top, preserving existing behavior for non-inline annotations.

## Impact

- **Editor users & theme/plugin developers:** No breaking changes or public API removals. The behavior change is confined to the collaborative notes sidebar UI.
- **Developers extending the notes store:** `getBlockRects()` has been renamed to `getAnchorRects()` and now returns anchor rects instead of raw block rects. If you override or wrap `createBoardStore()` or pass `blockRects` to `calculateNotePositions()`, update your calls to use the new method.
- **No action required** for standard site owners or developers not interacting with the collaborative notes system.

## Technical details

The primary change occurs in `packages/editor/src/components/collab-sidebar/board-store.js`, where `getBlockRects()` is replaced by `getAnchorRects()`. The new method resolves each thread's anchor at read time to avoid stale DOM references after rich-text re-renders:
- Inline notes query `el.querySelector( getNoteMarkerSelector( id ) )`, targeting `mark.wp-note[data-id]`.
- The pending `"new"` note uses `getSelectionRect( el )` with fallbacks for collapsed, absent, or unrendered selections.
- Block-level notes fall back to `el.getBoundingClientRect()`.

`useFloatingBoard` in `hooks.js` now passes `store.getAnchorRects()` to `calculateNotePositions()` and attaches a `ResizeObserver` to `rootEl` to reschedule the `requestAnimationFrame` when canvas content changes. The marker selector logic was extracted from `note-highlight-styles.js` into a shared `getNoteMarkerSelector()` helper in `utils.js` to ensure consistent targeting. The overlap resolution algorithm was updated to sort threads by measured anchor top before sweeping forward and backward.

## Contribution

Opened by @adamsilverstein and merged with co-authors @Mamaduka and @t-hamano. @t-hamano coordinated the backport to the `wp/7.1` branch for inclusion in WordPress 7.1 Beta 3 and Gutenberg 23.6. The PR required a rebase to resolve test conflicts with #80531, and a flaky test was flagged for separate handling. The implementation prioritized resolving anchors at read time rather than caching them, explicitly to handle rich-text re-rendering replacing marker elements.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
