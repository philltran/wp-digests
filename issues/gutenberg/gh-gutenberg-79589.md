# #79589: LineHeightControl: Hard deprecate 40px default size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Breaking Change`, `[Package] Block editor`
- **Merged:** [`074a32f`](https://github.com/WordPress/gutenberg/commit/074a32f684d67b0637444d614e29b4aa917c99ca)
- **Discussion:** [#79589](https://github.com/WordPress/gutenberg/pull/79589) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Hard deprecates the `__next40pxDefaultSize` prop on `LineHeightControl`, making the 40px line-height default permanent and removing console warnings for omitting it. This follows up on a soft deprecation introduced in WordPress 6.8, finalizing the typography control sizing update across the Block Editor.

## Impact

- **Block editor & plugin/theme developers**: Remove `__next40pxDefaultSize={true}` from any `<LineHeightControl />` instances to eliminate console warnings and align with the new permanent default.
- **Headless & REST consumers**: No impact; line-height rendering remains identical.
- **Hosting & platform teams**: No action required.

## Technical details

In `packages/block-editor/src/components/line-height-control/index.js`, the conditional `deprecated()` call and size-checking logic were removed. The component now destructures `__next40pxDefaultSize` as `_next40pxDefaultSize` (ignoring it) and statically passes `__next40pxDefaultSize` to the inner `NumberControl`. The prop's documentation was removed from `README.md`, and Storybook/unit test defaults were updated to omit it entirely. Internally, `LineHeightControl` was removed from the `components-no-missing-40px-size-prop` ESLint rule and added to the `restricted-syntax` forbid list in `tools/eslint/config.mjs`.

## Contribution

Opened by @mirka as a follow-up to #65751, this PR finalized the transition from WordPress 6.8's soft deprecation. Merged with co-authorship from @ciampo, it executed the hard-deprecation cleanup, removed remaining internal references to the opt-in prop, and aligned ESLint configurations with the permanent default.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
