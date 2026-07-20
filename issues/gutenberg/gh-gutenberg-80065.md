# #80065: Playlist Block: add waveform and waveform background color options

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jeryj
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Playlist`
- **Merged:** [`59ea392`](https://github.com/WordPress/gutenberg/commit/59ea392bad5a7828432d879e6d1c712e31f96ad5)
- **Discussion:** [#80065](https://github.com/WordPress/gutenberg/pull/80065) · 10 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Playlist block now supports customizing the waveform and waveform background colors, including solid colors and gradients. This enhancement adds four new block attributes and corresponding UI controls in the Block Editor, allowing theme authors and content creators to apply granular color styling to the audio player's visual waveform without writing custom CSS.

## Impact

- **Block & theme developers:** No breaking changes. The new attributes are opt-in and default to `undefined`. Existing playlists render unchanged.
- **Site owners/content creators:** Can now set solid colors or gradients for the waveform and its background directly in the Block Editor's Styles tab.
- **Headless & REST consumers:** The block's `block.json` schema now includes the four new string attributes. REST API responses will reflect these values if set.
- **No action required** for existing sites, plugins, or themes.

## Technical details

- **Attributes:** Added to `packages/block-library/src/playlist/block.json`: `waveformColor`, `waveformGradient`, `waveformBackgroundColor`, `waveformBackgroundGradient` (all `type: "string"`).
- **Editor UI:** `packages/block-library/src/playlist/edit.js` now uses `__experimentalUseMultipleOriginColorsAndGradients` and `__experimentalColorGradientSettingsDropdown` to render color/gradient pickers grouped under the waveform panel. It manages mutual exclusivity between color and gradient values via local state (`waveformColorGradientChange`, `waveformBackgroundColorGradientChange`).
- **Frontend rendering:** `packages/block-library/src/playlist/index.php` conditionally appends `data-waveform-player-color`, `data-waveform-player-gradient`, `data-waveform-player-background-color`, and `data-waveform-player-background-gradient` attributes to the `.wp-block-playlist__waveform-player` wrapper.
- **JS Player:** `packages/block-library/src/playlist/view.js` reads these data attributes and passes them to the `WaveformPlayer` component. The component recreates the player instance when color/gradient props change, or applies styles via `applyWaveformPlayerStyles` for background changes.
- **CSS:** `style.scss` updates `.waveform-btn` and `.waveform-container` focus styles to use `outline: revert` for better keyboard navigation. `editor.scss` adds layout rules for the new color controls.
- **Note:** Review acknowledged that gradients/colors do not currently work with CSS variables/functions, but this was accepted as an edge case and not treated as a blocker.

## Contribution

Opened by @jeryj, with significant implementation and UI refinement by @scruffian and @fcoveram. The PR consolidated separate Typography and Background panels into a single Colors pane per team feedback. @scruffian added gradient support and fixed keyboard focus style regressions. The approach was merged after brief review, with the CSS variable limitation acknowledged but not treated as a blocker.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
