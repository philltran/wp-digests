# #80069: Playlist Block: Allow font size customization

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jeryj
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Playlist`
- **Merged:** [`939e58d`](https://github.com/WordPress/gutenberg/commit/939e58d5e13780c024511e07ccabe29ef7df5341)
- **Discussion:** [#80069](https://github.com/WordPress/gutenberg/pull/80069) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Playlist block now supports custom font size adjustments via the block editor's typography controls. Previously, the block's typography was fixed, but this enhancement enables authors to modify the font size directly in the Styles pane. This brings the Playlist block in line with other core blocks that expose typography customization.

## Impact

- **Site owners & editors:** Can now adjust the font size of the Playlist block using the standard typography panel in the block editor.
- **Plugin & theme developers:** No breaking changes or migration steps required. The change is additive and relies on existing block supports infrastructure.
- **Hosting & platform teams:** No configuration or migration required.
- **Headless & REST consumers:** No impact; the change is purely editorial and rendered via existing CSS classes.

## Technical details

The change modifies `packages/block-library/src/playlist/block.json` to add `typography.fontSize: true` and `__experimentalDefaultControls.fontSize: true` to the block's `supports` object, registering the font size control in the editor UI. To ensure the custom font size applies consistently across the player's waveform elements, `packages/block-library/src/playlist/style.scss` was updated to set `font-size: inherit` on `.waveform-info`, `.waveform-artist`, `.waveform-title`, and `.waveform-time`. The block's documentation (`docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/playlist/README.md`) was updated to reflect the new `typography (fontSize)` support.

## Contribution

Opened and merged by @jeryj with co-authorship from @scruffian. The PR was straightforward, focusing on enabling an existing block support feature and aligning the waveform player's typography with the parent block. No significant design debates or alternative approaches were documented in the discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
