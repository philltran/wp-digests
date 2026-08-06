# #81210: Block Editor: Fix Select all freezing on large posts

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Performance`, `[Package] Block editor`
- **Merged:** [`c004e77`](https://github.com/WordPress/gutenberg/commit/c004e77c307eb7fa17e70b64657266195248fc36)
- **Discussion:** [#81210](https://github.com/WordPress/gutenberg/pull/81210) · 13 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Fixes a severe performance regression in the Block Editor where pressing the Select All shortcut on large posts (e.g., 1,000+ blocks) caused the editor to freeze for up to 16.8 seconds. The change optimizes the `hasSelectedInnerBlock` deep-check selector and skips redundant parent-walking when blocks are not inside a section, reducing execution time to approximately 0.4 seconds.

## Impact

- **Site owners & editors:** Eliminates editor freezes and unresponsive UI when using Select All on large posts.
- **Plugin & theme developers:** No breaking changes to public APIs. The `hasSelectedInnerBlock` selector contract remains identical, but its internal performance characteristics have improved significantly.
- **Hosting & platform teams:** Reduces CPU spikes and memory churn during heavy block selection operations, improving stability on resource-constrained environments.
- **No action required.** Existing codebases and block themes will automatically benefit from the optimization on upgrade.

## Technical details

The freeze originated from `BlockListBlock` calling `hasSelectedInnerBlock( clientId, true )` once per rendered block. With 1,000 blocks selected, this triggered 1,000,000 parent walks via `getBlockParents`, whose rememo cache used a linked list causing O(n) cache scans on every miss. Additionally, `isSelectionWithinCurrentSection` passed `undefined` when a block lacked a section parent, forcing a full scan that always returned `false`.

Two targeted optimizations were applied:
1. **Short-circuit section checks:** In `packages/block-editor/src/components/block-list/block.js`, `isSelectionWithinCurrentSection` now evaluates to `false` immediately when `sectionBlockClientId` is falsy, bypassing the expensive deep check.
2. **Ancestor set caching:** In `packages/block-editor/src/store/selectors.js`, a new `getSelectedBlockAncestors` selector builds a `Set` of all selected block ancestors once per selection change. `hasSelectedInnerBlock` now uses this set for deep checks, bypassing the slow `getBlockParents` cache.

Before (deep check):
```js
return selectedBlockClientIds.some( ( id ) =>
	getBlockParents( state, id, true ).includes( clientId )
);
```

After (deep check):
```js
return getSelectedBlockAncestors( state ).has( clientId );
```

The selector change is internal to `@wordpress/block-editor` and does not alter the public contract of `hasSelectedInnerBlock`.

## Contribution

Opened and merged by @Mamaduka with review from @youknowriad. The PR addressed a regression reported in #80953. During review, the team debated adding a `selectAll` performance metric to CI to prevent future regressions. While the metric was included, reviewers noted it increased job runtime from ~34m to ~41m and required reducing sample counts due to the extreme baseline latency, flagging it for future stability tuning. The final diff reflects the reduced sample configuration and the optimized selector logic.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
