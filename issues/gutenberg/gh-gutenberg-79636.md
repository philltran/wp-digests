# #79636: ComboboxControl: Hard deprecate 40px default size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Breaking Change`, `[Package] Components`, `[Package] Editor`, `[Package] Block library`, `[Package] Fields`
- **Merged:** [`77d7b3a`](https://github.com/WordPress/gutenberg/commit/77d7b3abfac4eb0c27173990900ddcfac49710ee)
- **Discussion:** [#79636](https://github.com/WordPress/gutenberg/pull/79636) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Hard deprecates the `__next40pxDefaultSize` prop on `ComboboxControl`, making the larger default height unconditional and removing all legacy sizing styles, warning hooks, and opt-in logic. The opt-in period for the upgraded control size is now complete, so the prop is ignored by the component and can be safely stripped from external call sites without altering rendering or behavior.

## Impact

- **Block & Plugin Developers**: Should remove the `__next40pxDefaultSize` prop from any custom instances of `ComboboxControl` to satisfy updated ESLint rules and clean up legacy code. Core functionality remains unchanged.
- **Hosting & Platform Teams**: No action required; this is a component-level API cleanup within the block editor ecosystem.

## Technical details

In `packages/components/src/combobox-control/index.tsx`, the PR removes the `useDeprecated36pxDefaultSizeProp` hook and `maybeWarnDeprecated36pxSize` conditional warning, replacing the destructured props with a direct `props` assignment. The `__next40pxDefaultSize` prop is stripped from `types.ts` (marked `@deprecated` and `@ignore`). Legacy CSS in `styles.ts` that conditionally applied a 28px height is deleted. Internal call sites across the monorepo are updated to drop the flag, including `packages/block-library/src/avatar/user-control.js`, `packages/block-library/src/page-list/edit.js`, `packages/editor/src/components/page-attributes/parent.js`, `packages/fields/src/fields/parent/parent-edit.tsx`, `packages/media-fields/src/attached_to/edit.tsx`, and `packages/validated-form-controls/components/combobox-control.tsx`. ESLint tooling is updated: `ComboboxControl` is removed from the required-props set in `components-no-missing-40px-size-prop.js` and added to the restricted syntax list in `tools/eslint/config.mjs`, with corresponding test and suppression updates.

## Contribution

Opened by @mirka with co-authorship from @ciampo. The change finalizes a broader deprecation sweep that previously made larger sizing the default for `BorderBoxControl`, `BorderControl`, `FontSizePicker`, and `TreeSelect`. Review focused on verifying ESLint rule alignment, confirming all internal editor surfaces render correctly without the opt-in flag, and ensuring test suites pass post-cleanup.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
