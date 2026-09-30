# #42503: Quote & List v2: Deleting empty list item should list block

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Block] List`, `[Block] Quote`, `[Package] Block editor`
- **Merged:** [`bc0a68e`](https://github.com/WordPress/gutenberg/commit/bc0a68e7aee81dc4b1ef143ca11c3c03e1ef1f31)
- **Discussion:** [#42503](https://github.com/WordPress/gutenberg/pull/42503) · 19 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

When every inner block of an otherwise unmodified container block is removed, the container is now removed along with them. Previously, deleting the only item in a List v2 or Quote v2 block left an empty wrapper that needed a second deletion. The behavior is implemented generically in the block-editor store's `privateRemoveBlocks` action rather than as a per-block opt-in.

## Impact

**Plugin & theme developers**
- Any container block whose inner blocks are all removed while the container's own attributes are still default will now be removed too. This affects custom blocks built on `InnerBlocks`, not just core List and Quote.
- Blocks with a `parent` restriction in `block.json` (e.g. `core/column` inside `core/columns`) are exempt from being removed this way, so their layout is preserved.
- A container is only removed if `isUnmodifiedBlock()` returns true. Setting any non-default attribute (e.g. an `anchor`) keeps the empty container.
- Code or tests that assume an empty container remains after the last child is removed (e.g. e2e tests asserting `innerBlocks: []`) may need updating.

**Site owners / editors**
- Deleting the last list item or quote paragraph via the block toolbar, or by multi-select plus Backspace, now removes the whole list or quote. This matches List v1 and Quote v1 behavior.
- Deleting all columns in a Columns block now removes the Columns block too.

No migration required.

## Technical details

The change is in `packages/block-editor/src/store/private-actions.js`, inside `privateRemoveBlocks`. After the existing checks, it computes a `removeRoot` flag:

- It gets `rootClientId` via `select.getBlockRootClientId( clientIds[ 0 ] )`.
- If a root exists and its block type has no `parent` restriction (`getBlockType( ... )?.parent?.length` is falsy), it compares `select.getBlockOrder( rootClientId )` against `clientIds`. The lengths must match and every child must be included in `clientIds`.
- It also requires `isUnmodifiedBlock( select.__unstableGetBlockWithoutInnerBlocks( rootClientId ) )` to be true, with `isUnmodifiedBlock` newly imported from `@wordpress/blocks`.

When `removeRoot` is true:

- `selectPreviousBlock` is called with `rootClientId` instead of `clientIds[ 0 ]`.
- The `REMOVE_BLOCKS` action is dispatched with `[ rootClientId ]` instead of `clientIds`.

Both run in the same `registry.batch`, so no extra undo step is created.

```js
// before
dispatch( { type: 'REMOVE_BLOCKS', clientIds } );

// after
dispatch( {
	type: 'REMOVE_BLOCKS',
	clientIds: removeRoot ? [ rootClientId ] : clientIds,
} );
```

The e2e specs are updated accordingly:

- **`list.spec.js`:** adds a new test for deleting a list through the toolbar Options menu, and changes two multi-select removal assertions to expect `[]`.
- **`multi-block-selection.spec.js`:** the columns deletion test now expects `[]`.
- **`block-deletion.spec.js`:** the group test now sets `anchor: 'group'` so the group counts as modified and survives.

No new hooks, `block.json` fields, or REST changes. The generic approach was chosen over an opt-in flag, though the PR discussion still refers to an option.

## Contribution

Opened by @ellatrix to fix #40979 with a generalized approach. @ntsekouras questioned whether the behavior fit the Group block, where discussion had leaned toward improving the empty-state design, and noted that blocks like Site Title lack `onRemove`; @ellatrix replied that this is independent of the wrapper removal and mirrors List v1 and Quote v1. She later put the PR on hold in favor of a separate keyboard-focused PR, while noting this one may still be needed for #40979. @t-hamano pointed out it may also resolve #45919. The PR sat quiet for a long time, and @fabiankaegy asked about its status ahead of the 6.5 Beta 1 cut before it was eventually merged.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
