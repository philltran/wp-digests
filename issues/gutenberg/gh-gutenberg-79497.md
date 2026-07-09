# #79497: Playlist block: Avoid laggy layout shift when changing tracks

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mcsf
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Playlist`
- **Merged:** [`f2ad022`](https://github.com/WordPress/gutenberg/commit/f2ad0228ff6aeb01b523e01bd18abe88d16603ba)
- **Discussion:** [#79497](https://github.com/WordPress/gutenberg/pull/79497) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Playlist block's editor component no longer destroys and recreates the underlying `@arraypress/waveform-player` instance when track metadata or the audio source changes. This eliminates a visible layout shift and rendering jank that occurred while editing track titles, artists, or switching tracks in the block editor.

## Impact

- **Block developers & theme authors:** No public APIs, props, or hooks changed; the `WaveformPlayer` component's external interface remains fully backward compatible.
- **Editor UX & plugin/theme maintainers:** Editors will experience smoother interactions when updating playlist track metadata or switching between tracks in the block editor, with no visible flicker or layout jump.
- **Action required:** None. This is an internal rendering optimization to the Playlist block's lifecycle and requires no migration or code changes.

## Technical details

In `packages/block-library/src/utils/waveform-player.js`, the `WaveformPlayer` component refactors its dependency tracking and lifecycle hooks. Previously, changes to `src`, `title`, `artist`, or `image` triggered a `destroy()` call on the existing player instance followed by a full reinitialization via `initWaveformPlayer()`, causing a layout shift. The diff replaces this with two targeted `useEffect` hooks: one updates metadata in-place via `updatePlayerMetadata()`, and the other calls the existing instance's `loadTrack()` method when the `src` prop changes. This approach keeps the player instance alive across re-renders, eliminating the destruction/recreation cycle that caused the UI jank. The corresponding test suite (`waveform-player.js`) was updated to mock `loadTrack()` and assert that `destroy` is no longer called when the source changes.

## Contribution

Opened and merged by @mcsf with co-authorship from @jeryj. The PR addressed a follow-up to #79448 after functional testing confirmed the smoother interaction. During review, @jeryj noted conflicts requiring a rebase (completed by @mcsf) and discussed an upstream metadata-swapping fix in `@arraypress/waveform-player` (#14). Auto-merge was initially blocked on a protected ref but proceeded successfully after rebasing.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
