# #79481: FontSizePicker: Hard deprecate 40px default size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] Block editor`
- **Merged:** [`8193614`](https://github.com/WordPress/gutenberg/commit/8193614095316086a3581b989bbcb9ca12f1c585)
- **Discussion:** [#79481](https://github.com/WordPress/gutenberg/pull/79481) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `FontSizePicker` component permanently adopts the 40px default control height by hard-deprecating the `__next40pxDefaultSize` and `size` props. This completes a soft deprecation introduced in WordPress 6.7 and executes the scheduled removal for WordPress 7.1.

## Impact

- Plugin & theme developers: If you explicitly passed `__next40pxDefaultSize` or `size` to `FontSizePicker`, you can safely remove them. The component now always renders at the 40px height.
- Block editor developers: Internal block editor wrappers (`packages/block-editor/src/components/font-sizes/font-size-picker.js`, `typography-panel.js`, and `hooks/font-size.js`) have been updated to drop the deprecated `size="__unstable-large"` prop.
- No action required for developers who did not explicitly pass these props or who already migrated to the new default.

## Technical details

The diff removes `__next40pxDefaultSize` and `size` from the `FontSizePicker` public API and TypeScript definitions (`packages/components/src/font-size-picker/types.ts`), marking them `@deprecated` and `@ignore`. The main component (`packages/components/src/font-size-picker/index.tsx`) stops destructuring these props and removes the `maybeWarnDeprecated36pxSize` warning logic. Internal sub-components (`font-size-picker-select.tsx`, `font-size-picker-toggle-group.tsx`) now hardcode `__next40pxDefaultSize` when passing it to underlying primitives (`UnitControl`, `RangeControl`, `CustomSelectControl`, `ToggleGroupControl`) until those primitives are also hard-deprecated. The block-editor wrapper and typography hooks drop `size="__unstable-large"`. Storybook stories and Jest tests are updated to reflect the new default behavior.

## Contribution

Opened and merged by @mirka (co-authored with @ciampo) as a follow-up to #65751. The change executes the scheduled hard deprecation timeline established during the WordPress 6.7 soft deprecation, with no alternative approaches debated in the review.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
