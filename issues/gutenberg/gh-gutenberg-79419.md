# #79419: BoxControl: Hard deprecate 40px default size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] Block editor`
- **Merged:** [`122a29e`](https://github.com/WordPress/gutenberg/commit/122a29e0638cc49033d9a5e8c05cf5cc4075b382)
- **Discussion:** [#79419](https://github.com/WordPress/gutenberg/pull/79419) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `BoxControl` component permanently adopts the 40px default input height, hard-deprecating the `__next40pxDefaultSize` prop. This change finalizes a soft deprecation introduced in WordPress 6.7 and scheduled for removal in WordPress 7.1. Developers no longer need to pass the prop, and the internal deprecation warning has been removed.

## Impact

- **Block editor & plugin developers:** The `__next40pxDefaultSize` prop is removed from `BoxControl`'s public API. If you explicitly passed it, you can safely remove it. If you relied on the legacy 36px default height, you will need to adjust your component styles or input values.
- **ESLint users:** The `components-no-missing-40px-size-prop` rule no longer checks `BoxControl`. A new `restricted-syntax` rule now blocks passing `__next40pxDefaultSize` to `BoxControl` (and `TextControl`), which may trigger linter errors in codebases that still reference it.
- **No action required** for standard block markup or themes not directly instantiating `BoxControl` via JavaScript.

## Technical details

The diff removes `__next40pxDefaultSize` from `BoxControlProps` in `packages/components/src/box-control/types.ts` and strips it from the component signature in `index.tsx`. The internal `maybeWarnDeprecated36pxSize` call is deleted. Instead of conditionally passing the prop, `BoxControl` now hardcodes `__next40pxDefaultSize` to `true` when rendering the internal `UnitControl` and `RangeControl` primitives. The internal input component is renamed from `InputControl` to `BoxInputControl` (`box-input-control.tsx`). In `packages/eslint-plugin/rules/components-no-missing-40px-size-prop.js`, `BoxControl` is removed from the `COMPONENTS_REQUIRING_40PX` set, and `tools/eslint/config.mjs` adds a `restricted-syntax` rule to prevent passing the prop to both `BoxControl` and `TextControl`.

**Before:**
```jsx
<BoxControl __next40pxDefaultSize values={ values } onChange={ setValues } />
```

**After:**
```jsx
<BoxControl values={ values } onChange={ setValues } />
```

## Contribution

Opened and merged by @mirka (co-authored with @ciampo), this PR serves as the scheduled follow-up to #65751. The change followed a straightforward review cycle with no major design debates or rejected alternatives documented in the PR thread. It was merged as part of the standard Gutenberg release cadence to align with the WordPress 7.1 deprecation timeline.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
