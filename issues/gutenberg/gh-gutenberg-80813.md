# #80813: Writing flow: forward delete an empty paragraph without breaking apart the next block

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `Backported to WP Core`
- **Merged:** [`fd22f1c`](https://github.com/WordPress/gutenberg/commit/fd22f1c370d248bccbe4e2ca25be8baa39d05ca7)
- **Discussion:** [#80813](https://github.com/WordPress/gutenberg/pull/80813) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Forward deleting an empty paragraph block no longer breaks apart the structure of the following block. Previously, pressing Delete on an empty paragraph before a list would pull the list's first item out and demote it to a paragraph. The fix removes the empty paragraph and places the caret at the start of the next block's first leaf, matching standard editor behavior and aligning with how backspace already works.

## Impact

- **Block editor users & content authors:** No manual action required. The writing experience now behaves consistently when using the forward delete key on empty paragraphs.
- **Plugin & theme developers:** No action required. This is an internal block editor behavior fix with no public API changes.
- **Hosting & platform teams:** No action required. The change is contained within the block editor package and ships via standard WordPress/Gutenberg updates.

## Technical details

The change modifies the forward-delete branch of the `onMerge` handler in `packages/block-editor/src/components/block-list/block.js`. Previously, the logic immediately called `moveFirstItemUp( nextBlockClientId, false )` whenever the next block had children. The diff introduces a check for `isUnmodifiedDefaultBlock( getBlock( clientId ) )`. When true, it executes a `registry.batch()` that calls `removeBlock( clientId )` and `selectBlock( firstLeafClientId )`, where `firstLeafClientId` is resolved by traversing `getBlockOrder()` until a block with no children is found. This mirrors the existing behavior in `mergeBlocks` and `switchToDefaultOrRemove`. An e2e regression test was added to `test/e2e/specs/editor/various/splitting-merging.spec.js` to verify the behavior with an empty paragraph followed by a `core/list`.

## Contribution

Authored by @ellatrix and reviewed by core contributors, the change was characterized as low-risk with a tiny surface area and fast-tracked into the 7.1 beta cycle. It was merged and cherry-picked without notable design debate or alternative approaches discussed in the thread.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
