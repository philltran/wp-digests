# #78698: Base Styles: Add wpds-var Sass helper for design token fallbacks

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Base styles`, `[Package] Theme`
- **Merged:** [`5439c7c`](https://github.com/WordPress/gutenberg/commit/5439c7cd849b4472a5695a5cbd70236c4545fa3f)
- **Discussion:** [#78698](https://github.com/WordPress/gutenberg/pull/78698) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `@wordpress/base-styles` package now includes a compile-time Sass helper (`wpds-var()`) that injects fallback values for `--wpds-*` design tokens directly in SCSS. This extends the existing `@wordpress/terrazzo-plugin-ds-token-fallbacks` build plugin to emit a generated SCSS map alongside the JS fallback map, allowing Sass consumers to use design tokens without requiring PostCSS build plugins or a runtime tokens stylesheet. The change also introduces an `outset-ring__focus` mixin that leverages this helper for consistent focus ring styling.

## Impact

- **Theme & plugin developers using Sass/SCSS:** Can now safely reference `--wpds-*` tokens in base-styles consumers without configuring PostCSS fallback injection or loading the tokens stylesheet.
- **Consumers of `@wordpress/base-styles`:** No action required; the new `outset-ring__focus` mixin is opt-in and does not alter existing styles unless explicitly included.
- **Build tooling:** `@wordpress/base-styles` now declares a dependency on `@wordpress/theme` to resolve the generated SCSS fallbacks.
- No breaking changes, deprecations, or migration steps required.

## Technical details

- Extends `packages/theme/bin/terrazzo-plugin-ds-token-fallbacks/index.ts` to generate `packages/theme/src/prebuilt/scss/design-token-fallbacks.scss` alongside the existing JS map.
- The generated file exports a `$fallbacks` Sass map and a `wpds-var($token)` function that returns `var(#{$token}, #{map.get($fallbacks, $token)})`.
- `@wordpress/base-styles/package.json` and `package-lock.json` now list `"@wordpress/theme": "file:../theme"` as a dependency.
- `packages/base-styles/_mixins.scss` imports the fallbacks via `@use "@wordpress/theme/src/prebuilt/scss/design-token-fallbacks" as wpds;` and defines the `@mixin outset-ring__focus()` mixin.
- `@wordpress/theme/package.json` adds a `./design-token-fallbacks` export pointing to the generated SCSS file.
- Before/after usage pattern:
  ```scss
  // Before (manual or build-plugin dependent)
  outline: var(--wpds-border-width-focus, 2px) solid var(--wpds-color-stroke-focus, #3858e9);

  // After (using the new helper)
  @use '@wordpress/theme/design-token-fallbacks' as wpds;
  outline: wpds.wpds-var('--wpds-border-width-focus') solid wpds.wpds-var('--wpds-color-stroke-focus');
  ```

## Contribution

Opened and merged by @mirka, with co-authorship from @ciampo. The PR addressed a gap in the design token adoption workflow by extending the existing Terrazzo fallback plugin to emit SCSS alongside JS, ensuring Sass-based consumers of `@wordpress/base-styles` aren't forced to alter their build pipelines. The discussion was minimal, focusing on implementation details and export paths.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
