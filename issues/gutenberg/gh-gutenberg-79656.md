# #79656: ToggleGroupControl: Hard deprecate 40px default size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Breaking Change`, `[Package] Components`, `[Package] Block library`, `[Package] Block editor`, `[Package] DataViews`
- **Merged:** [`619d499`](https://github.com/WordPress/gutenberg/commit/619d499eb7b14bf2dd032f6b4810ebe9b037a5e8)
- **Discussion:** [#79656](https://github.com/WordPress/gutenberg/pull/79656) · 2 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Hard deprecates the `__next40pxDefaultSize` and `size` props on the `ToggleGroupControl` component, making the 40px default height unconditional. The opt-in sizing prop was retained only to satisfy existing type definitions while being ignored at runtime, effectively removing an unstable sizing API from the public contract.

## Impact

- **Block & Plugin Developers**: Any call to `<ToggleGroupControl>` passing `__next40pxDefaultSize` or a `size` prop will now ignore that value and always render at 40px. Remove these props to clear deprecation warnings and prepare for eventual full removal.
- **Internal Gutenberg/Block Editor Users**: No immediate code changes required; internal call sites across `@wordpress/block-editor` and `@wordpress/block-library` have already been cleaned up in this PR.

## Technical details

The unified diff strips the `__next40pxDefaultSize` prop from ~25 `<ToggleGroupControl>` instances and removes `size="__unstable-large"` where explicitly passed. In `packages/components/src/toggle-group-control`, the component now destructures these props without usage, enforcing the 40px default layout unconditionally. The ESLint rule `components-no-missing-40px-size-prop` is updated to forbid `ToggleGroupControl` entirely since the prop is no longer functional or required.

Before/After migration pattern:
```diff
-<ToggleGroupControl
-  __next40pxDefaultSize
-  onChange={ handleToggle }
->
+<ToggleGroupControl
+  onChange={ handleToggle }
+>
```
The component's TypeScript types mark the props as deprecated but remain present to maintain backward compatibility during the hard-deprecation phase.

## Contribution

Opened and merged by @mirka with co-authorship from @ciampo. Labeled as a `[Type] Breaking Change` within the Components package, the PR was a targeted cleanup following earlier sizing deprecations for related controls. The diff shows a straightforward propagation of the change across block-editor and block-library consumer files without substantive design debate noted in the upstream material.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
