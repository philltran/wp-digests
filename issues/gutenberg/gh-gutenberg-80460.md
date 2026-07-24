# #80460: Media Editor: Show a loading state while the cropped file loads

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Feature] Media`, `[Package] Block library`, `Backported to WP Core`
- **Merged:** [`ba1392d`](https://github.com/WordPress/gutenberg/commit/ba1392d64d345799ca5217da79aeb4a20fc8490f)
- **Discussion:** [#80460](https://github.com/WordPress/gutenberg/pull/80460) · 9 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

After saving a crop or rotate in the media editor, the Image and Cover blocks now display a dimmed overlay and spinner until the browser finishes loading the newly generated file. This prevents the visual glitch where the old bitmap remains visible during slow downloads, which previously made edits appear to fail. The crop button is disabled rather than hidden during this transition to preserve keyboard focus.

## Impact

- **Site owners / editors:** Smoother UX when cropping or rotating images on slower connections; no broken visual state after saving edits.
- **Plugin & theme developers:** No public API changes. Blocks now use internal state (`isSwappingMedia`) and CSS classes (`is-swapping-media`, updated `is-transient`) to manage the loading phase. Custom block editor markup that relies on immediate DOM swaps after `setAttributes` may need to account for the brief swapping state.
- **Hosting & platform / Headless & REST consumers:** No impact; this is purely a client-side editor UX improvement.

## Technical details

The change introduces a `pendingSwapUrl` state in `packages/block-library/src/image/image.js` and `isSwappingMedia` in `packages/block-library/src/cover/edit/index.js`. The `useOpenImageMediaEditorModal` hook now accepts an `onUrlChange` callback that fires when the media editor returns a new attachment URL. Blocks set the swapping state immediately after `setAttributes`, then clear it via the `<img>` element’s `onLoad`/`onError` handlers (or immediately for Cover blocks using CSS backgrounds, which bypass load events). The crop button is disabled (`disabled={ isSwappingMedia }`) instead of conditionally rendered to maintain focus order. CSS updates in `image/editor.scss` and `cover/editor.scss` apply opacity dimming, z-index stacking for the spinner, and a `:has()` selector to target the swapped image.

## Contribution

Opened to address a visual regression where the media editor modal closed before the browser finished fetching the edited file. An alternative approach—keeping the modal open to preload the file—was discussed but rejected because it merely relocated the download latency and froze the UI. Support for the Site Logo block was deferred due to layout complexity. The PR was merged with co-authors @andrewserong and @jordesign.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
