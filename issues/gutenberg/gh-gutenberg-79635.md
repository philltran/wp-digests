# #79635: FontAppearanceControl: Hard deprecate 40px default size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Breaking Change`, `[Package] Block editor`
- **Merged:** [`6ea3deb`](https://github.com/WordPress/gutenberg/commit/6ea3deb210f5ab388f5433371264ab0f087ff5fc)
- **Discussion:** [#79635](https://github.com/WordPress/gutenberg/pull/79635) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Hard deprecates the `__next40pxDefaultSize` prop on `FontAppearanceControl`, making the larger 40px default height permanent and removing the soft-deprecation console warning introduced in WordPress 6.8. The prop is now accepted but ignored, allowing developers to safely remove it from all usages. This change finalizes a multi-version transition to standardize the Block Editor's typography control sizing without altering functional behavior.

## Impact

- **Block editor / component developers:** If you explicitly pass `__next40pxDefaultSize` to `FontAppearanceControl` (e.g., in custom blocks, custom panels, or theme templates), you can remove it. The 40px height is now the unconditional default.
- **Theme/plugin authors using typography wrappers:** No action required; internal wrappers have already dropped the redundant `size="__unstable-large"` attribute.
- **ESLint/plugin config maintainers:** The `components-no-missing-40px-size-prop` rule no longer enforces this prop on `FontAppearanceControl`. Consumers relying on that lint rule will see it removed from the required component list.

## Technical details

In `packages/block-editor/src/components/font-appearance-control/index.js`, the component destructures `__next40pxDefaultSize` as an ignored variable (`_next40pxDefaultSize`) and unconditionally passes `__next40pxDefaultSize` to the underlying `CustomSelectControl`. The conditional `deprecated()` call (which logged a warning when the prop was absent) is removed along with its import. A `@deprecated` JSDoc tag marks the default behavior as since WordPress 7.1. In `packages/block-editor/src/components/global-styles/typography-panel.js`, `size="__unstable-large"` is stripped from the component instance. The ESLint plugin updates remove `FontAppearanceControl` from the `COMPONENTS_REQUIRING_40PX` set in `components-no-missing-40px-size-prop.js`, and add it to a `restrictedSyntax` forbid list in `tools/eslint/config.mjs`. The `@wordpress/block-editor` CHANGELOG is updated with a breaking changes note.

## Contribution

Opened and merged by @mirka as a direct follow-up to #65751. The PR implements the hard deprecation phase of a previously soft-deprecated prop, aligning internal typography wrappers ahead of their underlying primitives. The discussion was brief, focusing on finalizing the console warning removal and ESLint rule alignment before the 7.1 release window.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
