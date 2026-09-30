# #83273: Components: Deprecate Scrollable

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Tool] ESLint plugin`
- **Merged:** [`8cde60a`](https://github.com/WordPress/gutenberg/commit/8cde60aeae4cd77c063513bc42874af942920989)
- **Discussion:** [#83273](https://github.com/WordPress/gutenberg/pull/83273) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `__experimentalScrollable` component in `@wordpress/components` is now formally deprecated with a runtime warning, scheduled for removal in WordPress 7.4. It was already marked not-recommended in Storybook and the `use-recommended-components` ESLint rule; this PR makes the deprecation concrete by routing the public barrel export through a new adapter that calls `@wordpress/deprecated`. No replacement component exists in `@wordpress/ui` — the guidance is to write custom CSS. `CardBody`'s internal use of Scrollable is unaffected and does not trigger the warning.

## Impact

- **Plugin & theme developers using `__experimentalScrollable`**: A console warning (`wp.components.__experimentalScrollable is deprecated since version 7.2 and will be removed in version 7.4.`) will now fire on every render. Replace with a plain `<div>` and CSS `overflow` / `max-height` rules before the 7.4 removal.
- **Developers using `CardBody` with `isScrollable`**: No action required. The internal `Scrollable` export used by `CardBody` is unchanged and does not log the deprecation.
- **ESLint users**: The `use-recommended-components` rule message for `__experimentalScrollable` changed from "planned for deprecation" to "Write your own CSS instead."
- **No breaking change yet.** The component still renders identically; removal is targeted for WordPress 7.4.

## Technical details

A new file `packages/components/src/scrollable/deprecated.tsx` defines `UnconnectedDeprecatedScrollable`, which calls `deprecated('wp.components.__experimentalScrollable', { since: '7.2', version: '7.4' })` from `@wordpress/deprecated` and then delegates to `UnconnectedScrollable`. It is wrapped with `contextConnect` and exported as the default.

The barrel export in `packages/components/src/index.ts` changed:

```ts
// before
export { Scrollable as __experimentalScrollable } from './scrollable';

// after
export { default as __experimentalScrollable } from './scrollable/deprecated';
```

`UnconnectedScrollable` in `packages/components/src/scrollable/component.tsx` was previously a module-private function; it is now `export`ed so the deprecated adapter can call it directly (avoiding a double `contextConnect`). The original connected `Scrollable` in `packages/components/src/scrollable/index.ts` remains untouched for internal use by `CardBody`.

In `packages/eslint-plugin/rules/use-recommended-components.js`, the `DENYLIST` entry for `__experimentalScrollable` was updated from `'{{ name }} is planned for deprecation.'` to `'Write your own CSS instead.'`

The Storybook story was moved from `Components/@wordpress-components/Scrollable` to `Components/@wordpress-components/Deprecated/Scrollable` and its `componentStatus.notes` changed to "Deprecated. Write your own CSS instead."

## Contribution

Opened by @mirka with co-authorship from @simison. It is part of a coordinated batch of component deprecations in the same release cycle (Elevation #83269, Divider #83040 appear alongside it in the changelog). The two comments on the PR are automated bot posts (bundle-size and performance reports); no design debate or alternative approaches are visible in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
