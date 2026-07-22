# #80462: Writing flow: fully select the ancestor when a text selection crosses a nesting boundary

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `Backported to WP Core`
- **Merged:** [`4787818`](https://github.com/WordPress/gutenberg/commit/4787818ca8f5093506202e16ac37ef5aefa21061)
- **Discussion:** [#80462](https://github.com/WordPress/gutenberg/pull/80462) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a block editor bug where text selections crossing a nesting boundary (e.g., from a nested list item to its parent) were mishandled, causing key presses like Backspace or Delete to be dropped. The selection observer now detects ancestor/descendant relationships and resolves the selection to the outer block, treating it as fully selected so it can be deleted in a single press. This restores expected writing flow behavior for nested blocks.

## Impact

- **Block editor users:** Text selections spanning nested blocks now behave consistently with multi-selections, allowing single-press deletion.
- **Plugin & theme developers:** No direct API changes or breaking changes. The fix operates entirely within the block editor's internal selection store and writing flow components.
- **No action required** for existing codebases; this ships as a core block editor bug fix.

## Technical details

The diff modifies three core files in `packages/block-editor/`. In `store/selectors.js`, a new `getSelectionNestingAncestor(state)` helper checks if one selection endpoint is a parent of the other using `getBlockParents()`. `getSelectedBlockClientIds` now returns `[nestingAncestorClientId]` when detected, and `__unstableIsFullySelected` returns `true` for this state. In `use-selection-observer.js`, an `isAncestorDescendant` guard prevents the observer from incorrectly promoting the selection to `multiSelect()`, allowing the native selection to persist while the store treats it as a single fully-selected block. `block-parent-selector/index.js` switches from `getSelectedBlockClientId()` to `getSelectedBlockClientIds()` to correctly resolve the ancestor client ID for parent selection UI. E2E tests in `list.spec.js` verify backward, forward, and drag selections across nesting boundaries.

## Contribution

Opened by @ellatrix to address #76233, the PR was reviewed and merged, then cherry-picked to both the `wp/7.1` and `release/23.6` branches for inclusion in the upcoming WordPress release. The discussion was minimal, with the author noting the implementation was authored with Claude Code under direction. No alternative approaches were debated in the thread; the fix focuses on resolving the selection state rather than implementing partial merge logic, which is tracked separately in #80489.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
