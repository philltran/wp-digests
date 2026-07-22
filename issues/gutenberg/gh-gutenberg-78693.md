# #78693: ESLint, UI, Components: Mark `Tooltip` from `@wordpress/ui` as recommended

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] Editor`, `[Package] Block editor`, `[Package] Edit Post`, `[Package] Edit Site`, `[Package] DataViews`, `[Package] Fields`, `[Package] UI`
- **Merged:** [`44e146e`](https://github.com/WordPress/gutenberg/commit/44e146e917d32097b6d6125c7b1436743c64cd46)
- **Discussion:** [#78693](https://github.com/WordPress/gutenberg/pull/78693) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `Tooltip` component is now officially recommended from `@wordpress/ui` and flagged as not recommended in `@wordpress/components`. This change finalizes an in-repo migration by updating the `use-recommended-components` ESLint rule to warn against legacy imports and cleans up previously added per-import disable directives across the codebase.

## Impact

- **Block editor & UI developers:** The `@wordpress/use-recommended-components` ESLint rule will now trigger a warning when importing `Tooltip` from `@wordpress/components`, directing you to use `@wordpress/ui` instead.
- **Storybook consumers:** The legacy `@wordpress/components` `Tooltip` story now displays a "not recommended" banner with a pointer to the replacement, while the `@wordpress/ui` version is marked as the standard.
- **No action required** for runtime behavior; existing code continues to function. Only linting and documentation visibility change.

## Technical details

The PR updates `packages/eslint-plugin/rules/use-recommended-components.js` to add `Tooltip` to the `@wordpress/ui` allow-list and the `@wordpress/components` deny-list, emitting the message `'Use \`Tooltip\` from \`@wordpress/ui\` instead.'`. Corresponding tests in `packages/eslint-plugin/rules/__tests__/use-recommended-components.js` are adjusted. Storybook metadata in `packages/components/src/tooltip/stories/index.story.tsx` and `packages/ui/src/tooltip/stories/index.story.tsx` are updated to reflect the new status. The diff also strips ~20 `// eslint-disable-next-line @wordpress/use-recommended-components` comments from files across `block-editor`, `boot`, `dataviews`, `edit-post`, `edit-site`, and `editor` now that the rule natively approves the `@wordpress/ui` import.

## Contribution

Opened and merged by @ciampo, with review from @aduth and @mirka. The PR closes out a series of five prior migration PRs that moved internal consumers to the new package. During review, @ciampo briefly considered refactoring `components/Button` to use `ui/Tooltip` first, but @mirka noted it wasn't a blocker, allowing the recommendation flip to ship as planned.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
