# #81176: Edit Widgets: Fix header toolbar button focus ring

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Bug`, `[Package] Edit Widgets`, `Backported to WP Core`
- **Merged:** [`95ca3f0`](https://github.com/WordPress/gutenberg/commit/95ca3f044ea3625e4673ee2edc5a8092da56f47e)
- **Discussion:** [#81176](https://github.com/WordPress/gutenberg/pull/81176) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Edit Widgets screen header toolbar icon buttons now use the standard design system focus ring instead of a legacy inset box-shadow. This aligns the widgets editor’s keyboard navigation and visual consistency with the rest of the block editor. The change replaces custom inline CSS with the `outset-ring__focus` mixin and adjusts the focus selector to match standard button behavior.

## Impact

- **Theme & plugin developers**: No code changes required. The fix is scoped entirely to the core widgets editor UI.
- **Site owners & editors**: Improved keyboard navigation consistency when tabbing through the widgets screen toolbar.
- **Hosting & platform teams**: No configuration or migration steps needed; the change ships as a CSS update in the next WordPress release.

## Technical details

The diff modifies `packages/edit-widgets/src/components/header/style.scss`. It replaces the previous `&:focus:not(:disabled)` rule with `&:focus:not(:active)` and swaps the custom `box-shadow`/`outline` declarations for the `@include outset-ring__focus();` mixin. This matches the selector and styling pattern used by the base `components-button` component.

```scss
/* Before */
&:focus:not(:disabled) {
  box-shadow: 0 0 0 var(--wp-admin-border-width-focus) var(--wp-admin-theme-color), inset 0 0 0 $border-width $white;
  outline: 1px solid transparent;
}

/* After */
&:focus:not(:active) {
  @include outset-ring__focus();
}
```

## Contribution

Opened to resolve a visual regression where header toolbar buttons reverted to legacy focus styles after being reset to standard `Button` components. During review, it was noted that the fix temporarily reverts a custom color scheme override, but rebasing it on top of a follow-up PR (#81173) yields the correct final styles. The change was subsequently cherry-picked to the `wp/7.1` branch for inclusion in the upcoming release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
