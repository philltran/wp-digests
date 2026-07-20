# #79420: BorderBoxControl: Hard deprecate 40px default size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] Block editor`
- **Merged:** [`3ae3274`](https://github.com/WordPress/gutenberg/commit/3ae3274a3cc7d1d908151ea525e2be195d213f33)
- **Discussion:** [#79420](https://github.com/WordPress/gutenberg/pull/79420) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `BorderBoxControl` component in `@wordpress/components` now permanently defaults to a 40px height, hard-deprecating the `__next40pxDefaultSize` and `size` props. This completes a migration path that began in WordPress 6.7, ensuring the larger control size is applied unconditionally across the block editor.

## Impact

- **Plugin & theme developers**: The `__next40pxDefaultSize` and `size` props on `BorderBoxControl` are now ignored. You can safely remove them from your component usage.
- **No action required** for developers who already passed the prop or relied on the default behavior, as the 40px height is now the permanent default.
- **ESLint users**: The `components-no-missing-40px-size-prop` rule no longer checks `BorderBoxControl`.

## Technical details

The change strips `__next40pxDefaultSize` and `size` from the `BorderBoxControlProps` type definition in `packages/components/src/border-box-control/types.ts`, marking them as deprecated/ignored. In `packages/components/src/border-box-control/border-box-control/hook.ts`, these props are destructured as `_size` and `_next40pxDefaultSize`, the `maybeWarnDeprecated36pxSize` warning logic is removed, and the `computedSize` fallback is eliminated. The component now unconditionally passes `size="__unstable-large"` and `width="116px"` to its internal `BorderControl` and `BorderBoxControlSplitControls` instances. Corresponding CSS helper functions in `styles.ts` (`borderBoxControlLinkedButton`, `borderBoxControlVisualizer`, `borderBoxControlSplitControls`) have been simplified to remove the `size` parameter and hardcode the larger spacing values. The `components-no-missing-40px-size-prop` ESLint rule has been updated to exclude `BorderBoxControl` from its checks.

## Contribution

Opened and merged by @mirka (co-authored by @ciampo), this PR executes the scheduled hard deprecation timeline announced in WordPress 6.7. It removes the legacy prop handling and internal warning logic now that WordPress 7.1 has shipped, following the initial soft deprecation in #65751.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
