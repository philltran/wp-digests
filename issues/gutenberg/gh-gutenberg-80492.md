# #80492: Writing flow: fully select the items when a selection extends down into a nested item

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `Backport to Gutenberg RC`, `Backported to WP Core`
- **Merged:** [`54c6c1d`](https://github.com/WordPress/gutenberg/commit/54c6c1d13e819c85cb40dee01277d99f10b0818f)
- **Discussion:** [#80492](https://github.com/WordPress/gutenberg/pull/80492) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a block editor bug where extending a text selection downward across a nested list item resulted in a partial text selection instead of a full block multi-selection. The selection observer correctly promoted the range to `MULTI_SELECT`, but the reducer's deduplication logic dropped the promotion when a text selection between the same blocks was already recorded. The fix ensures the selection promotes to a block multi-selection, allowing Backspace to remove the blocks as expected rather than merging text.

## Impact

- **Content editors:** Extending selections downward across nested list items now fully selects the parent list blocks, preventing accidental text merging when pressing Backspace.
- **Plugin & theme developers:** No direct API changes, hooks, or code modifications required.
- **Hosting & platform teams:** No configuration or migration steps needed.
- **No action required** for existing codebases; this is a core editor behavior fix.

## Technical details

The change modifies the `selection` reducer in `packages/block-editor/src/store/reducer.js`. Previously, the `MULTI_SELECT` case deduplicated state by comparing only `clientId` values:

```js
const { start, end } = action;
if (
    start === state.selectionStart?.clientId &&
    end === state.selectionEnd?.clientId
) {
    return state;
}
```

This caused the reducer to discard the block promotion whenever a text selection between the same blocks was already in state. The diff replaces this with a `fastDeepEqual` check against a normalized block selection object:

```js
const nextSelection = {
    selectionStart: { clientId: action.start },
    selectionEnd: { clientId: action.end },
};
if ( fastDeepEqual( state, nextSelection ) ) {
    return state;
}
```

This ensures that text selections (which include `attribute` and `offset` properties) are treated as distinct from block multi-selections, allowing the promotion to persist. Two new e2e tests in `test/e2e/specs/editor/blocks/list.spec.js` verify the downward selection and Backspace removal behavior.

## Contribution

Opened and merged by @ellatrix as a follow-up to #80462, with testing and review by @noruzzamans. The change was backported to the Gutenberg RC and WordPress Core release branches to maintain consistency across upcoming 7.1 and GB 23.6 releases. The discussion centered on aligning downward selection behavior with the existing upward selection logic, with no major alternative approaches debated or rejected.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
