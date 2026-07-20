# #80309: [WP7.1] Fix: Allow icon labels to wrap with word breaks and no ellipsis

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Icon`
- **Merged:** [`594d1ff`](https://github.com/WordPress/gutenberg/commit/594d1ffd8eb1460613dbc1ad5cf643979ad8ec95)
- **Discussion:** [#80309](https://github.com/WordPress/gutenberg/pull/80309) · 3 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

This change updates the CSS for icon labels in the block inserter grid, allowing long labels to wrap onto multiple lines instead of being truncated with an ellipsis. It resolves a UI usability issue where lengthy icon names were cut off in the inserter interface.

## Impact

- **Block editor users & editors**: Icon labels in the inserter grid will now wrap to multiple lines, improving readability for long names.
- **Plugin & theme developers**: No code changes or API adjustments required. This is a default editor UI styling update.
- **Hosting & platform teams**: No configuration or migration steps needed.

## Technical details

The diff modifies `packages/block-library/src/icon/editor.scss`. The `.wp-block-icon__inserter-grid-icons-list-item-title` class previously enforced single-line truncation with `overflow: hidden; text-overflow: ellipsis; white-space: nowrap; width: 100%;`. It now uses `overflow-wrap: break-word;`, enabling multi-line wrapping. This is a pure CSS adjustment with no JavaScript or PHP changes.

**Before:**
```css
.wp-block-icon__inserter-grid-icons-list-item-title {
	/* ... */
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
	width: 100%;
}
```

**After:**
```css
.wp-block-icon__inserter-grid-icons-list-item-title {
	/* ... */
	overflow-wrap: break-word;
}
```

## Contribution

Opened by @t-hamano as a manual cherry-pick of PR #80256 to the `wp/7.1` branch due to CI limitations for forked repositories. Merged in commit `594d1ff`. The change was straightforward with minimal review discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
