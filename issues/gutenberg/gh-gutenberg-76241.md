# #76241:  Block Editor: Don't close inserter panel when inline quick inserter opens

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mustafabharmal
- **Labels:** `[Type] Bug`, `[Feature] Inserter`, `[Package] Block editor`
- **Merged:** [`ec35342`](https://github.com/WordPress/gutenberg/commit/ec35342e1c1e46232a3628e0d4d81fbbc43911dd)
- **Discussion:** [#76241](https://github.com/WordPress/gutenberg/pull/76241) · 11 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a Gutenberg bug where clicking the in-between (inline) block appender while the sidebar inserter was open caused the inline quick inserter to vanish immediately. The sidebar inserter closing is still intended, but it no longer tears down the inline inserter that was just opened. The fix makes `useInsertionPoint` leave the shared insertion cue alone when it belongs to the in-between inserter, and clears the cue explicitly on insert instead.

## Impact

- **Plugin & theme developers:** No API changes. Anyone who renders or extends the block inserter via `@wordpress/block-editor` internals gets the corrected behavior automatically with the package update.
- **Site owners / editors:** With the Block Library panel open, clicking the in-between inserter now leaves a working inline quick inserter popover. The sidebar panel still closes, as designed.
- **Custom inserter consumers:** `onToggleInsertionPoint` no longer checks `isBlockInsertionPointVisible()` before showing the cue for a hovered item, and it skips `hideInsertionPoint()` when the current cue has `__unstableWithInserter`. Code or tests that relied on the old show/hide sequence could observe a difference.
- **No action required** for most sites.

## Technical details

**Root cause (from the review thread):** The sidebar inserter's `BlockTypesTabPanel` runs an effect on unmount that calls `onToggleInsertionPoint`, which calls `hideInsertionPoint()`. The insertion point is shared block-editor store state, and the in-between inserter mounts its `QuickInserter` inside that cue's popover. So when the sidebar closed, hiding the cue unmounted the inline inserter that had just opened.

**Changes in the diff:**

- `inserter/hooks/use-insertion-point.js`: replaces the `isBlockInsertionPointVisible` selector with `getBlockInsertionPoint`. `onToggleInsertionPoint( item )` now always shows the cue when `item` is provided. In the else branch, it calls `hideInsertionPoint()` only if `! getBlockInsertionPoint()?.__unstableWithInserter`.
- `inserter/menu.js`: `InserterMenu` now gets `hideInsertionPoint` from `useDispatch( blockEditorStore )`. `onInsert` calls it before `onInsertBlocks`, and `onInsertPattern` calls it directly instead of `onToggleInsertionPoint( false )`. The dependency arrays are updated accordingly. The rationale is that after insertion the cue points at a position that no longer exists, so it is cleared regardless of what showed it.
- Adds an e2e test in `test/e2e/specs/editor/various/inserting-blocks.spec.js`. It opens the Block Library, clicks the in-between `Add block` button, and asserts the panel is hidden while `.block-editor-inserter__quick-inserter` stays visible and usable (search "Table", insert it).
- Adds a `packages/block-editor/CHANGELOG.md` entry.

The originally proposed approach in the PR description (removing the `useEffect` in `QuickInserter` that called `setInserterIsOpened( false )`) is not what merged; the diff leaves `quick-inserter.js` untouched.

## Contribution

The first iteration removed the `setInserterIsOpened( false )` effect from `QuickInserter`, but @Mamaduka found it broke the "Browse all" button, and @jeryj noted that closing the sidebar on an on-canvas insert is intentional design. @t-hamano traced the real cause to `hideInsertionPoint()` unmounting the quick inserter and proposed checking `__unstableWithInserter`. The author reworked the PR, and @t-hamano asked for an e2e test. @Mamaduka also questioned an intermediate version that used a ref for bookkeeping rather than reading the selector, since refs can go stale. The merged diff reads `getBlockInsertionPoint()` directly.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
