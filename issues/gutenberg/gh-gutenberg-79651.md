# #79651: Base Styles: Make Sass token fallbacks self-contained

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Base styles`, `[Package] Theme`
- **Merged:** [`0fc4838`](https://github.com/WordPress/gutenberg/commit/0fc4838e1a403b102ee8594e11681c1da2c9f1f6)
- **Discussion:** [#79651](https://github.com/WordPress/gutenberg/pull/79651) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Vendors the design token fallback generator into `@wordpress/base-styles` to make it self-contained, eliminating a problematic transitive dependency on `@wordpress/theme`. This prevents Sass package resolution failures in isolated or hoisted dependency layouts, while simultaneously removing the newly introduced `@wordpress/theme/utils` Sass helper export.

## Impact

- **Theme/plugin developers using `@wordpress/theme` Sass tooling:** The `@wordpress/theme/utils` subpath export is removed. If you explicitly imported it via `@use '@wordpress/theme/utils' as wpds` or the `pkg:` scheme, you must switch to the local base-styles mixin helper.
- **Platform & monorepo maintainers:** Fixes a build-breaking issue where Sass could not resolve the transitive `@wordpress/theme` dependency from within `@wordpress/base-styles`. No action required if you rely on standard Base Styles usage or existing fallback injection plugins.

## Technical details

The diff moves the generated fallback SCSS from `packages/theme/src/prebuilt/scss/design-token-fallbacks.scss` to `packages/base-styles/internal/_wpds-token-fallbacks.scss`. `packages/base-styles/_mixins.scss` updates its import from `@use "@wordpress/theme/utils" as wpds;` to `@use "./internal/wpds-token-fallbacks" as wpds;`. The `@wordpress/theme` dependency is stripped from `packages/base-styles/package.json`, and the `"./utils": { "sass": "./utils.scss" }` subpath mapping is deleted from `packages/theme/package.json`. Additionally, `terrazzo-plugin-ds-token-fallbacks` is refactored to accept an `additionalScssFilenames` array, allowing the generated fallback map to be written to multiple output paths in a single build step. Stylelint configuration in `base-styles` is updated to reference the local helper instead of the removed theme export.

## Contribution

Opened by @mirka as a follow-up to #79470 to address transitive Sass dependency failures. During review, @aduth raised whether the `@wordpress/theme/utils` helper should remain publicly exported given existing alternatives; @mirka agreed to remove it for now rather than maintain dual APIs. The PR was merged by @aduth under commit `0fc4838`, with both credited as co-authors. A changelog entry was intentionally skipped since no package release occurred between the initial feature introduction and this amendment.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
