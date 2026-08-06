# #80665: Gallery: make dynamic mode conversion a single undo level

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Gallery`, `Backported to WP Core`
- **Merged:** [`df44798`](https://github.com/WordPress/gutenberg/commit/df4479856182b104b5e21c2a6da4cfc958fae87a)
- **Discussion:** [#80665](https://github.com/WordPress/gutenberg/pull/80665) · 5 comments · 2 reactions
- **Usefulness:** 3/5

## Summary

The Gallery block’s conversion between static and dynamic modes now registers as a single undo step in the block editor. Previously, toggling the mode triggered two independent persistent state updates, causing a single undo to revert only half the operation and leave the block in a corrupted, mixed state. This fix batches the dispatches so editors can cleanly revert the entire mode switch with one undo press.

## Impact

- **Block & plugin developers:** No code changes required. The change is internal to the Gallery block’s editor state management.
- **Site owners & editors:** No action required. Editors will now experience consistent undo behavior when toggling the Gallery block’s “Use images attached to the post” setting.
- **Hosting & platform teams:** No configuration or migration needed. The change is fully contained within the block editor and does not affect REST APIs, database schemas, or public hooks.

## Technical details

The diff modifies `packages/block-library/src/gallery/use-dynamic-gallery.js` to import `useRegistry` and `__unstableMarkNextChangeAsNotPersistent` from `@wordpress/data`. Inside the `enableDynamicMode` and `convertToStatic` functions, the two existing dispatches (`setAttributes` and `replaceInnerBlocks`) are now wrapped in `registry.batch()`. The second dispatch in each pair is immediately followed by `__unstableMarkNextChangeAsNotPersistent()`, which collapses the two state updates into a single undo level.

Before:
```js
setAttributes( { dynamicContent: { source: ATTACHED_MEDIA } } );
replaceInnerBlocks( clientId, [] );
```

After:
```js
registry.batch( () => {
	setAttributes( { dynamicContent: { source: ATTACHED_MEDIA } } );
	__unstableMarkNextChangeAsNotPersistent();
	replaceInnerBlocks( clientId, [] );
} );
```

This aligns the Gallery block with the same undo-batching pattern already implemented in the `tab-list`, `accordion`, and `quote` blocks. No public APIs, filters, or REST routes are affected.

## Contribution

The change addresses a half-revert bug reported in #80613, where undoing a Gallery mode switch left the block with mismatched attributes and inner blocks. Rather than introducing a new batching mechanism, the author adopted the existing `registry.batch()` + `__unstableMarkNextChangeAsNotPersistent()` pattern to standardize undo behavior across the block library. The approach received quick approval during review and was subsequently cherry-picked to the `wp/7.1` branch for inclusion in the upcoming release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
