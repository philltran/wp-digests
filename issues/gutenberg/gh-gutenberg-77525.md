# #77525: Fix cursor position during forward delete of empty blocks

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @dpmehta
- **Labels:** `[Type] Bug`, `[Feature] Writing Flow`, `[Package] Block editor`, `Backported to WP Core`
- **Merged:** [`1cfca19`](https://github.com/WordPress/gutenberg/commit/1cfca19c6b1c763892a7511c3a16219e0be4bc7a)
- **Discussion:** [#77525](https://github.com/WordPress/gutenberg/pull/77525) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a block editor bug where forward-deleting an empty paragraph block incorrectly placed the cursor at the end of the preceding block instead of the start of the following block. The `mergeBlocks` action in the block editor store now explicitly selects the next block at offset 0 when the removed block is the currently selected, unmodified default block. This restores expected forward-deletion behavior for users and plugin/theme developers relying on consistent writing flow.

## Impact

- **Block editor users:** Forward-deleting an empty paragraph now correctly moves the caret to the beginning of the next block, preventing unexpected cursor jumps.
- **Plugin & theme developers:** No breaking changes or API modifications. Existing code using the block editor store or writing flow actions requires no updates.
- **Hosting & platform teams:** No configuration or migration steps required. The fix ships in WordPress 7.1 via backport.
- **No action required** for existing implementations.

## Technical details

The change modifies the `mergeBlocks` action in `packages/block-editor/src/store/actions.js`. Previously, when merging an unmodified default block, the action called `dispatch.removeBlock( clientIdA, select.isBlockSelected( clientIdA ) )`, which defaulted selection to the previous block. The updated logic checks if the block being removed is currently selected. If so, it batches `dispatch.removeBlock( clientIdA, false )` with `dispatch.selectBlock( clientIdB, 0 )` to explicitly place the cursor at the start of the next block. Otherwise, it simply removes the block.

```js
// Before
if ( isUnmodifiedDefaultBlock( blockA ) ) {
    dispatch.removeBlock( clientIdA, select.isBlockSelected( clientIdA ) );
    return;
}

// After
if ( isUnmodifiedDefaultBlock( blockA ) ) {
    const isASelected = select.isBlockSelected( clientIdA );
    if ( isASelected ) {
        registry.batch( () => {
            dispatch.removeBlock( clientIdA, false );
            dispatch.selectBlock( clientIdB, 0 );
        } );
    } else {
        dispatch.removeBlock( clientIdA, false );
    }
    return;
}
```
An e2e test was added to `test/e2e/specs/editor/various/splitting-merging.spec.js` to verify the caret lands in the next block after forward deletion.

## Contribution

Opened by @dpmehta to resolve #64235, the PR initially targeted the Gutenberg monorepo. Reviewer @ellatrix added an e2e test to cover the behavior before merging. Because the PR was submitted from a forked repository, automatic cherry-picking failed, prompting @t-hamano to manually backport the fix to `wp/7.1` via #80827. @westonruter was added as a co-author during the review process.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
