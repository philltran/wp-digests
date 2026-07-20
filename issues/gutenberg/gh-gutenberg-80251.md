# #80251: Icon block: Show text and background color controls by default.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @juanfra
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Block] Icon`
- **Merged:** [`9dd84c4`](https://github.com/WordPress/gutenberg/commit/9dd84c43605b163cd038b5cb6ab965c92ba500cb)
- **Discussion:** [#80251](https://github.com/WordPress/gutenberg/pull/80251) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Icon block now exposes text and background color controls in the block inspector by default, eliminating the need for editors to manually expand the typography and background panels. This change aligns the Icon block with the established visibility pattern used by other core blocks like `core/paragraph`, improving styling discoverability without altering the underlying color support capabilities.

## Impact

- **Block & theme developers:** No code changes required. The block's `supports` schema remains functionally identical; only the UI visibility defaults have shifted.
- **Site owners & editors:** Color controls for text and background will appear immediately in the Styles sidebar upon inserting an Icon block, reducing clicks to apply styling.
- **Hosting & platform teams:** No configuration or migration steps needed. The change ships as a standard block library update.
- **Action required:** No action required.

## Technical details

The change modifies `packages/block-library/src/icon/block.json` to adjust how color support is declared. Previously, `supports.color` explicitly set `background` and `text` to `true`. The diff replaces these with `__experimentalSkipSerialization: true` and introduces `__experimentalDefaultControls: { background: true, text: true }`. The block editor's typography and background panels already consume `__experimentalDefaultControls` to determine initial panel visibility, so no JavaScript or PHP logic changes are required. The `README.md` documentation is updated to reflect the simplified `color: true` declaration in the supports list.

## Contribution

Opened by @juanfra and merged via commit `9dd84c4`, this enhancement was backported to the `wp/7.1` branch for inclusion in the next WordPress release. The PR received minimal review discussion, with the implementation relying on existing block editor infrastructure to handle the new `__experimentalDefaultControls` flag. Co-authors @t-hamano were credited during the merge process.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
