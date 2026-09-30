# #82145: List: Merge into the previous line on Backspace instead of outdenting

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mcsf
- **Labels:** `[Type] Bug`, `[Package] Block library`
- **Merged:** [`401f90b`](https://github.com/WordPress/gutenberg/commit/401f90b1c5cc428392130d8ad7f061f47d8ec7f6)
- **Discussion:** [#82145](https://github.com/WordPress/gutenberg/pull/82145) · 7 comments · 2 reactions
- **Usefulness:** 3/5

## Summary

Pressing Backspace at the start of a nested list item in the core List block now merges it into the previous line instead of outdenting it one level at a time. When a parent item's line is merged into (e.g. an emptied `c` merging into `b`), its remaining children keep their original indentation rather than becoming siblings. This removes the old behavior where repeated Backspace walked an item out to the top level before merging, which left former children at the wrong depth.

## Impact

- **Editors / site owners:** Backspace in nested lists now behaves like most text editors (e.g. Apple Notes): it merges with the previous line without changing indentation first. Outdenting is still available via `Shift+Tab`. Users who relied on repeated Backspace to outdent will notice the change.
- **Plugin & theme developers:** No API, hook, or block.json change. Stored markup is unchanged; only the editing interaction differs.
- **Test authors:** Custom E2E tests that simulate Backspace to outdent list items in `core/list` will need updating, e.g. to `Shift+Tab`, as the upstream test did.
- No migration or configuration required.

## Technical details

All logic changes are in `packages/block-library/src/list-item/hooks/use-merge.js` (`useMerge`).

**Backward merge (Backspace):**
- Removed the early return that called `outdentListItem( clientId )` whenever `getParentListItemId( clientId )` was truthy.
- The previous-sibling path is unchanged: `mergeWithNested( getTrailingId( previousBlockClientId ), clientId )`.
- New fallback: if there is no previous sibling but a parent list item exists, it calls `mergeWithNested( parentListItemId, clientId )`, i.e. a first child merges into its parent's own line.

**Forward merge (Delete):** the branch `else if ( getParentListItemId( nextBlockClientId ) ) { outdentListItem( nextBlockClientId ); }` was removed, so it now goes straight to `mergeWithNested( clientId, nextBlockClientId )`.

**`mergeWithNested`:**
- Added `getBlockIndex` to the selectors.
- New branch for when `getParentListItemId( clientIdB ) === clientIdA` (merging into the parent's own line): the merged item's nested children are moved via `moveBlocksToPosition` to `getBlockRootClientId( clientIdB )` at `getBlockIndex( clientIdB ) + 1`, so they take the item's place one level up.
- Captures `listId = getBlockRootClientId( clientIdB )` before `mergeBlocks( clientIdA, clientIdB )`, then calls `removeBlock( listId, false )` if that list is left with no items, preventing an empty nested `core/list`.

**Tests** (`test/e2e/specs/editor/blocks/list.spec.js`): the existing sequence test now expects 6 Backspace presses to remove the list instead of 9 (no intermediate outdent steps). One test that used Backspace to outdent now uses `Shift+Tab`. A new test, *should try to preserve the indentation level of nested items as their parent gets merged*, builds `a > b > (c, d)` plus `e`, empties `c`, and asserts that a further Backspace leaves `d` as a child of `b`.

## Contribution

Follow-up to #82011 and part of #81453. @mcsf opened it initially with only an E2E test after @sgomes observed that repeated Backspace left a nested item at the wrong depth. The description weighed two approaches: keep Backspace-as-outdent and make the editor remember the intended indentation (judged very difficult), or adopt the merge-without-reindent behavior of other software such as Apple Notes (judged very doable but a departure from convention). @ellatrix then pushed the implementation, taking the second approach, and @mcsf edited a comment and merged.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
