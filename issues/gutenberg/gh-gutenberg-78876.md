# #78876: Fix playlist metadata edits recreating player

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @scruffian
- **Labels:** `[Type] Bug`, `[Package] Block library`
- **Merged:** [`57e707b`](https://github.com/WordPress/gutenberg/commit/57e707b9f9f7949f6e446cb28abfe91abfd7f25d)
- **Discussion:** [#78876](https://github.com/WordPress/gutenberg/pull/78876) · 5 comments · 0 reactions

## Summary

Fixes a regression in the Playlist block where editing track metadata (title, artist, or artwork) caused the waveform audio player to destroy and reinitialize on every keystroke. This change decouples the player's lifecycle from non-critical metadata updates by switching to in-place DOM mutations for plain text edits, while preserving full reconstruction only when the audio `src` changes or artwork/artist fields are added or removed. The fix eliminates editor focus loss and visual flicker during metadata entry.

## Impact

- **Block & Theme Developers**: No breaking changes or API updates. The Playlist block's internal waveform integration now handles metadata edits deterministically without requiring external adjustments.
- **Plugin & Headless Developers**: No action required. Existing block variations or REST consumption of playlist attributes will automatically benefit from the stabilized editor behavior.
- **Site Owners/Editors**: Restores reliable focus retention in the track list and prevents the waveform player from reinitializing when typing track details.

## Technical details

The diff modifies the Playlist block's waveform initialization effect to separate playback reconstruction from metadata reactivity. Previously, a dependency on display metadata triggered a teardown/reinit cycle on every keystroke. The updated logic now triggers recreation exclusively when the audio `src` changes or when artist/image presence toggles. All other metadata edits bypass destruction and instead call `updatePlayerMetadata()` to mutate existing DOM elements in place. Additionally, the `onEnded` event callback is kept current via the `useEvent` hook, removing render-phase ref assignments and allowing cleanup of a stale ESLint suppression.

## Contribution

Opened and merged by @scruffian, with co-authorship from @jeryj. The PR targets a specific editor experience bug where metadata edits disrupted playback UI. The review process focused on validating the in-place update logic against full teardown/reinit strategies, settling on the refined `updatePlayerMetadata` approach as implemented. Merged as commit `57e707b`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
