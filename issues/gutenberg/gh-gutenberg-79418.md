# #79418: BorderControl: Hard deprecate 40px default size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Components`
- **Merged:** [`887080b`](https://github.com/WordPress/gutenberg/commit/887080b1885471798fef74ca2e2bbb6ec768691b)
- **Discussion:** [#79418](https://github.com/WordPress/gutenberg/pull/79418) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `BorderControl` component in `@wordpress/components` has hard-deprecated the `__next40pxDefaultSize` prop, making the 40px default height permanent. This completes a soft deprecation cycle that began in WordPress 6.7 and was scheduled for removal in WordPress 7.1. The `size` prop is also removed from runtime logic, with `size="default"` now permanently mapping to the larger `__unstable-large` dimensions.

## Impact

- **Block & plugin developers**: If your custom editor code passes `__next40pxDefaultSize` or `size` to `<BorderControl>`, these props are now ignored at runtime. You can safely remove them from your JSX.
- **No action required** if you are already relying on the default 40px height or using `BorderControl` without these props.
- **ESLint users**: The `components-no-missing-40px-size-prop` rule no longer checks `BorderControl`. A new `restricted-syntax` rule will now flag attempts to pass `__next40pxDefaultSize` to `BorderControl`.

## Technical details

- `packages/components/src/border-control/border-control/component.tsx` and `hook.ts` remove `__next40pxDefaultSize` and `size` from destructuring and stop calling `maybeWarnDeprecated36pxSize`.
- `packages/components/src/border-control/styles.ts` replaces the conditional `wrapperHeight` and `colorIndicatorWrapper` functions with hardcoded CSS values (`height: 40px`, indicator `width/height: 24px`, `padding: 2px`).
- Compact mode in `hook.ts` now unconditionally sets `wrapperWidth = '116px'`.
- `packages/components/src/border-control/types.ts` retains `size` and `__next40pxDefaultSize` in the `BorderControlProps` interface but marks them with `@deprecated` and `@ignore` JSDoc tags.
- `packages/eslint-plugin/rules/components-no-missing-40px-size-prop.js` removes `BorderControl` from the checked components set, while `tools/eslint/config.mjs` adds `BorderControl` to the `restrictedSyntax` array to prevent passing `__next40pxDefaultSize`.
- Internal calls in `border-box-control-split-controls/component.tsx` and `border-box-control/component.tsx` drop the explicit `size="__unstable-large"` attribute, relying on the new default.

## Contribution

Opened and merged by @mirka (co-authored with @ciampo) as a follow-up to #65751 and #79420 (`BorderBoxControl`). The change follows the planned deprecation timeline initiated in WordPress 6.7, finalizing the size standardization for the components package ahead of the WordPress 7.1 release window.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
