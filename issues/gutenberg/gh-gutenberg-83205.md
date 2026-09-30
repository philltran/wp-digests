# #83205: Column: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`2e72050`](https://github.com/WordPress/gutenberg/commit/2e720502afb25b51464895434b32ce15fbdec9f2)
- **Discussion:** [#83205](https://github.com/WordPress/gutenberg/pull/83205) · 2 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Column block (`core/column`) now supports background image, background size, and gradient via the `background` supports key in its `block.json`. This brings the block in line with other blocks (e.g. Group) that already expose these controls in the Background panel, as part of the broader design-tools consistency effort tracked in #43241. Because Column is a static block, the new supports are serialized onto the wrapper element by `useBlockProps.save()` and require no PHP render-callback changes.

## Impact

- **Site owners / editors:** The Background panel in the block inspector and in Global Styles (Blocks → Column → Background) now offers image, size, and gradient controls for individual columns. Gradient layers over the image (per #75859), so both render together.
- **Theme & plugin developers:** No code changes required. The legacy `color.gradients` support is retained alongside the new `background.gradient` (same dual-support pattern as the Group block), so existing gradient styling via the color panel continues to work. If you were previously working around the missing background supports with custom CSS or a filter, those workarounds can now be removed.
- **No breaking changes.** Existing saved posts and Global Styles that do not set the new properties are unaffected.

## Technical details

The diff modifies three files:

1. **`packages/block-library/src/column/block.json`** — adds a `background` supports object:

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

The `__experimentalDefaultControls` sub-object is a new (still experimental) field that controls which background controls are shown by default in the inspector. The pre-existing `color.gradients: true` entry is left in place alongside `background.gradient`.

2. **`packages/block-library/src/column/README.md`** — regenerated to document the three new `background` sub-supports.

3. **`docs/reference-guides/core-blocks/README.md`** — the Column entry's supports list is updated to include `background (backgroundImage, backgroundSize, gradient)`.

No PHP files are touched. Because `core/column` is a static block, `useBlockProps.save()` in the block's `save` function serializes the supported CSS custom properties onto the wrapper `<div>`, and the existing `wp_block_wrapper` / global-styles pipeline handles the rest. The gradient-over-image stacking behavior is inherited from the shared background-supports implementation referenced in #75859.

## Contribution

Opened by @aaronrobertshaw with @andrewserong as co-author. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." The two comments on the PR are both from `github-actions[bot]` (performance metrics and flaky-test report); no human review discussion is visible in the record. It is part of the design-tools consistency work tracked in #43241.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
