# #80953: List View: Collapse off-window placeholder rows into a single spacer row

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Performance`, `[Feature] List View`, `[Package] Block editor`
- **Merged:** [`1c4565c`](https://github.com/WordPress/gutenberg/commit/1c4565c241405993d8d79c0f57bb40eb0e0db18f)
- **Discussion:** [#80953](https://github.com/WordPress/gutenberg/pull/80953) · 8 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

The List View component now collapses consecutive off-window placeholder rows into a single spacer row sized to the run, rather than rendering a `<tr><td>` pair for every block outside the render window. This change drastically reduces DOM node count and eliminates unnecessary style recalculations when opening the List View on posts with many blocks.

## Impact

- **Block editor users & developers:** No action required. The List View renders significantly fewer DOM nodes on long posts, improving open and scroll performance.
- **Theme/plugin developers:** No API changes or configuration required. The `block-editor-list-view-placeholder` class remains, but multi-row spacers now use an inline `style` height instead of relying solely on CSS.
- **Platform/Hosting:** No server-side or configuration changes. Pure client-side block editor optimization.

## Technical details

The diff refactors `packages/block-editor/src/components/list-view/branch.js` to replace the per-block placeholder rendering loop with an accumulator (`placeholderRows`) that flushes a single `<tr><td class="block-editor-list-view-placeholder">` with an inline `height` calculated as `placeholderRows * BLOCK_LIST_ITEM_HEIGHT`. The `BLOCK_LIST_ITEM_HEIGHT` constant (value `32`) was moved from `index.js` to `utils.js` to resolve a circular dependency and align with `style.scss`. The `useRef` tracking for `nextPositionRef` was replaced with a local `nextPosition` variable, and the `react-hooks/refs` ESLint suppression for `branch.js` was removed. TreeGrid semantics, `windowOverscan` math, and navigation behavior remain unchanged; spacers carry no `role` and do not interfere with `[data-block]` drop zones or keyboard navigation.

## Contribution

Opened and merged by @Mamaduka with review contributions from @andrewserong and @jsnajdr. During review, @andrewserong noted the approach mirrors techniques used in other virtualized UIs and shared a screencast highlighting a separate CMD+A selection performance regression on long posts, which @Mamaduka acknowledged as a future optimization target. The final merge reflects only the spacer-row consolidation, with the selection lag noted but deferred.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
