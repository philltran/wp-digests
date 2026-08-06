# #80507: Block Editor: Try to fix typing performance regression

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Performance`, `[Package] Block editor`, `Backported to WP Core`
- **Merged:** [`bde8c97`](https://github.com/WordPress/gutenberg/commit/bde8c97981b8af7c31bde40a5068054cd7b7732b)
- **Discussion:** [#80507](https://github.com/WordPress/gutenberg/pull/80507) · 11 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Fixes a typing performance regression in the Block Editor caused by `canHostEditableRoot` performing an O(n) sibling scan inside a `useSelect` hook on every keystroke. The selector is now memoized as a private store selector keyed on block order and editing mode, ensuring cache hits during attribute updates and eliminating redundant subscriptions in unselected `RichText` instances.

## Impact

- **Plugin & theme developers:** No breaking changes or public API modifications. The internal `canHostEditableRoot` logic was moved to the store, but it was never part of the public hook API.
- **Block editor users:** Restores typing responsiveness in posts with many top-level blocks (e.g., 1000+ blocks) where the previous implementation saturated the frame budget and consumed ~26% of busy CPU.
- **Platform & hosting teams:** No configuration or migration steps required. The change ships with the Gutenberg package and is backported to WP Core.

## Technical details

- Moves `canHostEditableRoot` from `packages/block-editor/src/components/writing-flow/use-editable-root.js` to `packages/block-editor/src/store/private-selectors.js` and wraps it in `createSelector`.
- The new selector depends on `[state.blocks.order, state.blocks.parents, state.blocks.byClientId, state.blocks.blockEditingModes, state.derivedBlockEditingModes, state.blocksMode]`. Because typing dispatches `UPDATE_BLOCK_ATTRIBUTES`, which does not mutate these dependencies, the memoized result returns immediately on every keystroke.
- `RichTextWrapper` in `packages/block-editor/src/components/rich-text/index.js` now reads `hasDefaultEditingMode` from `blockEditingModeKey` and short-circuits the `useSelect` call when the block isn't selected or editable, reducing store subscriptions to zero for unselected instances.
- `useEditableRoot` in `packages/block-editor/src/components/writing-flow/use-editable-root.js` consolidates its two prior selectors into a single `useSelect` call and removes the now-redundant `useHasEditableRoot` hook.
- `useSelectionObserver` in `packages/block-editor/src/components/writing-flow/use-selection-observer.js` updates to call the store selector directly via `unlock( useSelect( blockEditorStore ) )`.

## Contribution

Opened and merged by @Mamaduka, with review input from @ellatrix and @t-hamano. The PR addresses a regression introduced in #79105. During review, @Mamaduka noted that while the memoization significantly improved performance, it hadn't fully restored the original baseline, prompting further investigation into remaining bottlenecks. The team agreed to land the memoization refactor immediately and backport it to the `wp/7.1` branch.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
