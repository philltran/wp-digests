# #80168: Add Playlist icon.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jasmussen
- **Labels:** `[Type] Enhancement`, `[Package] Icons`
- **Merged:** [`e82ef00`](https://github.com/WordPress/gutenberg/commit/e82ef00f418776ea5c768cb65304ec7d447c06f3)
- **Discussion:** [#80168](https://github.com/WordPress/gutenberg/pull/80168) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds a new `playlist` icon to the `@wordpress/icons` package to support the ongoing stabilization of the Playlist block. The icon is a variant of the existing `audio` icon with added horizontal lines to represent a list.

## Impact

- **Plugin & theme developers:** Can now reference the `playlist` icon slug in `@wordpress/icons` or block editor interfaces.
- **Playlist block users:** Will see the correct icon in the block editor once the block is stabilized.
- No breaking changes, deprecations, or migration steps required.

## Technical details

The PR adds `packages/icons/src/library/playlist.svg` and registers it in `packages/icons/src/manifest.json` with the slug `playlist`. The SVG uses a single `<path>` element with `fill="currentColor"` and draws the base audio shape plus two horizontal lines (`M20 15.75h-5v-1.5h5zM20 10.75h-5v-1.5h5z`) to indicate list items. Review noted a missing `viewbox` attribute, which was acknowledged by the author and deferred to a separate follow-up PR (#80180).

## Contribution

Opened and merged by @jasmussen with co-authorship from @scruffian. The change was reviewed quickly, with @t-hamano flagging a missing `viewbox` attribute that was acknowledged and scheduled for a separate follow-up PR.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
