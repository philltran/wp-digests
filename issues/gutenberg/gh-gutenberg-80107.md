# #80107: Tab List: Add toolbar buttons to reorder tabs

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Block] Tabs`
- **Merged:** [`2c4ca0b`](https://github.com/WordPress/gutenberg/commit/2c4ca0b2316c3c6732250d049de8c05dfae4fea6)
- **Discussion:** [#80107](https://github.com/WordPress/gutenberg/pull/80107) · 10 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Tabs block now includes toolbar buttons to reorder tabs directly from the block editor. Previously, only the underlying tab panels could be reordered, which did not update the tab list order. This change adds a `moveTab` action to the `useTabActions` hook and injects a `TabMovers` component into the block toolbar, allowing editors to shift the active tab left or right while keeping it selected.

## Impact

- **Site editors & content teams**: Can now reorder tabs via the block toolbar without navigating to the inner panel blocks.
- **Plugin & theme developers**: No breaking changes or public API modifications. The internal `useTabActions` hook gains a `moveTab` callback, but existing implementations remain fully compatible.
- **Platform & QA teams**: Verify the new toolbar buttons and ensure undo/redo (`primary+z`) correctly restores tab order.
- No migration, configuration, or code changes required.

## Technical details

- Adds `TabMovers` component (`packages/block-library/src/tab-list/tab-movers.js`) which renders two `ToolbarButton` controls inside a `BlockControls` group `"parent"`.
- Extends `useTabActions` (`packages/block-library/src/tabs/use-tab-actions.js`) with a `moveTab( direction )` callback that calls `moveBlocksToPosition` from `blockEditorStore` to reorder the underlying `core/tab-panel` blocks and updates the `editorActiveTabIndex` attribute on the parent `core/tabs` block.
- Registers an editor-only stylesheet via the `editorStyle` field in `packages/block-library/src/tab-list/block.json` (`wp-block-tab-list-editor`), scoped to `.wp-block-tab-list__mover-button`.
- Before/after pattern for the hook:
  ```js
  // Before
  const { insertTab, removeTab } = useTabActions( clientId );
  // After
  const { insertTab, removeTab, moveTab } = useTabActions( clientId );
  moveTab( -1 ); // Move active tab left
  moveTab( 1 );  // Move active tab right
  ```

## Contribution

Opened by @t-hamano to address the asymmetry in tab reordering. Reviewed and approved by @jeryj and @noruzzamans, with additional testing from @ciampo and @Mamaduka. The PR was backported to WordPress 7.1 as a usability enhancement that introduces no new public APIs.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
