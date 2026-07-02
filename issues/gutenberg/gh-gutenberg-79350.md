# #79350: Mark all controlled/mode block changes non-persistent

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jsnajdr
- **Labels:** `[Type] Regression`, `[Package] Editor`, `[Package] Block library`, `[Package] Block editor`, `Backport to Gutenberg Minor Release`, `Backported to WP Core`
- **Merged:** [`09893de`](https://github.com/WordPress/gutenberg/commit/09893de1255802f9329ab3cb2f1387ee7826b773)
- **Discussion:** [#79350](https://github.com/WordPress/gutenberg/pull/79350) · 15 comments · 2 reactions
- **Usefulness:** 3/5

## Summary

Fixes a regression where internal editor side effects incorrectly marked the post as dirty. Runtime-only mutations to controlled inner blocks and block editing modes were being written persistently, causing the `withPersistentBlockChange` higher-order component to reset the persistent edit flag and trigger false unsaved-changes prompts.

## Impact

- **Block/extension developers**: The persistence behavior of `setBlockEditingMode()`, `unsetBlockEditingMode()`, and `setHasControlledInnerBlocks()` is corrected. Mount-time or cleanup-side edits will no longer incorrectly mark the editor dirty.
- **Core block authors** (Navigation, Site Logo, Query Pagination Modal): Internal mount/cleanup effects are now properly flagged as non-persistent, eliminating false-positive dirty states after initial render.
- **Plugin & theme developers**: No external API changes. Existing code using these dispatches will work correctly; new implementations should ensure their own `useEffect` hooks and cleanup functions align with the `__unstableMarkNextChangeAsNotPersistent()` pattern to avoid persisting runtime-only state.

## Technical details

The patch injects `__unstableMarkNextChangeAsNotPersistent()` immediately before every dispatch of `setBlockEditingMode`, `unsetBlockEditingMode`, and `setHasControlledInnerBlocks` within `useEffect` hooks and their corresponding cleanup functions. It also applies the flag to runtime-side edits in `replaceInnerBlocks([])` (cleanup), `toggleSelection()`, and attribute resets in `enhanced-pagination-modal.js` and `site-logo/edit.js`. By marking these `state.blocks` subtree mutations as non-persistent before they reach the data layer, the `withPersistentBlockChange` HOC stops treating editor-controlled state shifts as user edits. Affected files include:
- `packages/block-editor/src/components/block-editing-mode/index.js`
- `packages/block-editor/src/components/provider/use-block-sync.js`
- `packages/editor/src/components/provider/disable-non-page-content-blocks.js`
- `packages/editor/src/components/provider/navigation-block-editing-mode.js`
- `packages/block-library/src/navigation/edit/index.js`
- `packages/block-library/src/query/edit/enhanced-pagination-modal.js`
- `packages/block-library/src/site-logo/edit.js`

## Contribution

Opened and merged by @jsnajdr with extensive review from the Gutenberg core team. The PR closes #78989 by applying a targeted patch using `__unstableMarkNextChangeAsNotPersistent()` rather than refactoring the HOC's change-detection logic, which was flagged as a cleaner long-term fix but deferred until subtree segregation is implemented. Backported to both the Gutenberg minor release and WordPress Core.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
