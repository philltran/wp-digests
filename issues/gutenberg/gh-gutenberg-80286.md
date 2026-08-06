# #80286: Writing flow: mark shift+click as a multi-selection gesture

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `Backported to WP Core`
- **Merged:** [`39e5ec1`](https://github.com/WordPress/gutenberg/commit/39e5ec1d89b73ee0244e7cdaf40fe38aa4f7f3f9)
- **Discussion:** [#80286](https://github.com/WordPress/gutenberg/pull/80286) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The block editor now tracks multi-selection gestures using the `isMultiSelecting` store state instead of a module-level global flag. This change replaces the `shiftClickInProgress` utility with the existing `startMultiSelect()` and `stopMultiSelect()` dispatch actions, ensuring the selection observer and focus handler share a single source of truth. It resolves state synchronization issues that could cause native selection ranges to collapse prematurely during shift+click gestures.

## Impact

- **Plugin & theme developers:** No breaking changes or public API modifications. The refactored state remains internal to `@wordpress/block-editor`.
- **Block editor users:** Fixes edge cases where shift+click multi-selection across groups or empty blocks (images, spacers) would incorrectly collapse or fail to span multiple blocks.
- **No action required** for external code. The change is purely internal to the block editor's writing flow components.

## Technical details

The diff removes the `shiftClickInProgress` module global and its `setShiftClickInProgress()`/`isShiftClickInProgress()` exports from `packages/block-editor/src/components/writing-flow/utils.js`. In `packages/block-editor/src/components/writing-flow/use-selection-observer.js`, the `onMouseDown` handler now calls `startMultiSelect()` when `event.shiftKey` is true, and `onMouseUp` calls `stopMultiSelect()`. In `packages/block-editor/src/components/block-list/use-block-props/use-focus-handler.js`, the guard that previously checked `isShiftClickInProgress()` now reads `isMultiSelecting()` from the `blockEditorStore` via `useSelect()`. This consolidates gesture state into the Redux store, eliminating cross-component module globals and aligning with how `use-multi-selection` already checks `isMultiSelecting` to preserve native selection.

## Contribution

Opened by @ellatrix as a follow-up to #79105, the change consolidates an ad-hoc module global into the existing block editor store. @t-hamano backported it to the `wp/7.1` branch to resolve CI failures, with no alternative approaches debated during review.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
