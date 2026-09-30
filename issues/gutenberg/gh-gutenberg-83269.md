# #83269: Components: Deprecate Elevation

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Components`
- **Merged:** [`8276851`](https://github.com/WordPress/gutenberg/commit/827685105be82cb40ec3359aa9d0598b0654e6bf)
- **Discussion:** [#83269](https://github.com/WordPress/gutenberg/pull/83269) · 2 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `__experimentalElevation` component in `@wordpress/components` is now formally deprecated with a runtime console warning. It was already flagged as "not-recommended" in Storybook and the `use-recommended-components` lint rule, but this PR makes the deprecation visible at runtime via `@wordpress/deprecated`. The recommended replacement is the elevation tokens from `@wordpress/base-styles`. The component still renders normally; removal is planned for WordPress 7.4.

## Impact

- **Plugin & theme developers:** If you import `__experimentalElevation` from `@wordpress/components`, you will now see a console warning: `wp.components.__experimentalElevation is deprecated since version 7.2 and will be removed in version 7.4.` Migrate to elevation tokens from `@wordpress/base-styles` before the 7.4 removal.
- **`Card` users:** No action required. `Card` uses an internal `Elevation` export that does not trigger the deprecation warning.
- **No breaking change yet.** The component still renders its shadow; the warning is informational. No configuration or migration is required immediately, but the 7.4 removal timeline means this should be on your upgrade checklist.

## Technical details

A new file `packages/components/src/elevation/deprecated.tsx` introduces `UnconnectedDeprecatedElevation`, which calls `deprecated('wp.components.__experimentalElevation', { since: '7.2', version: '7.4' })` and then delegates to the existing `UnconnectedElevation` render. It is wrapped with `contextConnect(UnconnectedDeprecatedElevation, 'Elevation')` and exported as the default.

`UnconnectedElevation` in `packages/components/src/elevation/component.tsx` is now exported (previously module-private) so the deprecated adapter can reuse it without double-connecting context.

The barrel export in `packages/components/src/index.ts` changes:

```ts
// before
export { Elevation as __experimentalElevation } from './elevation';

// after
export { default as __experimentalElevation } from './elevation/deprecated';
```

The internal connected `Elevation` in `packages/components/src/elevation/index.ts` is untouched and remains what `Card` imports internally, so `Card` does not log the warning.

The Storybook story moves from `Components/Elevation` to `Components/Deprecated/Elevation` (keeping `id: 'components-elevation'` for URL stability), and the README callout is updated to point to the elevation tokens documentation in `@wordpress/base-styles`.

## Contribution

Opened by @mirka with co-authorship from @simison. The PR carries only 2 comments and 0 reactions, with no visible design debate or alternative approaches discussed in the provided record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
