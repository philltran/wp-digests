# #79470: Base Styles: Reapply wpds-var Sass helper

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Base styles`, `[Package] Theme`
- **Merged:** [`cb8e665`](https://github.com/WordPress/gutenberg/commit/cb8e665a35e6af6a9225d31bd392598974d8029e)
- **Discussion:** [#79470](https://github.com/WordPress/gutenberg/pull/79470) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/theme` package now exports a generated Sass helper that injects compile-time fallback values for `--wpds-*` design tokens. This restores functionality previously reverted, ensuring `@wordpress/base-styles` and custom SCSS can safely reference design tokens without relying on the PostCSS token-fallback plugin or runtime CSS variable definitions.

## Impact

- Theme & block developers: No immediate action required unless you are compiling SCSS outside standard build pipelines; the new utility resolves compile-time token fallbacks.
- Package consumers: `@wordpress/base-styles` now declares a direct dependency on `@wordpress/theme`, which may affect monorepo or bundler configurations.
- Custom Sass authors: Can now import `@wordpress/theme/utils` to safely resolve tokens at build time, preventing compilation failures in environments lacking token metadata.

## Technical details

- The diff reverts a prior removal and extends the `@wordpress/terrazzo-plugin-ds-token-fallbacks` script (`packages/theme/bin/terrazzo-plugin-ds-token-fallbacks/index.ts`) to generate an additional `.scss` file alongside the existing JS module. This generated file creates a `$fallbacks` Sass map paired with a `@function var($token)` that returns `var(--token, <fallback>)` or throws an error for unknown tokens.
- The utility is publicly exported via `packages/theme/package.json` under the field `"./utils": { "sass": "./utils.scss" }`.
- `packages/base-styles/package.json` gains a `"dependencies": { "@wordpress/theme": "file:../theme" }` entry to satisfy the new import path.
- `packages/base-styles/_mixins.scss` demonstrates usage by importing the helper as `wpds` and applying it to a new `outset-ring__focus()` mixin:
  ```scss
  @use "@wordpress/theme/utils" as wpds;
  
  @mixin outset-ring__focus() {
    outline: wpds.var("--wpds-border-width-focus") solid wpds.var("--wpds-color-stroke-focus");
    outline-offset: wpds.var("--wpds-border-width-focus");
  }
  ```
- Documentation in `packages/theme/README.md` is updated to explicitly recommend this helper when build plugins are unavailable.

## Contribution

Open and merged by @mirka, with code review from @aduth and @simison. The PR reapplies Sass helper work previously reverted in #79429, stacking changes on top of #79471. Review discussion focused on namespace scope pollution when using legacy `@import` versus modern `@use`, ultimately deciding to keep the current implementation while noting `@use` is the recommended approach. Merged as commit `cb8e665`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
