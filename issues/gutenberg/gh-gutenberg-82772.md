# #82772: UI: Fix highlighted item indicator in forced-colors mode

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Bug`, `[Focus] Accessibility (a11y)`, `[Package] UI`
- **Merged:** [`6b60dd6`](https://github.com/WordPress/gutenberg/commit/6b60dd6b1dba254b24e93fe00f1ab3f550982b9c)
- **Discussion:** [#82772](https://github.com/WordPress/gutenberg/pull/82772) · 2 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/ui` components `Menu`, `Autocomplete`, `Combobox`, and `Select` regain their highlighted-item indicator in forced-colors (Windows High Contrast) mode inside wp-admin. The existing `Highlight` outline was declared in `@layer wp-ui`, but wp-admin's unlayered `a, div { outline: 0 }` in `common.css` overrode it, so keyboard users saw no indicator. The fix routes the outline through the package's existing global CSS defense bridge.

## Impact

- **Users of forced-colors / High Contrast mode:** Keyboard-highlighted items in these menus and listboxes now show a visible `Highlight` outline in wp-admin. This is an accessibility fix.
- **Plugin & theme developers using `@wordpress/ui`:** No API changes and no action required. The item components gain an extra CSS module class (`defenseStyles.div`), which is only relevant if you override item styling or snapshot class names.
- **Hosting / platform / headless:** Not affected.

## Technical details

The root cause is a cascade-layer conflict: unlayered declarations beat layered ones regardless of specificity, so wp-admin's `div { outline: 0 }` beat the outline declared inside `@layer wp-ui`.

The diff makes two kinds of change:

- **CSS** (`packages/ui/src/menu/style.module.css`, `packages/ui/src/utils/css/item-popup.module.css`): inside the highlighted-item `@media ( forced-colors: active )` block, the direct `outline` declaration is replaced with custom properties consumed by the defense bridge.

```css
/* before */
@media ( forced-colors: active ) {
	outline: 1px solid Highlight; /* menu: var(--wpds-border-width-xs) */
}

/* after */
@media ( forced-colors: active ) {
	--_gcd-div-outline: var(--wpds-border-width-focus) solid Highlight;
	/* menu only: */
	--_gcd-a-outline: var(--wpds-border-width-focus) solid Highlight;
}
```

- **TSX**: `defenseStyles.div` (from `utils/css/global-css-defense.module.css`) is added to the `clsx` class list of `autocomplete/item.tsx`, `combobox/item.tsx`, `select/item.tsx`, `menu/item.tsx`, `menu/checkbox-item.tsx`, `menu/radio-item.tsx`, and `menu/submenu-trigger.tsx`.

The outline width also changes from `1px` (or `--wpds-border-width-xs` in Menu) to `--wpds-border-width-focus`. A `Bug Fixes` entry is added to `packages/ui/CHANGELOG.md`.

## Contribution

The PR closes #82771, a companion issue filed for the same problem. The PR description states it was authored with Claude Code assistance, with the cascade-layer analysis and fix AI-assisted and human-verified. The discussion contains only bot output (bundle size, performance, flaky-test reports, props list); no design debate is recorded.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
