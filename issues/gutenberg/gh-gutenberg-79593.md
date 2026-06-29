# #79593: FontFamilyControl: Hard deprecate 40px default size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Breaking Change`, `[Package] Block editor`
- **Merged:** [`15e5d7d`](https://github.com/WordPress/gutenberg/commit/15e5d7d3c0918f8e0fc291e72e9e6e606a4ed9e9)
- **Discussion:** [#79593](https://github.com/WordPress/gutenberg/pull/79593) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Hard-deprecates the `__next40pxDefaultSize` prop on the `FontFamilyControl` component in `@wordpress/block-editor`, making the 40px default height permanent and removing the soft console warning introduced in WordPress 6.8. This cleanup ensures consistent typography sizing across the Block Editor while retiring legacy opt-in props.

## Impact

- Theme and plugin developers using `FontFamilyControl` from `@wordpress/block-editor` should remove explicit `__next40pxDefaultSize` props, as the prop is now accepted but ignored.
- No runtime behavior change for end users; the control will continue to render at 40px height by default.
- Console warnings previously logged when the prop was omitted are removed.

## Technical details

In `packages/block-editor/src/components/font-family/index.js`, the soft deprecation logic and `deprecated()` call were removed. The component now destructures the prop as `_next40pxDefaultSize` (prefixed to signal it is ignored) and unconditionally passes `__next40pxDefaultSize={true}` to the inner `<CustomSelectControl>`. The component was removed from the `components-no-missing-40px-size-prop` ESLint rule checks in `packages/eslint-plugin/rules/components-no-missing-40px-size-prop.js` and its test suite, while simultaneously being added to the `restrictedSyntax` forbid list in `tools/eslint/config.mjs`. Storybook defaults in `stories/index.story.jsx` and the component README were stripped of the prop. Documented as a breaking change in `@wordpress/block-editor/CHANGELOG.md`.

## Contribution

Opened and merged by @mirka (co-authored with @ciampo) as a follow-up to #65751. Part of a coordinated update series applying the 40px default size to Block Editor typography controls before their underlying primitives. Merged into the Gutenberg repository.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
