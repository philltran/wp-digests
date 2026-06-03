# #78528: Components: Remove deprecated `__experimentalApplyValueToSides` export

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Breaking Change`, `[Type] Code Quality`, `[Package] Components`, `Needs Dev Note`
- **Merged:** [`bffbc43`](https://github.com/WordPress/gutenberg/commit/bffbc43331780a87a64af866278de3018a99e040)
- **Discussion:** [#78528](https://github.com/WordPress/gutenberg/pull/78528) · 2 comments · 0 reactions

## Summary

The `@wordpress/components` package has removed the deprecated `__experimentalApplyValueToSides` export and its underlying implementation. This cleanup completes the deprecation cycle that began in WordPress 6.8, eliminating an unused utility to reduce the public API surface ahead of the 7.0 release.

## Impact

- **Plugin & Theme Developers:** This is a breaking change. Importing `__experimentalApplyValueToSides` from `@wordpress/components` will no longer resolve.
- **Action Required:** Remove any direct references to this utility. Since it was marked experimental and deprecated, no official drop-in replacement is provided; you must implement custom spacing/value transformation logic if previously relied upon for block controls or CSS spacing calculations.
- **Core/Block Editor & Platform Teams:** No action required. The function was not used within Gutenberg's internal components or the WordPress.org plugin directory.

## Technical details

- The change removes the explicit export from `packages/components/src/box-control/index.tsx`:
  ```diff
  -export { applyValueToSides } from './utils';
  export default BoxControl;
  ```
- It also strips the internal `applyValueToSides` implementation from `packages/components/src/box-control/utils.ts`, which was responsible for transforming spacing values for linked/unlinked box model sides.
- The removal directly reduces the minified build size by ~124 B in `build/scripts/components/index.min.js`.
- Any third-party code or block scripts that imported this symbol will encounter a runtime error or `undefined` export, requiring migration to standard CSS custom properties or internal `BoxControl` spacing APIs.

## Contribution

Open and merged by @mirka (commit [`bffbc43`](https://github.com/WordPress/gutenberg/commit/bffbc43331780a87a64af866278de3018a99e040)) with co-authorship from @ciampo. The PR followed a standard 6.8-to-7.0 deprecation timeline. During the review process, the author considered leaving a stubbed no-op export for compatibility but decided on full deletion after confirming zero usage across the official WordPress.org plugin repository and Gutenberg core.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
