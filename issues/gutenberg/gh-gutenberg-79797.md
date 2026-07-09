# #79797: SelectControl: Hard deprecate 40px default size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Breaking Change`, `[Package] Components`, `[Package] Editor`, `[Package] Block library`, `[Package] Format library`, `[Package] Block editor`, `[Package] DataViews`, `[Package] Fields`
- **Merged:** [`36ab58f`](https://github.com/WordPress/gutenberg/commit/36ab58fe0ffc8e07d132205d2c6d4f1a4eafad97)
- **Discussion:** [#79797](https://github.com/WordPress/gutenberg/pull/79797) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `SelectControl` component now makes the 40px default height unconditional, effectively hard-deprecating the previously opt-in `__next40pxDefaultSize` prop. This change completes a broader editor UI standardization campaign by removing legacy size migration helpers and locking the updated layout as the baseline. Plugin and block authors should stop passing the prop to ensure forward compatibility and align with current design system expectations.

## Impact

['- **Block & plugin developers**: The `__next40pxDefaultSize` prop on `<SelectControl />` is hard-deprecated. Developers must audit and remove this prop from all custom or third-party usages to prevent deprecation warnings and ensure seamless upgrades to future WordPress/Gutenberg releases.', '- **Agency/Platform teams**: No immediate configuration or migration steps are required for site runtime, but internal block library updates should clear the stripped prop during routine dependency bumps.', '- **Headless & REST consumers**: No impact; this change is isolated to editor-side component rendering and does not alter REST schema or data endpoints.']

## Technical details

The PR modifies `SelectControl` to remove the 36px fallback migration helpers and make the 40px height unconditional in the base styles and internal `InputBase` wrapper. The diff strips the `__next40pxDefaultSize` prop from dozens of call sites across `@wordpress/block-library` (including `archive`, `form`, `gallery`, `query`, `search`, and `video` blocks) as well as internal wrappers like `TreeSelect`. A deprecated `@ignore` type stub remains on the `SelectControlProps` interface for TypeScript consumers, while ESLint bookkeeping is updated to block future opt-ins. Developers migrating existing code should remove the prop entirely:

## Contribution

Opened and merged by `@mirka` with co-authorship from `@ciampo`, this PR serves as the final cleanup step following an earlier design standardization (referenced in #65751). The change systematically strips the legacy prop across the monorepo, updates ESLint rules to prevent re-introduction, and documents the shift in the `components` package changelog before being merged into the main repository.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
