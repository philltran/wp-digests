# #80093: DS: Name font weight tokens by intent

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] Editor`, `[Package] Block library`, `[Package] Block editor`, `[Package] Edit Post`, `[Package] Edit Widgets`, `[Package] Interface`, `[Package] Edit Site`, `[Package] Base styles`, `[Package] Preferences`, `[Package] Media Utils`, `[Package] Commands`, `[Package] DataViews`, `[Package] Fields`, `[Package] Theme`, `[Package] UI`
- **Merged:** [`0cf7ede`](https://github.com/WordPress/gutenberg/commit/0cf7ede6504680579f4cdb20de25f58adf4eed5b)
- **Discussion:** [#80093](https://github.com/WordPress/gutenberg/pull/80093) · 15 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The WordPress Design System renamed its typography font-weight tokens to reflect their intended use rather than numeric values. `--wpds-typography-font-weight-regular` is now `--wpds-typography-font-weight-default` (400), and `--wpds-typography-font-weight-medium` is now `--wpds-typography-font-weight-emphasis` (600). This shift allows the underlying numeric values to change in the future without breaking token references, while preserving weight 500 for authored site content and theme data.

## Impact

- **Plugin & theme developers:** The legacy Sass variables `$font-weight-regular` and `$font-weight-medium` are removed. You must migrate to the new CSS tokens: `var(--wpds-typography-font-weight-default)` or `var(--wpds-typography-font-weight-emphasis)`.
- **Block & UI developers:** Editor and component styles now consume the semantic tokens directly. No immediate action is required unless you were overriding these specific weights in custom stylesheets.
- **Site owners & content creators:** No action required. The block typography controls and Font Library continue to expose the full authored weight range, including 500.

## Technical details

The change lives primarily in `packages/base-styles/_variables.scss` and `packages/base-styles/_mixins.scss`, where the legacy `$font-weight-regular` (400) and `$font-weight-medium` (499) Sass variables were removed. The typography mixins now reference the design-system tokens directly:

```scss
// Before
font-weight: variables.$font-weight-medium;
// After
font-weight: wpds.var("--wpds-typography-font-weight-emphasis");
```

The fallback definitions in `packages/base-styles/internal/_wpds-token-fallbacks.scss` were updated to map `--wpds-typography-font-weight-default` to `400` and `--wpds-typography-font-weight-emphasis` to `600`. Across `packages/block-editor`, `packages/block-library`, `packages/components`, and related UI packages, hardcoded weights (e.g., `font-weight: 600;` or `$font-weight-medium`) were replaced with `var(--wpds-typography-font-weight-emphasis)`. The canonical system font stack was also aligned in `packages/base-styles/_variables.scss` to use `-apple-system, system-ui, "Segoe UI", Roboto, Oxygen-Sans, Ubuntu, Cantarell, "Helvetica Neue", sans-serif` for both headings and body. The legacy Components JS typography config retains its numeric value due to Emotion build constraints, but its internal helper was renamed to `fontWeightEmphasis`.

## Contribution

Opened by @ciampo as a follow-up to #80020, the PR was merged following review feedback from @jasmussen, @aduth, @mirka, @simison, and @t-hamano. Early in the review cycle, an AI-assisted migration initially overrode weights in content-facing areas like the global styles UI and Font Library; @ciampo reverted those content-specific changes to preserve the intended 500 weight for authored content. The team also discussed aligning quick-inserter popover items with the broader menu-item pattern, deferring further UI typography iterations to the 7.1 release testing period.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
