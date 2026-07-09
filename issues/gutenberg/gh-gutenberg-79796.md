# #79796: CustomSelectControl: Hard deprecate 40px default size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Breaking Change`, `[Package] Components`, `[Package] Block editor`
- **Merged:** [`af401ae`](https://github.com/WordPress/gutenberg/commit/af401ae08ac7539bc44c807c9ddccf51a791e1a0)
- **Discussion:** [#79796](https://github.com/WordPress/gutenberg/pull/79796) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Hard deprecates the `__next40pxDefaultSize` prop on `CustomSelectControl`, making the 40px default height unconditional. The opt-in transition period has concluded, so the prop no longer affects rendering logic and can be safely removed from all call sites.

## Impact

- **Block & theme developers**: Remove `__next40pxDefaultSize` from custom `<CustomSelectControl>` instances; it now acts as a no-op.
- **Plugin/Component authors**: The internal size translation is simplified, meaning `size="default"` and `size="__unstable-large"` both unconditionally map to the 40px default height.
- **No action required** for standard Gutenberg blocks or end-users, though custom editor UIs should audit their `<CustomSelectControl>` usages.

## Technical details

In `packages/components/src/custom-select-control/index.tsx`, the component's prop destructure and the `maybeWarnDeprecated36pxSize` utility call were removed. The conditional size translator was replaced with a direct ternary:

```tsx
// Before
const translatedSize = ( () => {
  if ((__next40pxDefaultSize && size === 'default') || size === '__unstable-large') return 'default';
  if (!__next40pxDefaultSize && size === 'default') return 'compact';
  return size;
} )();

// After
size={ size === '__unstable-large' ? 'default' : size }
```

The diff strips `__next40pxDefaultSize` from internal call sites in `packages/block-editor/src/components/` (e.g., `font-appearance-control`, `position.js`) and `packages/components/src/`. The TypeScript definitions in `types.ts` mark the prop as `@deprecated Default behavior since WordPress 7.1` with `@ignore`. Corresponding ESLint rules (`components-no-missing-40px-size-prop`) and test suites were updated to remove checks for this prop.

## Contribution

Opened and merged by @mirka, with co-authorship noted by @juanfra. The PR acts as a follow-up to #65751 to formally close the deprecation lifecycle. Review focused on ensuring internal monorepo call sites were cleaned up and that ESLint rules no longer enforced or warned about the prop.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
