# #78954: Add playlist track length setting

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @scruffian
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Playlist`
- **Merged:** [`d46d47c`](https://github.com/WordPress/gutenberg/commit/d46d47cda4d37846c7e88176b1625f9e7239dde8)
- **Discussion:** [#78954](https://github.com/WordPress/gutenberg/pull/78954) · 1 comments · 0 reactions

## Summary

The Playlist block now includes a `showTrackLength` attribute that controls whether audio track durations are displayed in the rendered tracklist. A new sidebar toggle allows editors to hide or show lengths without manually editing HTML, extracting this usability improvement from the broader playlist waveform tracking initiative.

## Impact

- **Editors & content authors**: Gain explicit control over track duration visibility via the block settings panel.
- **Theme/Plugin developers**: The parent tracklist wrapper now receives `wp-block-playlist__tracklist-length-is-hidden` when lengths are disabled, enabling targeted CSS overrides or conditional layout adjustments.
- **No migration or breaking changes required**: The attribute defaults to `true`, preserving existing markup and styling for current sites without requiring updates.

## Technical details

- **`packages/block-library/src/playlist/block.json`**: Registers `showTrackLength` as a boolean attribute with a `default` of `true`.
- **`packages/block-library/src/playlist/edit.js`**: Destructures the new attribute and injects a `ToolsPanelItem` wrapping a `ToggleControl` labeled “Show track length in Tracklist”. Conditionally appends `wp-block-playlist__tracklist-length-is-hidden` to the `.wp-block-playlist__tracklist` container via `clsx`.
- **`packages/block-library/src/playlist/save.js`**: Outputs the same visibility class on the frontend wrapper based on the saved attribute.
- **`packages/block-library/src/playlist/style.scss`**: Adds a CSS rule targeting `.wp-block-playlist-track__length` inside the hidden container, applying `display: none;`.
- **`docs/reference-guides/core-blocks.md`**: Appends `showTrackLength` to the documented block attributes list.

## Contribution

Authored by @scruffian, this PR was explicitly extracted from the larger #75060 playlist waveform effort to enable independent review. It underwent minimal reviewer interaction and merged cleanly, focusing strictly on decoupling the length visibility toggle from broader UI tracking work.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
