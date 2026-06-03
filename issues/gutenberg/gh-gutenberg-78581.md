# #78581: Edit Post: Hoist setupEditor to run before root.render

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Performance`, `[Package] Editor`, `[Package] Edit Post`
- **Merged:** [`1c3f7ba`](https://github.com/WordPress/gutenberg/commit/1c3f7baa123ba5fb950ea58b75417ebfb59bb9a7)
- **Discussion:** [#78581](https://github.com/WordPress/gutenberg/pull/78581) · 3 comments · 0 reactions

## Summary

Hoists the `setupEditor()` initialization call in the post editor to execute synchronously before `root.render()`, reducing the initial mount from three React re-renders to one. This optimization ensures the block editor’s data and registry state are fully initialized before the first paint, improving boot performance for single-post editing screens.

## Impact

- **Plugin & theme developers**: No action required. The public editor API surface remains unchanged; initialization now completes earlier in the render cycle.
- **Performance/platform teams**: Expect fewer mount re-renders and faster initial content paint for `edit-post` screens. No breaking changes, deprecations, or migration steps introduced.

## Technical details

In `packages/edit-post/src/index.js`, the `setupEditor()` dispatch has been moved out of its previous callback/effect scope and placed directly before `root.render(<StrictMode>...</StrictMode>)`. The diff adds a guard that fetches the entity record via `select( coreDataStore ).getEntityRecord( 'postType', postId )` and conditionally calls `dispatch( editorStore ).setupEditor( post, initialEdits, settings.template )` only when both `postType` and `postId` exist.

In `packages/editor/src/components/provider/index.js`, the provider's internal call to `setupEditor` is adjusted to recognize the hoisted dispatch, preventing duplicate state writes during the mount phase. The change ensures React commits the DOM tree once the editor store is pre-populated, eliminating intermediate render passes.

## Contribution

PR #78581 was authored and merged by @ellatrix as a targeted performance patch. Discussion focused on eliminating redundant mount cycles caused by late state initialization. With no significant design debate or alternative approaches presented, the change was accepted to streamline the editor boot sequence without altering public interfaces or data contracts.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
