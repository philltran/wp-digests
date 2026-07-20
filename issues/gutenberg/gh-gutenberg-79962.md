# #79962: InputControl: Hard deprecate 40px default size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] Editor`, `[Package] Block library`, `[Package] Format library`, `[Package] Block editor`, `[Package] DataViews`, `[Package] Fields`
- **Merged:** [`3a65a6a`](https://github.com/WordPress/gutenberg/commit/3a65a6a9a161339d92726cc7eeea30fb861ba08e)
- **Discussion:** [#79962](https://github.com/WordPress/gutenberg/pull/79962) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `InputControl` and `InputBase` components in `@wordpress/components` now default to a 40px height, removing the previous 36px fallback. The `__next40pxDefaultSize` prop has been hard-deprecated and is now a no-op stub, as the opt-in period for the larger size has ended. This ensures consistent control sizing across the block editor and component library.

## Impact

- **Plugin & theme developers:** If you render `InputControl` or `InputBase` directly, the default height will increase from 36px to 40px. Remove any `__next40pxDefaultSize` props you pass, as they are now ignored.
- **Block editor developers:** Internal editor components that previously passed `__next40pxDefaultSize` or `size="__unstable-large"` have been updated. No migration needed for standard block development.
- **No action required** if you rely on default component styling and don't pass the deprecated prop.

## Technical details

The change lives in `packages/components/src/input-control/`. `getSizeConfig` in `input-control-styles.tsx` no longer accepts `__next40pxDefaultSize` and unconditionally returns the 40px `default` size instead of falling back to `compact`. `InputControl` and `InputBase` now destructure and ignore `__next40pxDefaultSize`, `__next36pxDefaultSize`, and `__shouldNotWarnDeprecated36pxSize` instead of passing them through or triggering `maybeWarnDeprecated36pxSize`. The props are moved from `BaseProps` to `InputControlProps` with `@deprecated` JSDoc stubs. Internal wrappers (`NumberControl`, `SearchControl`, `SelectControl`, `CustomSelectControl` v2, `ValidatedInputControl`) and editor call sites (`child-layout-control`, `media-placeholder`, `embed-placeholder`, `rss/edit`, `hex-input`) have all had explicit `__next40pxDefaultSize` or `size="__unstable-large"` attributes removed.

## Contribution

Opened and merged by @mirka (with @ciampo as co-author) as a follow-up to #65751. The PR systematically cleaned up the deprecation lifecycle by stripping warning utilities, updating TypeScript types, and removing the prop from all internal Gutenberg call sites and stories.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
