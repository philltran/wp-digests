# #79942: Playlist: Show album art thumbnails

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @scruffian
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Playlist`
- **Merged:** [`05c519b`](https://github.com/WordPress/gutenberg/commit/05c519b35575a62bf89c9e11d76f3d97454ceb9d)
- **Discussion:** [#79942](https://github.com/WordPress/gutenberg/pull/79942) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The core Playlist block now correctly renders album art thumbnails next to track titles when the "Show images" setting is enabled. Previously, the block stored and exposed artwork metadata but failed to output it in both editor and server-side markup. This merge fixes the rendering gap and introduces an `imageAlt` attribute to improve accessibility for track cover images.

## Impact

- **Site owners & editors:** No migration or configuration changes required. Playlists that previously had album covers selected will now display thumbnails as intended when "Show images" is toggled on.
- **Block & theme developers:** The `core/playlist-track` inner block now consumes a new `showImages` context value from its parent. If you override playlist rendering or inspect `$block->context`, note that track markup now conditionally includes an `<img>` element based on this flag.
- **No breaking changes:** Existing block schemas and REST responses remain compatible; the change is purely additive and aligns with existing metadata.

## Technical details

The diff coordinates context propagation and conditional rendering across `packages/block-library/src/playlist/` and `src/playlist-track/`. The parent `block.json` adds `"showImages": "showImages"` to `providesContext`, while the child `block.json` registers `"showImages"` in `usesContext` and declares a new `imageAlt` attribute. In `edit.js`, the parent passes `image` and `imageAlt` to children only when `showImages !== false`. The server-side renderer `render_block_core_playlist_track()` reads `$block->context['showImages']` and conditionally injects `<img class="wp-block-playlist-track__image" src="..." alt="...">` when enabled. A new utility function `getAlbumCoverAttributes()` in `utils.js` extracts both URL and alt text from media library objects, replacing the previous single-property assignment. SCSS in `style.scss` defines a 50×50 fixed thumbnail with an 8% opacity black border and `object-fit: cover` to prevent visual blending on white backgrounds.

## Contribution

Opened by @scruffian as a bug fix to make the existing "Show images" toggle functional. Co-authored by @jeryj and @fcoveram. During review, @fcoveram suggested adding an 8% opacity border to avoid white album covers floating against light backgrounds; the change was incorporated before merge. The PR landed in commit `05c519b` after straightforward approval with no further iteration required.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
