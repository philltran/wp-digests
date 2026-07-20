# #80254: DataViews: Fix the unintended gap between `list` layout items when `groupBy` is set

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Bug`, `Backport to Gutenberg RC`, `[Feature] DataViews`, `Backported to WP Core`, `[Package] DataViews`
- **Merged:** [`98ef602`](https://github.com/WordPress/gutenberg/commit/98ef60236e75d9eb8e7ab12bb26bcf85ea4efc60)
- **Discussion:** [#80254](https://github.com/WordPress/gutenberg/pull/80254) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request resolves a UI regression in the DataViews `list` layout where an unintended extra vertical gap appeared between grouped items when `groupBy` was enabled. The fix removes the `gap="sm"` prop from the group wrapper `<Stack>` component and increases the group header margin to preserve the intended visual spacing. This restores consistent row spacing in admin list views that use grouping.

## Impact

- **Plugin & theme developers / UI engineers:** No code changes required. The correction is internal to the `@wordpress/dataviews` package.
- **WordPress admin users:** List views with `groupBy` enabled (e.g., Media library, Posts list) will no longer show inconsistent vertical spacing between grouped items.
- **No action required.** The change is a transparent UI correction shipped in the DataViews package.

## Technical details

The diff modifies `packages/dataviews/src/components/dataviews-layouts/list/index.tsx` and `packages/dataviews/src/components/dataviews-layouts/list/style.scss`. In `index.tsx`, the `<Stack>` component wrapping `groupItems` previously received `gap="sm"`, which compounded with the existing header margin to create an unintended gap. The prop is removed:

```tsx
// Before
<Stack direction="column" key={ groupName } gap="sm">
// After
<Stack direction="column" key={ groupName }>
```

To compensate and maintain proper separation between groups, the margin on the `.dataviews-view-list__group-header` class in `style.scss` is updated from `var(--wpds-dimension-gap-sm)` to `var(--wpds-dimension-gap-lg)`. The change is scoped to the DataViews list layout and does not affect grid or table layouts.

## Contribution

Opened by @ntsekouras and merged as `98ef602`, with co-authors @ciampo, @jasmussen, and @priethor. The fix continues work from PR #80105 and was cherry-picked to the `wp/7.1` branch by @t-hamano to ensure inclusion in the upcoming release. No major design debates were recorded; the change was validated via Storybook and accepted as a straightforward regression fix.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
