# #83257: Terms List: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Categories`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`d6d3e68`](https://github.com/WordPress/gutenberg/commit/d6d3e68bb8d9e34b9d76a1018b1772ace5687922)
- **Discussion:** [#83257](https://github.com/WordPress/gutenberg/pull/83257) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Terms List block (`core/categories`) now supports the Background panel's image, size, and gradient controls in both Global Styles and the block inspector. Previously the block supported color (including legacy gradients), border, spacing, and typography, but lacked the `background` supports group. This change brings the block in line with other core blocks as part of the ongoing design-tools consistency effort (related to #43241).

## Impact

- **Site builders / theme developers:** The Terms List block now accepts `backgroundImage`, `backgroundSize`, and `gradient` via Global Styles (Blocks → Terms List → Background) and the block-level inspector. No code changes required; the supports serialize through `get_block_wrapper_attributes()` automatically.
- **Plugin & theme developers:** No breaking changes. The legacy `color.gradients` support remains alongside the new `background.gradient`. No PHP or template changes are needed.
- **No action required** for existing sites — the block renders identically until a background is explicitly set.

## Technical details

The change is confined to `packages/block-library/src/categories/block.json`, adding a `background` object under `supports`:

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

Because `core/categories` is server-rendered through `get_block_wrapper_attributes()` in both its list mode (the `<ul>`) and dropdown mode (the wrapper around the label and `<select>`), the new supports serialize onto the appropriate element with no PHP modification. The gradient layers over the background image (per the behavior established in #75859), so both are visible simultaneously. The `__experimentalDefaultControls` key enables the image and gradient controls in the block inspector by default, while `backgroundSize` is available but not shown by default.

The core-blocks reference (`docs/reference-guides/core-blocks/README.md`) and the block's own `README.md` were regenerated to list the new supports.

## Contribution

Opened by @aaronrobertshaw and co-authored with @ramonjd. The PR body notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no recorded design debate or rejected alternatives — it follows the established pattern of adding missing `background` supports to a core block.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
