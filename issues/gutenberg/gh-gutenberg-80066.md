# #80066: Second click or space/enter keypress on playing track pauses it

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jeryj
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Playlist`
- **Merged:** [`a65fc3e`](https://github.com/WordPress/gutenberg/commit/a65fc3eb9c9c4d105d8fbcc38585d75ff390963a)
- **Discussion:** [#80066](https://github.com/WordPress/gutenberg/pull/80066) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Playlist block now toggles playback on the currently playing track when clicked again or activated via keyboard, instead of restarting it. The track button’s screen-reader text dynamically updates between “Play” and “Pause” to reflect the active state. This aligns the block’s behavior with standard media player expectations and improves keyboard accessibility.

## Impact

- **Block users & frontend developers**: The Playlist block’s interactive behavior changes; clicking or pressing Enter/Space on a playing track now pauses it. No code changes or migrations are required.
- **Plugin & theme developers**: No breaking changes to public APIs, hooks, or block.json schemas. Existing implementations remain compatible.
- **Accessibility**: Screen-reader announcements and button labels now accurately reflect play/pause state, reducing confusion for keyboard and assistive technology users.

## Technical details

The change modifies the `core/playlist` store in `packages/block-library/src/playlist/view.js`. A new `isPlaying` flag is added to the block’s `data-wp-context` (rendered in `packages/block-library/src/playlist/index.php` alongside `labelPauseTrack` and `labelSelectTrack`). The `changeTrack` action now checks if `context.currentId === context.trackId`; if true, it retrieves the active player instance from `playlistPlayerState`, toggles `context.isPlaying`, and calls `player.pause()` or `player.play()` directly. State sync is handled via `waveformplayer:play`, `waveformplayer:pause`, and `waveformplayer:ended` event listeners attached to the player container. The track button’s screen-reader text in `packages/block-library/src/playlist-track/index.php` and `packages/block-library/src/playlist-track/edit.js` now uses `data-wp-text="state.trackButtonActionLabel"` to dynamically render “Play” or “Pause”. A new `playlistPlayerState` Map tracks the active player instance for cleanup and reuse, while `playerState` remains a `WeakMap` for per-ref tracking.

## Contribution

Opened and merged by @jeryj (commit `a65fc3e`). @scruffian suggested pushing the toggle logic back to the library, but the implementation remained scoped to the Playlist block. The PR resolved merge conflicts via Copilot and shipped with a +192 B size increase. No major design debates or alternative approaches were documented in the discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
