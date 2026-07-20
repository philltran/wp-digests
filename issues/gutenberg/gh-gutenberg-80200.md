# #80200: Playlist: Fix track insertion

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @scruffian
- **Labels:** `[Type] Bug`, `[Package] Block library`, `Backport to Gutenberg RC`, `Backported to WP Core`
- **Merged:** [`9e34936`](https://github.com/WordPress/gutenberg/commit/9e34936b8ea576f16a4c764bb5119dcee702153d)
- **Discussion:** [#80200](https://github.com/WordPress/gutenberg/pull/80200) · 12 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Playlist block’s track insertion flow has been refactored to replace the inline placeholder appender with toolbar-based “Add” controls and a drag-and-drop zone. This prevents the persistent, non-dismissable media placeholder that previously cluttered the block canvas. The change also introduces toolbar controls to individual Playlist Track blocks, supports local audio file uploads via temporary blob URLs, deduplicates appended tracks, and ensures child track blocks remain mounted when the track list is visually hidden.

## Impact

- **Editors & Content Creators:** No more stuck inline placeholders when adding tracks; track addition is now handled via toolbar buttons and drag-and-drop.
- **Plugin & Theme Developers:** No breaking changes to block attributes, frontend rendering, or public APIs. The `PlaylistContext` now exposes an internal `addTracks` callback, but it is not a public hook.
- **Hosting & Platform Teams:** No configuration or migration required.
- **Action Required:** None.

## Technical details

In `packages/block-library/src/playlist/edit.js`, `renderAppender` is explicitly set to `false`, removing the inline `InnerBlocks.ButtonBlockAppender`. A new `onAddTracks` callback replaces `onSelectTracks` for the main `MediaReplaceFlow` (renamed to “Add”), filtering duplicates by comparing `id`, `src`, or `blob` via `getTrackIdentifier`. Local `File` objects are now handled by `createTrackBlocks`, which generates temporary blob URLs using `createBlobURL` from `@wordpress/blob` until upload completes. The tracklist `<ol>` is always rendered but conditionally styled with `wp-block-playlist__tracklist-is-hidden` instead of being unmounted, preserving inner block state.

In `packages/block-library/src/playlist-track/edit.js`, `addTracks` is consumed from `PlaylistContext` and rendered in a new `BlockControls` group (`group="block"`) via `MediaReplaceFlow` with `variant="toolbar"`. The `useUploadMediaFromBlobURL` hook now receives `url` instead of `src`.

```diff
// Before: inline appender rendered conditionally
renderAppender: hasAnySelected && InnerBlocks.ButtonBlockAppender,

// After: appender disabled, tracklist always mounted but visually hidden
renderAppender: false,
// ...
<ol className={ clsx( 'wp-block-playlist__tracklist', {
    'wp-block-playlist__tracklist-is-hidden': ! showTracklist,
} ) }>
```

`packages/block-library/src/playlist/context.js` adds `addTracks: undefined` to the context default, and `packages/block-library/src/playlist/editor.scss` removes the legacy `.block-list-appender` positioning rules.

## Contribution

Opened and merged by @scruffian, with co-authors @jeryj, @getdave, @noruzzamans, @jasmussen, and @fcoveram. Review feedback from @jeryj led to adding drag-and-drop support on the populated playlist block and toolbar controls for individual track blocks. @scruffian noted that `__experimentalExposeControlsToChildren` was considered for sharing controls but rejected because it would pass all controls down to children. The change was backported to WordPress Core.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
