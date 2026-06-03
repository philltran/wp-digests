# #78742: Writing flow: Delete at end of nested list item should merge into next block

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Package] Block library`
- **Merged:** [`9afcf2a`](https://github.com/WordPress/gutenberg/commit/9afcf2a47961bc783450fd07b16c15ef63e54b22)
- **Discussion:** [#78742](https://github.com/WordPress/gutenberg/pull/78742) · 3 comments · 0 reactions

## Summary

Pressing the Delete key at the end of a deeply-nested `core/list-item` now correctly absorbs the following sibling block into the outermost containing list, instead of silently doing nothing. The bug stemmed from the block editor's fallback `onMerge` handler only walking one parent level up the block tree — not far enough to reach past the outer `<ul>`/`<ol>` wrapping a nested item. The fix is self-contained inside `list-item`'s `useMerge` hook; the shared `mergeBlocks` action is unchanged.

## Impact

**Content editors**
- Delete-at-end-of-nested-list-item now works as expected: a following paragraph collapses into the outermost list as a new item; a following list contributes its items as siblings. Caret position is preserved.

**Plugin & theme developers**
- No action required. No public APIs, filters, hooks, block.json fields, or REST schema were changed. The change is internal to `packages/block-library/src/list-item/hooks/use-merge.js` and ships only in `build/scripts/block-library/index.min.js` (+67 B).

## Technical details

All new logic lives in `packages/block-library/src/list-item/hooks/use-merge.js` inside the `useMerge` hook for `core/list-item`.

When Delete is pressed at the end of a nested item and no next sibling exists within the immediate `core/list` parent, the hook now:

1. **Walks the ancestor chain** — climbs from the current `list-item` through successive `core/list` / `core/list-item` parent pairs until it reaches the topmost `core/list-item`, then retrieves the block immediately following that item's containing `core/list`.
2. **Merges a same-name list** — if the following block is another `core/list`, its child `core/list-item` blocks are repositioned into the outermost list via `moveBlocksToPosition`, and the now-empty container block is removed.
3. **Converts other block types** — for any other block (e.g. `core/paragraph`, `core/heading`), the hook runs the list's registered `from` transforms via `switchToBlockType`, then inserts the resulting `core/list-item` blocks with `insertBlocks( blocks, index, rootClientId, { updateSelection: false } )` so the caret stays at the end of the nested item.

The generic `mergeBlocks` dispatch action in `@wordpress/block-editor` was not modified.

## Contribution

PR opened and merged by @ellatrix (Ella van Durpe), closing issue #77245. Co-authors credited in the merge commit are @dpmehta and @mcsf. The PR notes it was authored with Claude Code (Opus 4.7) under direction and review. Three bot comments (props attribution, size report, flaky-test notice) make up the full recorded discussion; no alternative approaches or design debate are documented in the thread.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
