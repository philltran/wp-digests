# #79938: Playlist Block: Add artwork to play button

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @scruffian
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Playlist`
- **Merged:** [`7fb708f`](https://github.com/WordPress/gutenberg/commit/7fb708f01950147293c163d18dafd74f0bbb90c8)
- **Discussion:** [#79938](https://github.com/WordPress/gutenberg/pull/79938) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Playlist block now supports rendering the current track's cover art directly inside the waveform player's play button. This is controlled by a new `showPlayButtonArtwork` block attribute (defaulting to `false`), which shifts the artwork treatment from the standard track image slot into the play button UI using a CSS variable and an SVG contrast filter.

## Impact

- **Block & theme developers:** No immediate action required. The new `showPlayButtonArtwork` attribute is opt-in and defaults to `false`, preserving existing markup and interactivity behavior.
- **Site owners:** Can enable the feature via the block editor's Tools Panel under "Show track image on play button".
- **Headless & REST consumers:** The block's `block.json` schema now includes the new boolean attribute, which will be available in block serialization and the REST API.

## Technical details

- Adds `showPlayButtonArtwork` (boolean, default `false`) to `packages/block-library/src/playlist/block.json`.
- `packages/block-library/src/playlist/edit.js` registers a `ToolsPanelItem` toggle for the attribute and passes it to the `WaveformPlayer` component.
- `packages/block-library/src/playlist/index.php` removes the previous conditional logic that suppressed `image`/`imageAlt` when `showImages` was false, now always passing track metadata to the interactivity context alongside the new `showPlayButtonArtwork` flag.
- `packages/block-library/src/playlist/view.js` reads `context.showPlayButtonArtwork` via the Interactivity API. When enabled, it suppresses the standard `artwork` prop passed to `loadTrack()` and instead calls the new `setupPlayButtonArtwork()` utility to inject the track image as a CSS variable (`--wp--playlist--play-button-artwork`) on the container. The store also recreates the player instance if the presence of play button artwork changes between tracks to avoid DOM state mismatches.
- `packages/block-library/src/playlist/style.scss` introduces `.waveform-player.has-play-button-artwork` styling that applies the CSS variable as a `background-size: cover` image to `.waveform-btn`, adds a `drop-shadow` filter to the play icon, and forces SVG path fills to `#fff !important` for contrast.
- The `setupPlayButtonArtwork` utility in `packages/block-library/src/utils/waveform-utils.js` handles CSS variable injection, class toggling, and safe URL escaping.

## Contribution

Opened and merged by @scruffian with co-authorship from @jeryj. The PR was split from a larger waveform player enhancement to allow independent review and testing. It includes comprehensive unit tests for the new `setupPlayButtonArtwork` utility, player recreation logic, and CSS variable injection. No major design debates were recorded in the discussion; the implementation was accepted after size and test checks passed.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
