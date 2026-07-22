# #79093: Components, DataViews: Adopt --wpds-dimension-size-* tokens

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @shrivastavanolo
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] DataViews`, `[Package] UI`
- **Merged:** [`8ff27f2`](https://github.com/WordPress/gutenberg/commit/8ff27f25a72ad14740ee8fa9aff7724a8d4f728d)
- **Discussion:** [#79093](https://github.com/WordPress/gutenberg/pull/79093) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg `components` and `dataviews` packages replace hardcoded pixel values, Sass variables, and `calc(var(--wpds-dimension-base) * N)` expressions with the new `--wpds-dimension-size-*` CSS custom properties. This change standardizes sizing across UI components and data view layouts, aligning them with the WordPress Design System token scale. No visual changes are intended; the update purely modernizes the underlying CSS architecture.

## Impact

- **Plugin & theme developers:** No breaking changes or required code modifications. The replaced Sass variables and `calc()` expressions were internal to the packages.
- **Block developers using DataViews:** Layouts that rely on default component sizing will automatically inherit the new tokens. External consumers are unaffected unless they were directly overriding internal package styles with hardcoded values.
- **Hosting & platform teams:** No action required. This is a standard internal CSS refactoring within the Gutenberg monorepo.

## Technical details

The diff replaces legacy sizing patterns across `packages/components/` and `packages/dataviews/` with WPDS tokens:
- `calc(var(--wpds-dimension-base) * 4)` → `var(--wpds-dimension-size-2xs)` (16px)
- `calc(var(--wpds-dimension-base) * 6)` → `var(--wpds-dimension-size-sm)` (24px)
- `calc(var(--wpds-dimension-base) * 8)` → `var(--wpds-dimension-size-md)` (32px)
- `$button-size-small` and `$icon-size` Sass variables → `var(--wpds-dimension-size-sm)`
- Additional tokens applied: `var(--wpds-dimension-size-3xs)` (12px) and `var(--wpds-dimension-size-4xs)` (8px) for activity/picker layouts.

Before/after example from `packages/components/src/button/style.scss`:
```scss
// Before
&.is-small {
	height: $button-size-small;
	&.has-icon:not(.has-text) { min-width: $button-size-small; }
}
// After
&.is-small {
	height: var(--wpds-dimension-size-sm);
	&.has-icon:not(.has-text) { min-width: var(--wpds-dimension-size-sm); }
}
```
The change removes `// TODO: use size token when available` comments and eliminates the `@use "@wordpress/base-styles/variables" as vars;` import in `packages/dataviews/src/components/dataviews-layouts/list/style.scss`. No new public APIs, hooks, or REST routes are introduced.

## Contribution

Opened by @shrivastavanolo as part of the broader WPDS token adoption effort. Reviewer @ciampo provided a detailed audit of additional replacement candidates across `packages/ui/` and dashboard routes, which the author addressed in follow-up commits. @jameskoster noted that the `Tabs` component’s `48px` height lacks a matching token, prompting the team to defer that specific case to a dedicated issue rather than forcing a mismatched replacement. The PR was rebased to resolve a `CHANGELOG.md` conflict and merged with co-authors @ciampo and @jameskoster.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
