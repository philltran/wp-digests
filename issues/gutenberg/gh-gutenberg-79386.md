# #79386: TextControl: Hard deprecate 40px default size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Core data`, `[Package] Components`, `[Package] Editor`, `[Package] Block library`, `[Package] Format library`, `[Package] Block editor`, `[Package] Edit Post`, `[Package] Edit Site`, `[Package] Patterns`, `[Package] Fields`
- **Merged:** [`a616a85`](https://github.com/WordPress/gutenberg/commit/a616a85571708dca6a088a909dfaa04bd10e0d71)
- **Discussion:** [#79386](https://github.com/WordPress/gutenberg/pull/79386) · 2 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `TextControl` component permanently adopts the 40px default height, hard-deprecating the `__next40pxDefaultSize` prop. This completes the first phase of a per-component rollout to standardize input sizing across the block editor, following a soft deprecation introduced in WordPress 6.7. The change ensures consistent UI spacing without requiring explicit prop overrides.

## Impact

- **Plugin & theme developers**: Explicitly passing `__next40pxDefaultSize` to `TextControl` is now deprecated. The prop will be ignored, and the component will render at 40px by default. Remove the prop from your code to silence deprecation warnings.
- **Block authors**: Existing blocks using the prop will continue to render identically. No visual migration is required, but cleanup is recommended.
- **No action required** for site owners or end users.

## Technical details

The diff strips the `__next40pxDefaultSize` prop from `TextControl` across `packages/block-editor/`, `packages/block-library/`, and documentation files. The component's runtime handling of the prop is removed, and the 40px styling is merged into the default CSS. A `@deprecated` `@ignore` TypeScript stub remains for external callers. The ESLint rule `components-no-missing-40px-size-prop` is removed, and a `restricted-syntax` rule is added to block passing the prop to `TextControl`.

```jsx
// Before
<TextControl __next40pxDefaultSize label="Title" />
// After
<TextControl label="Title" />
```

## Contribution

Opened and merged by @mirka with co-authorship from @ciampo, this PR executes the hard-deprecation phase of a scheduled rollout initiated in #65751. Review was minimal, with no alternative approaches debated or rejected; the work focused on systematic cleanup of core usages and documentation examples.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
