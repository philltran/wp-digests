# #79861: NumberControl: Hard deprecate 40px default size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Breaking Change`, `[Package] Components`, `[Package] Editor`, `[Package] Block library`, `[Package] Format library`, `[Package] Block editor`, `[Package] DataViews`
- **Merged:** [`b8228b2`](https://github.com/WordPress/gutenberg/commit/b8228b2b2493839930c08e210b3befe38bf020b7)
- **Discussion:** [#79861](https://github.com/WordPress/gutenberg/pull/79861) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `NumberControl` component in `@wordpress/components` has hard-deprecated the `__next40pxDefaultSize` prop, permanently locking the control to a 40px height. This follow-up to an earlier opt-in standardization removes deprecation warnings and internal size-mismatch logic, ensuring the larger default is enforced across all call sites without console notices. Plugin and block developers will see no visual changes but should update type definitions or legacy prop passes accordingly.

## Impact

- **Block & Plugin Developers:** No action required for rendering or functionality. The `__next40pxDefaultSize` prop is now officially deprecated (marked `@deprecated` in types) and safely ignored without triggering warnings.
- **Component Maintainers:** The `size="__unstable-large"` prop is redundant and has been stripped from internal implementations. The `components-no-missing-40px-size-prop` ESLint rule no longer enforces this check for `NumberControl`.
- **Editor UI Teams:** Internal controls (Range, Unit, Validated, TimePicker) automatically inherit the 40px height, eliminating scattered sizing flags across the Gutenberg monorepo.

## Technical details

In `packages/components/src/number-control/index.tsx`, the deprecation warning pipeline (`useDeprecated36pxDefaultSizeProp` and `maybeWarnDeprecated36pxSize`) is removed. The component now explicitly destructures `__next40pxDefaultSize` from props to prevent it from leaking downstream, while hardcoding `__next40pxDefaultSize` on the internal `<Input>` element. `packages/components/src/number-control/types.ts` adds a `@deprecated` JSDoc tag stating "Default behavior since WordPress 7.1". Call sites across `RangeControl`, `UnitControl`, `ValidatedNumberControl`, `TimePicker`, and layout components have had their explicit `size="__unstable-large"` props removed. The ESLint rule `components-no-missing-40px-size-prop` excludes `NumberControl` from its enforcement set, shifting it to a repo-wide forbid list for the prop.

*Before*:
```tsx
<NumberControl __next40pxDefaultSize size="__unstable-large" />
```
*After*:
```tsx
<NumberControl /> // Always renders at 40px; __next40pxDefaultSize is safely dropped
```

## Contribution

Merged by @mirka with co-author @ciampo, this PR closes the opt-in lifecycle for `NumberControl` sizing initiated in #65751. The diff reflects a targeted cleanup to harden the deprecation cycle, strip remaining warning logic, and standardize internal call sites across the components and block-editor packages. No alternative approaches were debated; the change proceeds directly as a final enforcement step.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
