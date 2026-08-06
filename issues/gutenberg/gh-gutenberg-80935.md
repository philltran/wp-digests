# #80935: List View: Speed up opening by removing a second render pass

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Performance`, `[Package] Compose`, `[Feature] List View`
- **Merged:** [`4641cbd`](https://github.com/WordPress/gutenberg/commit/4641cbdf31cc80708d9c99982a5fa8340f262070)
- **Discussion:** [#80935](https://github.com/WordPress/gutenberg/pull/80935) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `useFixedWindowList` hook in `@wordpress/compose` now skips a redundant render pass when opening the block editor’s List View. Previously, the hook would render an initial 30-item window, measure the viewport in a layout effect, and then force a second synchronous render to adjust the window—triggering duplicate style recalculations before paint. The fix checks if the initial window already covers the visible items and, if so, retains it, eliminating the pre-paint double commit.

## Impact

- **Block editor users**: Faster List View opening on standard desktop resolutions (up to ~1080px panels).
- **Plugin & theme developers**: No direct API changes; the optimization is internal to `@wordpress/compose`.
- **Hosting & platform teams**: No configuration or migration required.
- **Note**: The optimization only applies when the default 30-item initial window covers the viewport; taller panels retain the original two-pass behavior.

## Technical details

In `packages/compose/src/hooks/use-fixed-window-list/index.ts`, `visibleItems` is moved from state to a `useRef` (`visibleItemsRef`) to prevent measurement from triggering re-renders. A new `isFirstMeasurementRef` gates a guard inside `setFixedListWindow`: if the current window already spans `[firstViewableIndex, firstViewableIndex + visibleItems]`, it returns `lastWindow` instead of updating. The `measureWindow` callback is stabilized with `useEvent`, and the original single effect is split into two `useLayoutEffect` hooks—one for measurement and one for attaching/detaching `scroll`, `resize`, and `keydown` listeners. Debouncing is removed since those events already align with the rendering rate. Keyboard paging (`PAGEUP`/`PAGEDOWN`) now reads `visibleItemsRef.current` instead of `fixedListWindow.visibleItems`.

## Contribution

Merged by @Mamaduka with co-authors @tyxla and @jsnajdr. During review, @jsnajdr proposed removing the debounce, restructuring effect dependencies, and detaching listeners only when the scroll container changes. @Mamaduka benchmarked the alternatives and found that making the scroll container reactive or removing the initial measurement guard reintroduced the double-render penalty or caused stale viewport calculations on dynamic list growth. @jsnajdr accepted the trade-offs, and the change shipped with the original guard and dependency structure.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
