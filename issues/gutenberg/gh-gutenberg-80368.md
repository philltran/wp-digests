# #80368: Pass Playlist controls to track blocks

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @scruffian
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Backported to WP Core`, `[Block] Playlist`
- **Merged:** [`7b1b49d`](https://github.com/WordPress/gutenberg/commit/7b1b49d6e427c92ccab15567f27be640676baa57)
- **Discussion:** [#80368](https://github.com/WordPress/gutenberg/pull/80368) · 14 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Playlist block now delegates its "Add track" toolbar control to selected Playlist Track child blocks using the existing block toolbar sharing mechanism. Previously, the parent block passed an `addTracks` callback through `PlaylistContext`, requiring each track to render its own control. This change consolidates the control under the parent block's toolbar area, reducing duplication and aligning with the editor's built-in child toolbar capture behavior.

## Impact

- **Plugin & theme developers:** No direct API changes. The `addTracks` context value is removed from `PlaylistContext`, but it was never part of the public API and was only used internally by the block library.
- **Block authors:** If you are building custom blocks that consume `PlaylistContext`, you should remove any references to `addTracks`.
- **Site owners & editors:** No action required. The "Add track" button remains available in the block toolbar when a track is selected.
- **Backport note:** This enhancement has been backported to WordPress Core.

## Technical details

- `packages/block-library/src/playlist/block.json`: Adds `"__experimentalExposeControlsToChildren": true` to the `supports` object.
- `packages/block-library/src/playlist/edit.js`: Adds `__experimentalShareWithChildBlocks` to the `<BlockControls group="other">` wrapper. Removes `addTracks: onAddTracks` from the `PlaylistContext.Provider` value. Updates the button label to `__( 'Add track' )`.
- `packages/block-library/src/playlist-track/edit.js`: Removes `addTracks` from the context destructuring and deletes the conditional `<BlockControls group="block">` that previously rendered `<MediaReplaceFlow>` for adding tracks.
- `packages/block-library/src/playlist/context.js`: Removes `addTracks: undefined` from the default context value.

The change leverages the experimental block toolbar sharing API (`__experimentalExposeControlsToChildren` and `__experimentalShareWithChildBlocks`) to route the parent's `BlockControls` fill to the active child block. Usage pattern changed:

```diff
- const { currentTrackClientId, setCurrentTrackClientId, addTracks } =
-   useContext( PlaylistContext );
+ const { currentTrackClientId, setCurrentTrackClientId } =
+   useContext( PlaylistContext );
```

Unit tests for the removed track-level control were replaced with an E2E test in `test/e2e/specs/editor/blocks/playlist.spec.js` to verify the shared toolbar behavior.

## Contribution

Opened as an alternative to #80246, the PR was reviewed by @jasmussen, who suggested grouping the "Add" and "Replace" controls together and shortening the label to "Add". The author kept the label as "Add track" for clarity and shifted the test strategy, replacing unit tests for the removed track-level control with an E2E test to verify the shared toolbar behavior. The change was merged after a brief review cycle.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
