# #79984: Enable default gap processing on Gallery block

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Gallery`, `[Feature] Style States`
- **Merged:** [`77efbc9`](https://github.com/WordPress/gutenberg/commit/77efbc9e591ed6657a5c2d70fb6f0ab12ad165c6)
- **Discussion:** [#79984](https://github.com/WordPress/gutenberg/pull/79984) · 5 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

The Gallery block now enables default gap serialization, removing the previous `__experimentalSkipSerialization` flag that forced a custom output path. This change ensures viewport-specific block gaps are automatically serialized and that the `--wp--style--unstable-gallery-gap` CSS variable is correctly output within responsive media queries on both the frontend and PHP-rendered templates.

## Impact

- **Block & Theme Developers**: Galleries now serialize `blockGap` natively. If you relied on previous custom gap serialization logic or manual workarounds to inject responsive gap styles, those patterns are no longer needed.
- **Theme Authors**: The block now outputs the `--wp--style--unstable-gallery-gap` variable inside per-viewport media queries, allowing themes to reliably calculate column widths using CSS `calc()` without hardcoding breakpoint logic or relying on PHP-only fallbacks.
- **Hosting & Platform Teams**: No configuration changes required. The shift back to default serialization aligns the Gallery block with other core blocks and improves consistency across responsive style states.

## Technical details

- **`block.json`**: Replaces the array-based `blockGap` config with a structured object and removes `__experimentalSkipSerialization`. The new default fallback is `var( --wp--style--gallery-gap-default, var( --gallery-block--gutter-size, var( --wp--style--block-gap, 0.5em ) ) )`.
- **JS (`gap-styles.js`)**: Imports `privateApis as globalStylesEnginePrivateApis` from `@wordpress/global-styles-engine` and unlocks `getResponsiveMediaQueries`. The component now iterates through viewport media queries to output a dedicated rule for `--wp--style--unstable-gallery-gap` per breakpoint.
- **PHP (`index.php`)**: Extracts the legacy sanitization/variable substitution logic into a new `block_core_gallery_get_column_gap_value()` function. Adds responsive CSS rule generation by iterating over breakpoints returned by `WP_Theme_JSON_Gutenberg::get_viewport_media_queries()` and `WP_Theme_JSON::get_viewport_media_queries()`, attaching the gallery gap variable to each breakpoint's media query.
- **Before/After Pattern**:
```diff
// Previously (skipped default serialization, manual gap output)
"blockGap": [ "horizontal", "vertical" ],
"__experimentalSkipSerialization": [ "blockGap" ],

// Now (native serialization + responsive fallback)
"blockGap": {
    "sides": [ "horizontal", "vertical" ],
    "__experimentalDefault": "var( --wp--style--gallery-gap-default, var( --gallery-block--gutter-size, var( --wp--style--block-gap, 0.5em ) ) )"
}
```
- **Runtime Behavior**: The Gallery block now respects the full `style` object (including viewport states) instead of only reading `attributes.style?.spacing?.blockGap`, ensuring editor and frontend renderers stay in sync with responsive gap values.

## Contribution

Opened by @tellthemachines to address skipped gap serialization and missing viewport styles in the Gallery block. Reviewed and approved by @andrewserong, @talldan, @t-hamano, and @andreawetz. The initial merge enabled responsive gap output, with reviewers noting it likely resolves #56554 and #60505. A follow-up PR (#80030) was opened to address remaining editor-side viewport state rendering issues that were not covered in this merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
