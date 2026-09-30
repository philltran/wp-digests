# #83199: Post Author Biography: Add background gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Post Author Biography`
- **Merged:** [`c58fd34`](https://github.com/WordPress/gutenberg/commit/c58fd34e699624cd4d3ceec6863827dc80db98d7)
- **Discussion:** [#83199](https://github.com/WordPress/gutenberg/pull/83199) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Post Author Biography block (`core/post-author-biography`) now supports the Background panel's gradient controls, adding `background.gradient` to its `block.json` supports. This brings the block in line with other core blocks as part of the ongoing design-tools consistency work (related to #43241). No PHP changes were required because the block is server-rendered through `get_block_wrapper_attributes()`, which serializes the new support onto the wrapper automatically.

## Impact

- **Site editors / theme developers:** The Author Biography block now exposes a Background → Gradient control in both Global Styles (Blocks → Author Biography → Background) and the block-level inspector. Gradients can be set globally, per-block, and per-responsive state (e.g. mobile vs. desktop).
- **Plugin & theme developers:** No code changes needed. The block's `block.json` gains a new `supports` entry; any theme or plugin that reads `block.json` supports will see `background.gradient: true`.
- **Existing sites:** No action required. The legacy `color.gradients` support is retained alongside the new `background.gradient`, so previously applied gradient styles continue to work. The block returns early when the author has no biography, so no empty gradient box is rendered.
- **Headless / REST consumers:** No schema or route changes; the support is purely a rendering concern handled by `get_block_wrapper_attributes()`.

## Technical details

The diff is minimal and confined to three files:

1. **`packages/block-library/src/post-author-biography/block.json`** — a new key is added to the `supports` object:

```json
"background": {
    "gradient": true
}
```

This sits alongside the existing `color.gradients: true` entry, which is left in place for backward compatibility.

2. **`packages/block-library/src/post-author-biography/README.md`** — regenerated to document the new `background.gradient` support.

3. **`docs/reference-guides/core-blocks/README.md`** — the core-blocks reference table for `core/post-author-biography` is updated to list `background (gradient)` in its supports column.

No PHP, JS, or CSS changes are present in the diff. The block's render path calls `get_block_wrapper_attributes()`, which reads the `supports` declaration and emits the corresponding CSS custom properties (e.g. `--wp--preset--gradient--*`) on the wrapper element. Because the block's render callback returns early when the author has no biography text, the gradient is only painted when there is content to display.

## Contribution

Opened by @aaronrobertshaw with co-authorship from shail-mehta. The PR notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." The discussion is minimal (2 comments, 0 reactions), with no visible design debate or rejected alternatives. It is part of the broader design-tools consistency effort tracked in #43241.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
