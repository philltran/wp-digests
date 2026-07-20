# #80030: Fix global gap styles for Gallery block

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Gallery`, `Global Styles`
- **Merged:** [`664c2d7`](https://github.com/WordPress/gutenberg/commit/664c2d7960d2cd49e0dcfcab1b2e8e162c53b0b9)
- **Discussion:** [#80030](https://github.com/WordPress/gutenberg/pull/80030) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a bug where the Gallery block’s pseudo-grid layout width calculations ignored global gap styles when the block instance did not explicitly set a gap. It also corrects the global styles engine to properly output viewport-specific gap values in the editor for all gap-supporting blocks.

## Impact

- **Block & theme developers:** No breaking changes. Gallery blocks relying on global styles for `spacing.blockGap` will now render correctly in both editor and frontend.
- **Site owners:** Gallery layouts with global gap settings will now compute column widths accurately, preventing layout breaks when large gap values are applied.
- **No migration or code changes required.**

## Technical details

In `packages/block-library/src/gallery/gap-styles.js`, the component now fetches global styles via `blockEditorStore` and resolves `blockGap` using a priority chain: block instance → global Gallery styles (`globalStyles.blocks['core/gallery']`) → fallback. The same priority logic applies to viewport-specific media queries. In `packages/block-library/src/gallery/index.php`, `block_core_gallery_render()` statically caches `wp_get_global_styles()` and resolves `$block_gap` and `$viewport_gap` using the same chain before passing them to `block_core_gallery_get_column_gap_value()`. In `packages/global-styles-engine/src/core/render.tsx`, `getResponsiveStyleNodes()` now receives and forwards `fallbackGapValue`, `hasLayoutSupport`, `layoutHasBlockGapSupport`, and `layoutSelector` to ensure responsive gap styles render correctly in the editor. `packages/block-library/package.json` exports `wp_get_global_styles` to support the PHP changes.

## Contribution

Opened by @tellthemachines as a follow-up to #79984, which missed the global gap fallback logic. Merged after review by @andrewserong and others, with viewport handling refined during discussion. Co-authored by multiple contributors per the merge commit.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
