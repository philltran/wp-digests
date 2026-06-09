# #78864: RTC: Fix Yjs undo manager to update UI state when undo stack changes

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @alecgeatches
- **Labels:** `[Type] Bug`, `[Package] Core data`, `[Package] Block editor`, `[Feature] Real-time Collaboration`, `[Package] Sync`
- **Merged:** [`cd45134`](https://github.com/WordPress/gutenberg/commit/cd4513491ec3375b7e6debba59e0fac1820e8c96)
- **Discussion:** [#78864](https://github.com/WordPress/gutenberg/pull/78864) · 6 comments · 0 reactions

## Summary

Fixes a Real-Time Collaboration (RTC) bug where the Yjs undo manager fails to trigger UI re-renders when its internal undo stack changes. The update synchronizes stack mutations with the core-data store and introduces a `history` flag to `__unstableMarkNextChangeAsNotPersistent()` to prevent async inner-block template loads from erroneously creating independent undo entries that break state tracking.

## Impact

- **Block editor & RTC developers**: The sync manager now explicitly bridges Yjs stack mutations with the `core-data` store, ensuring `hasUndo()`/`hasRedo()` selectors accurately reflect editor state for toolbar and UI controls.
- **Plugin/theme developers using unstable APIs**: `__unstableMarkNextChangeAsNotPersistent()` now accepts a `history` parameter (`'merge'` or `'ignore'`). Default behavior remains unchanged (`'merge'`), but async template synchronization explicitly opts into `'ignore'` to drop stale edits from history. No breaking changes for standard implementations.
- **Site editors**: More reliable undo/redo UI state during real-time collaboration and when inner-block templates load asynchronously.

## Technical details

- Added an `onUndoStackChange()` callback to the sync manager’s handler registry, forcing the `core-data` store to re-evaluate when Yjs modifies its stack.
- Extended `__unstableMarkNextChangeAsNotPersistent()` to accept a `history` config value. Previously, non-persistent calls relied on surrounding state changes for implicit history merging; the diff applies `history: 'ignore'` in `packages/block-editor/src/components/inner-blocks/use-inner-block-template-sync.js` to prevent fresh page loads from generating isolated undo levels.
- Updated core-data selectors (`hasUndo()`, `hasRedo()`) to react directly to the newly synced stack state, replacing the previous implicit dependency on unrelated store mutations for reactivity.

## Contribution

Merged by @alecgeatches. Opened to address a test failure in `inner-blocks-templates.spec.js` exposed by CRDT timing adjustments in #78756. The resolution consolidated explicit stack synchronization and introduced a dedicated history flag to separate merge semantics from ignored async template edits.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
