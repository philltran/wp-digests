# #80312: Fix playlist block styling

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @shimotmk
- **Labels:** `[Type] Bug`, `[Package] Block library`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Block] Playlist`
- **Merged:** [`f5f8f59`](https://github.com/WordPress/gutenberg/commit/f5f8f59ee64e271bf8765ec25023ec40ff999a0b)
- **Discussion:** [#80312](https://github.com/WordPress/gutenberg/pull/80312) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Playlist block now applies `box-sizing: border-box` to its root container, fixing a layout bug where setting padding incorrectly expanded the block's total width. This aligns the Playlist block's sizing behavior with other core blocks and ensures padding behaves predictably in the editor and on the front end.

## Impact

- **Block & theme developers**: No code changes required. The fix applies automatically via the block's default stylesheet.
- **Site owners**: Padding adjustments in the block editor will now correctly add space inside the block rather than expanding its overall width.
- **No action required** for existing sites, custom themes, or headless setups.

## Technical details

The change modifies `packages/block-library/src/playlist/style.scss` by adding `box-sizing: border-box;` to the `.wp-block-playlist` selector. Previously, the block inherited the default `content-box` sizing, causing padding to be added to the element's width calculation. The diff adds a single CSS property to the root container, ensuring padding and borders are included within the specified width.

```css
.wp-block-playlist {
	// This block has customizable padding, border-box makes that more predictable.
	box-sizing: border-box;
	// ...
}
```

## Contribution

Opened by @shimotmk and co-authored with @t-hamano and @noruzzamans, the PR was merged as `f5f8f59` and cherry-picked to the `wp/7.1` branch for inclusion in the upcoming release. The fix was straightforward, addressing a reported sizing inconsistency without requiring architectural changes or alternative approaches.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
