# #80104: Fix playlist waveform artist rendering

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @scruffian
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Playlist`
- **Merged:** [`2b6da42`](https://github.com/WordPress/gutenberg/commit/2b6da4250bb8052cd706fea789f039abbcc86408)
- **Discussion:** [#80104](https://github.com/WordPress/gutenberg/pull/80104) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Playlist block's waveform player now correctly renders artist metadata. Previously, the block wrapper passed artist data via `data-subtitle` and referenced `subtitleEl`, but the underlying `@arraypress/waveform-player` package expects `data-artist` and exposes `artistEl`. This fix aligns the Playlist block's data attributes and DOM references with the waveform player's API, ensuring artist names display correctly under track titles in the editor and on the front end.

## Impact

- **Block & theme developers:** No public API deprecations or breaking changes to JavaScript/PHP interfaces. However, custom CSS or JS that targets `.waveform-subtitle` inside the Playlist block will need to be updated to `.waveform-artist`.
- **Site owners & editors:** Artist names will now render correctly in the Playlist block's waveform player UI without workarounds.
- **Hosting & platform teams:** No action required. The change is a contained bug fix with a negligible bundle size impact (-33 B).
- **Headless & REST consumers:** No impact; the change is purely client-side rendering logic within the block's view layer.

## Technical details

The diff updates the Playlist block's waveform utilities to match the `@arraypress/waveform-player` package's expected attribute and DOM element names:
- `packages/block-library/src/utils/waveform-utils.js`: Changes `container.setAttribute( 'data-subtitle', artist )` to `data-artist`.
- `packages/block-library/src/utils/waveform-player.js`: Renames `instance.subtitleEl` to `instance.artistEl` and updates the `updatePlayerMetadata` function to write artist text to `artistEl` instead of `subtitleEl`.
- `packages/block-library/src/playlist/style.scss`: Updates the SCSS selector from `.waveform-subtitle` to `.waveform-artist`.
- Test files (`waveform-player.js`, `waveform-utils.js`) are updated to reflect the new naming convention.

Before/after pattern for metadata updates:
```js
// Before
if ( instance.subtitleEl ) {
    instance.subtitleEl.textContent = artist ?? '';
    instance.subtitleEl.style.display = artist ? '' : 'none';
}
// After
if ( instance.artistEl ) {
    instance.artistEl.textContent = artist ?? '';
    instance.artistEl.style.display = artist ? '' : 'none';
}
```

## Contribution

Opened and merged by @scruffian, with co-authorship from @jeryj. The PR was prepared with AI assistance (OpenAI Codex) and reviewed internally. The discussion centered on a CI linting failure that occurred after the merge was triggered, which was resolved in a follow-up commit (#80124). No alternative approaches were debated; the fix directly aligns the block's implementation with the upstream waveform player package API.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
