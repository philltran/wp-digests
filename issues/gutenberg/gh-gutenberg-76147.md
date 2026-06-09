# #76147: Playlist Block: Add visualization style selector

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @scruffian
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Playlist`
- **Merged:** [`cab6c21`](https://github.com/WordPress/gutenberg/commit/cab6c219be9e3b5c795178a346379d0442e9bd1a)
- **Discussion:** [#76147](https://github.com/WordPress/gutenberg/pull/76147) · 5 comments · 0 reactions

## Summary

The Playlist block now exposes a “Visualization style” dropdown in the block inspector, enabling editors to control how audio waveforms are rendered. This change surfaces all six waveform visualization styles supported by the `@arraypress/waveform-player` library while preserving backwards compatibility through a default “bars” value.

## Impact

- **Content Editors**: Can now switch waveform visuals directly in the block sidebar without code changes. Existing playlists automatically retain the default “bars” style.
- **Plugin & Theme Developers**: No breaking changes or API deprecations. The new `visualizationStyle` attribute is registered natively in the block schema, requiring no migration scripts.
- **Headless/REST Consumers**: The block now exposes a contextual attribute via the Interactivity API, which may be leveraged by custom frontend renderers or headless consumers reading block context.

## Technical details

- Registers `visualizationStyle` (string, default: `"bars"`) in `block.json`.
- **Editor (`edit.js`)**: Adds a `SelectControl` dropdown within `InspectorControls`. The selected value is threaded as a prop to the `<WaveformPlayer>` component.
- **Interactivity API Thread**: `index.php` serializes the attribute into `data-wp-context`. Frontend logic in `view.js` reads this context and passes it to `@arraypress/waveform-player`.
- **Utility Updates**: `waveform-player.js` and `waveform-utils.js` update their signatures: `initWaveformPlayer()` and `createWaveformContainer()` now accept a `visualizationStyle` parameter that is forwarded downstream.
- **Pattern Shift**: The block previously relied on an implicit or hardcoded waveform mode. Contextual threading now ensures editor selections persist to the frontend:
  ```javascript
  // index.php (excerpt)
  'data-wp-context', json_encode( array( 'playlist' => array( 'visualizationStyle' => $attributes['visualizationStyle'] ?? 'bars' ) ) )

  // view.js (excerpt)
  const style = el.dataset.wpContext?.playlist?.visualizationStyle || 'bars';
  initWaveformPlayer(container, { visualizationStyle: style });
  ```
- Updates `core-blocks.md` to reflect the new attribute in the schema documentation.

## Contribution

Merged as commit `cab6c219be9e3b5c795178a346379d0442e9bd1a` with co-authors @scruffian, @jeryj, and @talldanwp. The PR builds directly on the architectural pattern established in #75060, focusing specifically on exposing the underlying library’s visualization options rather than altering core player logic. Validation centered on inspector UX, context threading reliability, and default style fallback behavior.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
