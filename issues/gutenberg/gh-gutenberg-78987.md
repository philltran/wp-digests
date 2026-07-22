# #78987: Icons: Add media control and backup icons

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @scruffian
- **Labels:** `[Type] Enhancement`, `[Package] Icons`, `[Feature] Icons`
- **Merged:** [`1d5c1fe`](https://github.com/WordPress/gutenberg/commit/1d5c1fefd719940843dbaef56691ba40e114c562)
- **Discussion:** [#78987](https://github.com/WordPress/gutenberg/pull/78987) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `@wordpress/icons` package now exports six new media control icons (`pause`, `play`, `repeat`, `replay`, `skip-back`, `skip-forward`) and ships an updated visual design for the existing `backup` icon. These additions provide reusable SVG assets for block and editor components that require standard playback controls, while the refreshed `backup` icon aligns with the new visual style. This expands the available iconography for Gutenberg block development and internal editor UI without requiring custom SVG imports.

## Impact

- **Plugin & theme developers:** No immediate action required. The new icons are now available via `@wordpress/icons` exports.
- **Block developers:** Can now use the new media control icons in block toolbars, settings panels, or UI components without creating custom SVGs.
- **Editor/Platform teams:** The `backup` icon appearance will update automatically in components like the post revisions panel and sidebar navigation details footer once the package is updated.
- **No breaking changes, deprecations, or migration steps.**

## Technical details

The diff adds six new SVG source files to `packages/icons/src/library/` (`pause.svg`, `play.svg`, `repeat.svg`, `replay.svg`, `skip-back.svg`, `skip-forward.svg`) and updates `packages/icons/src/manifest.json` with corresponding `slug`, `label`, and `filePath` entries. The existing `packages/icons/src/library/backup.svg` path data is replaced with a revised vector that shares a circular-arrow base with the new `replay` icon. Running `npm run build --workspace=@wordpress/icons` regenerates the compiled library exports, making the new slugs available as named exports in the `@wordpress/icons` package. No new hooks, filters, or REST schema changes are introduced.

## Contribution

Opened by @scruffian to address issue #76009, the PR initially proposed adding the media control icons alongside a visual refresh for `backup`. During review, @jameskoster suggested redrawing the assets as strokes to align with ongoing icon standardization efforts. @fcoveram agreed to provide stroke-based versions, but @jasmussen clarified that the immediate bottleneck is PR review volume rather than asset redrawing, so the team proceeded with the filled-shape assets to unblock usage. The PR was merged with co-authors including @jeryj, @MaggieCabrera, @getdave, @fcoveram, @jameskoster, and @jasmussen.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
