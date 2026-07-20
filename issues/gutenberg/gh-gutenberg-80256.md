# #80256: Fix: Allow icon labels to wrap with word breaks and no ellipsis

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @milindmore22
- **Labels:** `[Type] Bug`, `[Package] Block library`, `First-time Contributor`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Block] Icon`
- **Merged:** [`330bd7b`](https://github.com/WordPress/gutenberg/commit/330bd7bd4bc83a844a4dd6430d2da560f07f6f09)
- **Discussion:** [#80256](https://github.com/WordPress/gutenberg/pull/80256) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request fixes a UI bug in the Icon block inserter where long custom icon labels were truncated with an ellipsis instead of wrapping. The change updates the editor stylesheet to allow text to wrap within its container, improving readability for custom icon sets with descriptive names.

## Impact

- **Block & theme developers**: No code changes required. The fix applies automatically to the Icon block inserter UI in the editor.
- **Site owners & editors**: Long custom icon labels will now wrap instead of being cut off, making them easier to read.
- **No action required** for existing sites, plugins, or themes.

## Technical details

The change modifies `packages/block-library/src/icon/editor.scss`. It replaces the `.wp-block-icon__inserter-grid-icons-list-item-title` rule's `overflow: hidden`, `text-overflow: ellipsis`, `white-space: nowrap`, and `width: 100%` declarations with `overflow-wrap: break-word`. This allows the label text to wrap naturally within the grid item container.

```css
/* Before */
.wp-block-icon__inserter-grid-icons-list-item-title {
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
	width: 100%;
}

/* After */
.wp-block-icon__inserter-grid-icons-list-item-title {
	overflow-wrap: break-word;
}
```

## Contribution

Opened by @milindmore22 and merged after review by @jasmussen and @t-hamano. Reviewers suggested applying the smallest allowed font size and removing the semibold weight, but noted the font weight is tied to design tokens and deferred to a follow-up. The change was manually backported to the `wp/7.1` branch via #80309.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
