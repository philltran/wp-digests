# #79550: TreeSelect: Hard deprecate 40px default size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Breaking Change`, `[Package] Components`, `[Package] Editor`
- **Merged:** [`2715f8d`](https://github.com/WordPress/gutenberg/commit/2715f8d770de066666b65ac37f5a436cd57be548)
- **Discussion:** [#79550](https://github.com/WordPress/gutenberg/pull/79550) · 2 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `TreeSelect` component hard-deprecates the `__next40pxDefaultSize` prop, making the larger default height permanent and removing the opt-in runtime API. This cleanup finalizes the deprecation cycle that began in WordPress 6.7 and aligns with the scheduled removal before WordPress 7.1.

## Impact

- **Block & plugin developers**: If you explicitly pass `__next40pxDefaultSize` to `<TreeSelect>`, the prop will be silently ignored and stripped at runtime. Remove it from your implementations to prevent lint warnings and ensure clean builds.
- **Hosting & platform teams**: No action required; default rendering behavior remains unchanged after this cleanup.
- **Internal/editor consumers**: All core usages have been updated to rely on the permanent default without explicit prop passing.

## Technical details

In `packages/components/src/tree-select/index.tsx`, the component now directly destructures props (`} = props`) instead of wrapping them with `useDeprecated36pxDefaultSizeProp`. The runtime warning logic from `maybeWarnDeprecated36pxSize` is removed. The TypeScript interface in `types.ts` marks `__next40pxDefaultSize` as `@deprecated` and removes it from the `Omit` list, though it remains passed internally to `SelectControl` until that primitive is fully deprecated. Internal callers (`query-controls/author-select.tsx`, `query-controls/category-select.tsx`, and `post-taxonomies/hierarchical-term-selector.js`) have all had the prop stripped. Additionally, `TreeSelect` was removed from the `components-no-missing-40px-size-prop` ESLint allowlist in `packages/eslint-plugin/rules/components-no-missing-40px-size-prop.js` and added to a `restrictedSyntax` rule in `tools/eslint/config.mjs` to block monorepo regressions.

## Contribution

Pull request #79550 was authored by @mirka and merged following the planned soft deprecation cycle from WordPress 6.7. The change streamlined the component lifecycle ahead of the WordPress 7.1 release, with review contributions from @ciampo focusing on cleanup verification and ESLint configuration alignment.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
