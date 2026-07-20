# #80087: Editor: use the DS focus color for all sidebar elements

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @fushar
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `[Package] Edit Site`
- **Merged:** [`3e79f5e`](https://github.com/WordPress/gutenberg/commit/3e79f5e072a0cf1fda182a7a653ec9f59825ebf7)
- **Discussion:** [#80087](https://github.com/WordPress/gutenberg/pull/80087) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request standardizes the keyboard focus ring color across all editor sidebar elements by replacing the legacy `--wp-admin-theme-color` CSS variable with the new Design System token `--wpds-color-stroke-focus`. The change ensures focus indicators remain visually consistent and accessible across themed surfaces like the Site Editor navigation and the block list view. It also resolves a visual regression where the block list view's three-dot menu displayed a duplicate focus ring.

## Impact

- **Plugin & theme developers:** No code changes required. The focus ring is now driven by the WPDS token, which may adapt to custom admin themes or dark mode if they override `--wpds-color-stroke-focus`.
- **Site editors & keyboard users:** Improved visual consistency and accessibility when navigating sidebar menus and the block list view.
- **No action required** for existing plugins, themes, or hosting environments. This is an internal UI polish and accessibility fix within the block editor.

## Technical details

The diff modifies two SCSS files in the Gutenberg monorepo:
- `packages/block-editor/src/components/list-view/style.scss`: Replaces `var(--wp-admin-theme-color)` with `var(--wpds-color-stroke-focus)` in three `box-shadow` declarations for focus rings. It also removes `.block-editor-list-view-block__menu:focus` from the selector targeting the three-dot menu, eliminating the duplicate focus ring since the `Button` component already handles its own focus state.
- `packages/edit-site/src/components/sidebar-navigation-item/style.scss`: Adds a `--wp-components-color-accent` override inside `&:focus-visible` that maps to `var(--wpds-color-stroke-focus)`, ensuring the `<Item>` component's focus style aligns with the new token until the component itself migrates.

Before/after example for the list view focus ring:
```css
/* Before */
box-shadow:
  inset 0 0 0 1px $white,
  0 0 0 var(--wp-admin-border-width-focus)
  var(--wp-admin-theme-color);

/* After */
box-shadow:
  inset 0 0 0 1px $white,
  0 0 0 var(--wp-admin-border-width-focus)
  var(--wpds-color-stroke-focus);
```

## Contribution

Opened and merged by @fushar, with co-authorship from @ciampo. The PR was a direct follow-up to review comments on PR #80029, addressing a discussion about standardizing focus tokens across the editor. The implementation applied the new WPDS token directly to the affected SCSS files without introducing new components, hooks, or configuration options.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
