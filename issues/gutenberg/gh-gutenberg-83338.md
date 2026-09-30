# #83338: Icons: Rebalance a few, add other new icons.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jasmussen
- **Labels:** `[Type] Enhancement`, `[Package] Icons`
- **Merged:** [`7c3edd3`](https://github.com/WordPress/gutenberg/commit/7c3edd3e974f9dacb270ccb61d2bcbf0cfc018e2)
- **Discussion:** [#83338](https://github.com/WordPress/gutenberg/pull/83338) · 10 comments · 3 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/icons` package gains three new icons (`fullscreenExit`, `justifySpaceEvenly`, `reaction`) and receives visual redraws of four existing icons: `brush` (pencil → wide paintbrush), `media` (play-button-in-rectangle → camera with music notes), `comment` (optical rebalance), and `plugins` (optical rebalance). The redraws support the ongoing effort to replace Dashicons in the WordPress admin menu, where the previous `media` and `image` icons were too similar to distinguish.

## Impact

- **Plugin & theme developers:** The `media` and `brush` icons now render with substantially different artwork. Any UI that references these slugs via `@wordpress/icons` or `wp_get_icon()` will show the new drawings with no code change required. The `comment` and `plugins` icons are optically rebalanced (slightly shifted geometry, same silhouette).
- **Admin / core consumers:** The `media` icon is the one used for the Media Library menu item in the admin; its new camera-with-music-notes design is what will appear there once the Dashicons replacement lands. The `plugins` icon (in the `core-admin` collection) is similarly updated.
- **No action required** for existing code — all slugs are unchanged. New icons (`fullscreenExit`, `justifySpaceEvenly`, `reaction`) are available for use but are not yet wired into any core UI in this PR.

## Technical details

Three new SVG files are added under `packages/icons/src/library/`:

- `fullscreen-exit.svg` — inward-pointing corner arrows, counterpart to the existing `fullscreen` icon.
- `justify-space-evenly.svg` — two filled rectangles flanking a vertical center line.
- `reaction.svg` — a smiley face (circle, two dot eyes, curved mouth).

Four existing SVGs are rewritten in place:

- `brush.svg`: the old three-path pencil drawing is replaced by a single complex path depicting a wide paintbrush with bristle strokes.
- `media.svg`: the old play-triangle-in-rectangle is replaced by a camera body (rounded rect + lens circle) plus two eighth-note glyphs.
- `comment.svg`: the speech-bubble path is shifted down ~0.75 units and the tail geometry adjusted for optical centering.
- `plugins.svg`: the old puzzle-tab shape is replaced by a new interlocking-blocks path.

In `manifest.json` and `manifest.php`, the three new slugs are registered (without a `collections` array, so they ship to the JS library only and are not exposed via the icons REST API or `wp_get_icon()`). Several existing entries (`arrow-down`, `arrow-up`, `contents`, `plugins`, `replace`, `skip-back`, `skip-forward`, `tab-list`, `tab-panel`) are moved to their correct alphabetical positions. The `plugins` entry retains its `"collections": ["core-admin"]` assignment.

The CHANGELOG records the new icons under **New Features** and the redraws under **Enhancements**.

## Contribution

Opened by @jasmussen as a follow-up to feedback on the Dashicons-replacement PR (wordpress-develop#12270). The original proposal included two *new* icons — `paintbrush` and `multimedia` — alongside the three that shipped. @jameskoster asked whether `multimedia` was necessary or whether the existing `media` icon should simply be updated; @fushar agreed, noting that `media` and `image` were already too similar and that updating in place would avoid touching the core PR. @jasmussen revised the PR to redraw `brush` and `media` in place rather than add new slugs. The final merge credits jasmussen, t-hamano, fushar, and jameskoster.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
