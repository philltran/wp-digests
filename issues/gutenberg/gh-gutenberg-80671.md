# #80671: Background image control: Remove duplicated focus ring

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `CSS Styling`, `Backported to WP Core`
- **Merged:** [`f78ad5e`](https://github.com/WordPress/gutenberg/commit/f78ad5e69ebfc95036c8ba02ad5c3f08ff950301)
- **Discussion:** [#80671](https://github.com/WordPress/gutenberg/pull/80671) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Removes a custom `box-shadow` focus style from the background image button in the Block Editor, which was causing a duplicated focus ring after the `Button` component switched to using native `outline` for focus states. This resolves a minor visual regression in the Block Inspector’s Background panel.

## Impact

- **Block editor users**: Fixes a UI glitch where focusing the background image upload button renders two overlapping focus rings.
- **Plugin & theme developers**: No code changes, API modifications, or migration steps required.
- **Hosting & platform teams**: No action required; this is a client-side editor styling fix.

## Technical details

The change removes the `&:focus { box-shadow: inset 0 0 0 var(--wp-admin-border-width-focus) var(--wp-admin-theme-color); }` rule from `packages/block-editor/src/components/background-image-control/style.scss`. The `Button` component now handles focus visibility via `outline`, making the legacy `box-shadow` redundant and visually conflicting. The compiled output (`build/styles/block-editor/style.css`) shrinks by approximately 2 bytes.

**Before** (removed):
```scss
&:focus {
	box-shadow: inset 0 0 0 var(--wp-admin-border-width-focus) var(--wp-admin-theme-color);
}
```

**After**:
```scss
/* Focus ring is now handled entirely by the Button component's outline */
```

No new hooks, filters, or block.json fields are introduced.

## Contribution

Opened by @t-hamano as part of the broader #80138 initiative to standardize focus ring rendering across the Block Editor. The fix was reviewed, merged, and subsequently cherry-picked to the `wp/7.1` branch for inclusion in the upcoming release. Co-authored with @tyxla. The approach required no debate or alternative consideration, as the removal was a direct consequence of the `Button` component's focus behavior update.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
