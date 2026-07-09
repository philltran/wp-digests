# #79834: Playlist: seek value text localization

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jeryj
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Playlist`
- **Merged:** [`c4e7e5b`](https://github.com/WordPress/gutenberg/commit/c4e7e5b2553b8e98586797ec1d6d44fe0ceb148f)
- **Discussion:** [#79834](https://github.com/WordPress/gutenberg/pull/79834) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Upgrades the `@arraypress/waveform-player` dependency to v1.20.0 and migrates Playlist seek slider localization entirely into the upstream library. The change eliminates a previous client-side workaround that intercepted time updates to inject an English "X of Y" string, replacing it with a native template interpolation that properly outputs localized `aria-valuetext` for screen readers.

## Impact

["**Plugin & theme developers**: No action required. The Playlist block's server-side rendering and JS interactivity remain unchanged from the developer's perspective.", '**Accessibility compliance**: Improves keyboard navigation and screen reader feedback for the waveform seek slider, ensuring `aria-valuetext` dynamically reflects the translated time format instead of hardcoded English strings or fragile mocks.', 'Relies on upstream `@arraypress/waveform-player` v1.20.0+ introducing native support for reading `data-seek-value-text` and handling interpolation.']

## Technical details

The diff upgrades `@arraypress/waveform-player` in `packages/block-library/package.json` and `package-lock.json` to `^1.20.0`. In `packages/block-library/src/playlist/index.php`, the Playlist block now explicitly passes `$label_seek_value` (formatted as `'%1$s of %2$s'`) via a new `data-label-seek-value` attribute on the `.wp-block-playlist__waveform-player` container. 

In `packages/block-library/src/playlist/view.js`, `initWaveformPlayer()` forwards this label and a new `seekValueText` string to `createWaveformContainer()`. The updated `createWaveformContainer()` in `packages/block-library/src/utils/waveform-utils.js` reads the `seekValueText` option and sets it as `data-seek-value-text` on the DOM element. Consequently, the library natively interpolates live current time and total duration into this template for the seek slider's `aria-valuetext`, removing the need for client-side time interpolation or monkey-patched event listeners previously used to approximate localization.

## Contribution

Opened by @jeryj as a follow-up to #78980, merged with co-author @scruffian. The PR closes a gap left by an earlier attempt to localize the Playlist's waveform controls by leveraging a recently merged upstream feature in `@arraypress/waveform-player`. Co-authors verified the dependency bump and added corresponding unit test updates for `waveform-utils.js` alongside new e2e specs validating keyboard focus, playback state, and localized `aria-valuetext` output.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
