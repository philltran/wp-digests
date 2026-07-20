# #80060: Move styles into specific waveform styles dropdown area

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jeryj
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Playlist`
- **Merged:** [`36b37df`](https://github.com/WordPress/gutenberg/commit/36b37dff81f2e6444467704b631b32e3be2a65f2)
- **Discussion:** [#80060](https://github.com/WordPress/gutenberg/pull/80060) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Playlist block’s waveform style is migrating from a block style variation (CSS class) to a dedicated `waveformStyle` block attribute. This moves the control into a dedicated dropdown within the Styles panel, reserving the traditional block styles pane for multi-style combinations and simplifying how the block stores and renders waveform data.

## Impact

- **Plugin & theme developers:** The `styles` array in `block.json` is removed. Any code that reads or writes `className` to control waveform styles (e.g., `is-style-mirror`) must now use the `waveformStyle` attribute.
- **Site owners & end users:** No action required; the control remains accessible in the block inspector.
- **Migration:** Update any custom scripts or server-side rendering that parse `className` for waveform values to read `$attributes['waveformStyle']` instead.

## Technical details

In `packages/block-library/src/playlist/block.json`, the `styles` array is removed and replaced with a `waveformStyle` attribute (`type: string`, `enum: ["bars", "mirror", "line", "blocks", "dots", "seekbar"]`, `default: "bars"`). `edit.js` replaces the previous `className.match( /is-style-([\w-]+)/ )` extraction with direct attribute management and registers a new `<InspectorControls group="styles">` panel containing a `SelectControl` for waveform shapes. `index.php` updates `render_block_core_playlist` to read `$attributes['waveformStyle'] ?? 'bars'` instead of parsing `className`. Additionally, `<InspectorControls.Slot group="styles" />` is moved to the top of `block-inspector/index.js` and `inspector-controls-tabs/styles-tab.js` to ensure the new panel renders first.

## Contribution

Opened and merged by @jeryj with co-authors @scruffian and @t-hamano (commit `36b37df`). During review, @t-hamano cautioned against reordering public API slots solely for a single block; the author acknowledged the oversight in the initial implementation. The final diff consolidates waveform control into a dedicated `waveformStyle` attribute and UI panel, while retaining the inspector slot reordering.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
