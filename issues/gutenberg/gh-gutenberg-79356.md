# #79356: Grid: Add option to stretch columns with auto-fit for better layout flexibility

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @shrivastavanolo
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `[Feature] Layout`
- **Merged:** [`9268f64`](https://github.com/WordPress/gutenberg/commit/9268f648eb4dd90ab6e11fb55ea6bdab372dd74d)
- **Discussion:** [#79356](https://github.com/WordPress/gutenberg/pull/79356) · 1 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Adds a "Fill available space" toggle to the Grid block layout settings that switches the generated `grid-template-columns` CSS from `repeat(auto-fill, ...)` to `repeat(auto-fit, ...)`. This allows grid columns to stretch and collapse empty tracks instead of preserving them, giving authors explicit control over row-filling behavior without altering the default layout.

## Impact

- **Block & theme developers:** No breaking changes. The new `autoFit` boolean property is opt-in via the block.json schema and inspector controls.
- **Site owners / content editors:** Can enable "Fill available space" in the Grid block inspector to make columns stretch across the full row width when items don't evenly divide the grid.
- **No action required** for existing sites or themes; the default remains `auto-fill`.

## Technical details

- **Schema:** `schemas/json/block.json` adds an `autoFit` boolean (default `false`) to the `layout` object.
- **PHP:** `lib/block-supports/layout.php` updates `gutenberg_get_layout_style()` to include `autoFit` in the viewport override check and computes `$auto_placement = ! empty( $layout_for_styles['autoFit'] ) ? 'auto-fit' : 'auto-fill';`, which replaces the hardcoded `auto-fill` in the `grid-template-columns` declaration.
- **JS:** `packages/block-editor/src/layouts/grid.js` introduces `autoFit` to the layout prop validation, style generation (`getLayoutStyle`), and UI. A new `GridLayoutFillControl` component renders a `ToggleControl` labeled "Fill available space", which is conditionally shown via `showFillControl = ! layout?.isManualPlacement`. The generated CSS switches between `auto-fill` and `auto-fit` based on the `autoFit` flag.
- **Before/After:**
  ```css
  /* Default */
  grid-template-columns: repeat(auto-fill, minmax(min(12rem, 100%), 1fr));
  /* With "Fill available space" enabled */
  grid-template-columns: repeat(auto-fit, minmax(min(12rem, 100%), 1fr));
  ```

## Contribution

The PR was opened by @shrivastavanolo to close #57186 and merged after a single review cycle. The provided record carries no discussion detail regarding alternative approaches or design debates, and the implementation shipped as a straightforward opt-in toggle.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
