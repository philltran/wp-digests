# #79720: FormTokenField: Hard deprecate 40px default size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Breaking Change`, `[Package] Components`, `[Package] Editor`, `[Package] Block library`, `[Package] Patterns`
- **Merged:** [`7c8d0e0`](https://github.com/WordPress/gutenberg/commit/7c8d0e0dc3dbe872694bfa4185c5eb92fd83084e)
- **Discussion:** [#79720](https://github.com/WordPress/gutenberg/pull/79720) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `__next40pxDefaultSize` prop on the `FormTokenField` component has been hard-deprecated and removed from the runtime API, finalizing a deprecation cycle that began in WordPress 6.7. The component now permanently renders at the 40px height, eliminating the legacy size toggle and associated warning logic.

## Impact

- **Plugin & theme developers**: Must strip `<FormTokenField __next40pxDefaultSize />` from custom implementations; the prop is no longer accepted at runtime and will trigger ESLint's new restricted-syntax rule.
- **No visual migration required**: Since the 40px rendering is now baked into the default styles, standard integrations continue functioning without layout shifts.
- **Type definitions updated**: The TypeScript interface marks the prop as `@deprecated` with an `@ignore` tag to prevent IDE autocompletion while preserving backward compatibility for static analysis.

## Technical details

- In `packages/components/src/form-token-field/index.tsx`, destructuring of `__next40pxDefaultSize` was removed alongside imports for `useDeprecated36pxDefaultSizeProp` and `maybeWarnDeprecated36pxSize`. The conditional size flag passed to `TokensAndInputWrapperFlex` was eliminated.
- `packages/components/src/form-token-field/styles.ts` drops the `css` import, the `TokensAndInputWrapperProps` interface, and the `deprecatedPaddings` function, locking the wrapper to a static `padding: 7px;`.
- `packages/components/src/form-token-field/types.ts` updates the prop's JSDoc to `@deprecated Default behavior since WordPress 7.1. Prop can be safely removed.` with `@ignore`.
- The ESLint plugin drops `FormTokenField` from the `components-no-missing-40px-size-prop` rule and adds it to a new `restricted-syntax` configuration in `tools/eslint/config.mjs` to prevent accidental propagation.
- Internal usages were stripped across `block-library/src/query/edit/inspector-controls/*.js`, `patterns/src/components/category-selector.js`, `editor/src/components/post-taxonomies/flat-term-selector.js`, and the wrapped `ValidatedFormTokenField` implementation.

## Contribution

Merged by @mirka with co-author @ciampo as a direct follow-up to #65751. The PR executes the scheduled WordPress 7.1 removal window by cleaning up all internal monorepo usages across `block-library`, `patterns`, `editor`, and component utilities, while enforcing the API change through updated TypeScript stubs and ESLint configurations.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
