# #83236: Pagination: Add background gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Query Pagination`
- **Merged:** [`13e909f`](https://github.com/WordPress/gutenberg/commit/13e909faddc101ae2e02b1391b538fcad8b767be)
- **Discussion:** [#83236](https://github.com/WordPress/gutenberg/pull/83236) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds `background.gradient` support to the `core/query-pagination` block, enabling the Background panel's gradient controls in both Global Styles and the block inspector. This brings the Pagination block in line with other core blocks as part of the ongoing design-tools consistency effort (related #43241). Because the block is server-rendered through `get_block_wrapper_attributes()`, the support serializes onto the wrapper element automatically and no PHP change is required.

## Impact

- **Site owners / block-theme users:** Can now apply a background gradient to the Pagination block via Global Styles (Blocks → Pagination → Background) or the block inspector. Block-level gradients override Global Styles; responsive (mobile) variants are supported.
- **Plugin & theme developers:** No code changes required. The support is declarative in `block.json`. The legacy `color.gradients` support remains alongside the new `background.gradient`, so existing gradient assignments via the old path continue to work.
- **No action required** for existing sites; the change is purely additive.

## Technical details

The diff is confined to three files:

1. **`packages/block-library/src/query-pagination/block.json`** — adds the new support declaration:

```json
"background": {
  "gradient": true
}
```

This sits alongside the existing `"color": { "gradients": true, "link": true, "text": true, "background": true }` entry. Both the legacy `color.gradients` and the new `background.gradient` supports are active simultaneously.

2. **`packages/block-library/src/query-pagination/README.md`** — regenerated to list the new `background.gradient` support.

3. **`docs/reference-guides/core-blocks/README.md`** — the core-blocks reference table for `core/query-pagination` is updated to include `background (gradient)` in the supports list.

No PHP, JS, or CSS changes. The block renders server-side via `get_block_wrapper_attributes()`, which reads the `background.gradient` support and emits the corresponding `background-image` style on the wrapper. When no gradient is set the function returns an empty string, so no empty box is painted.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @shail-mehta. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no visible design debate or rejected alternatives; it is a straightforward declarative addition consistent with the broader design-tools consistency track.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
