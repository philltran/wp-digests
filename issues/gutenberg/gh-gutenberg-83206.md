# #83206: Columns: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Columns`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`2b6608b`](https://github.com/WordPress/gutenberg/commit/2b6608b22c5c357d97a9a8e929c638a43ea2c4ac)
- **Discussion:** [#83206](https://github.com/WordPress/gutenberg/pull/83206) · 2 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Columns block (`core/columns`) now supports background image, background size, and gradient via the `background` supports key in its `block.json`. This closes a gap where Columns already supported color, border, layout, shadow, spacing, and typography but lacked the Background panel's image, size, and gradient controls. The change is purely declarative—no PHP render-callback modification is needed because Columns is a static block and `useBlockProps.save()` serializes the supports onto the wrapper element.

## Impact

- **Site owners / editors:** Columns blocks can now receive a background image (with `backgroundSize`), a gradient, or both (gradient layers over the image) via Global Styles → Blocks → Columns → Background, or per-instance in the block inspector. Responsive (mobile) overrides work as with other blocks.
- **Theme developers:** No code changes required. The new supports are handled by the block editor's CSS generation pipeline. If a theme ships custom CSS targeting `.wp-block-columns` backgrounds, it should account for the new `background-image`, `background-size`, and gradient properties that may now appear.
- **Plugin developers:** No breaking changes. The legacy `color.gradients` support is retained alongside the new `background.gradient` (same pattern as the Group block), so existing code reading `color.gradients` continues to work.
- **No action required** for most developers; the change is additive and opt-in through the editor UI.

## Technical details

The diff adds a single `background` object to `packages/block-library/src/columns/block.json`:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true,
  "__experimentalDefaultControls": {
    "backgroundImage": true,
    "gradient": true
  }
}
```

The `__experimentalDefaultControls` key enables the image and gradient controls in the block inspector by default (without requiring the user to toggle them on), while `backgroundSize` is available but not shown by default.

Because `core/columns` is a static block (no `render` callback in PHP), the supports are serialized onto the wrapper `<div class="wp-block-columns">` by `useBlockProps.save()` in the editor and by the block's static markup on the front end. No changes to `packages/block-library/src/columns/index.php` or any PHP file are present in the diff.

The gradient renders as a layer over the background image (per the pattern established in #75859), so both are visible simultaneously. The pre-existing `color.gradients` support (legacy gradient-as-color) is left in place alongside `background.gradient`, matching the Group block's dual-support approach.

The core-blocks reference (`docs/reference-guides/core-blocks/README.md`) and the block's own `packages/block-library/src/columns/README.md` are regenerated to list the new `background` supports.

## Contribution

Opened by @aaronrobertshaw as part of the design-tools consistency work tracked in #43241. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." @ramonjd is listed as a co-author. The record shows only 2 comments and 0 reactions—no visible design debate or rejected alternatives. Merged as `2b6608b`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
