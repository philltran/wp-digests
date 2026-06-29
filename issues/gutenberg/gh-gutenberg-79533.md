# #79533: LetterSpacingControl: Hard deprecate 40px default size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Breaking Change`, `[Package] Block editor`
- **Merged:** [`b7477a7`](https://github.com/WordPress/gutenberg/commit/b7477a7389df88ea575b857578f834e65148944a)
- **Discussion:** [#79533](https://github.com/WordPress/gutenberg/pull/79533) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Hard deprecates the `__next40pxDefaultSize` prop on `LetterSpacingControl`, making the 40px default height permanent and removing the soft-deprecation warning introduced in WordPress 6.8. This change finalizes the sizing timeline for letter spacing controls across the block editor, ensuring all instances now render at the larger default height without runtime warnings.

## Impact

- Plugin & block developers using `@wordpress/block-editor`'s `LetterSpacingControl` should remove explicit `__next40pxDefaultSize` usage, as it is now unconditionally true and will be silently ignored if passed.
- Internal typography wrappers (e.g., `TypographyPanel`) are updated to stop passing the deprecated prop.
- No action required for site owners or REST/headless consumers.

## Technical details

In `packages/block-editor/src/components/letter-spacing-control/index.js`, the soft deprecation block triggering `@wordpress/deprecated` is removed, and the prop is unconditionally passed to the embedded `UnitControl`. The JSDoc retains a stub signature but ignores the argument. Internally, `packages/block-editor/src/components/global-styles/typography-panel.js` drops the obsolete prop from its instance. The ESLint plugin `components-no-missing-40px-size-prop` removes `LetterSpacingControl` from its required-prop validation set and adds it to a new `restricted-syntax` rule in `tools/eslint/config.mjs` to prevent passing the obsolete attribute.

## Contribution

Merged by @mirka (co-authored with @ciapo) following the deprecation timeline outlined in #65751. The PR executed the finalized hard-deprecation phase by stripping warning logic, updating internal callers, and locking down the prop via ESLint restrictions to prevent legacy usage patterns.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
