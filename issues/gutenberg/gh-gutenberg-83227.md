# #83227: Latest Comments: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Latest Comments`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`7d6131c`](https://github.com/WordPress/gutenberg/commit/7d6131c2f861919e7af4e7152699de979ec2f840)
- **Discussion:** [#83227](https://github.com/WordPress/gutenberg/pull/83227) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Latest Comments block (`core/latest-comments`) now supports the Background panel's image, size, and gradient controls in Global Styles and the block inspector. Previously the block only supported color (including legacy `color.gradients`), shadow, spacing, and typography. This change brings it in line with other core blocks as part of the ongoing design-tools consistency effort (related to #43241). Because the block is server-rendered through `get_block_wrapper_attributes()`, the new supports serialize onto the list wrapper automatically with no PHP change.

## Impact

- **Theme & site builders:** The Latest Comments block now accepts `backgroundImage`, `backgroundSize`, and `gradient` in Global Styles (Blocks → Latest Comments → Background) and in the block-level inspector. The gradient renders layered over the background image.
- **Plugin & theme developers:** No code changes required. The supports are declared in `block.json` and flow through the standard `get_block_wrapper_attributes()` path. The legacy `color.gradients` support remains alongside the new `background.gradient` for backward compatibility.
- **No action required** for existing sites; the block renders identically until a background image, size, or gradient is explicitly set.

## Technical details

The change is confined to `packages/block-library/src/latest-comments/block.json`, where a new `background` key is added to the `supports` object:

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

The `__experimentalDefaultControls` sub-object enables the image and gradient controls by default in the block inspector (the size control is available but not shown by default). The existing `color.gradients: true` entry is left in place so legacy gradient assignments continue to work.

No PHP, JS, or CSS files are modified. The block's server-side render path (`get_block_wrapper_attributes()`) picks up the new supports and emits the corresponding inline styles on the `<ul>` wrapper. The gradient layers over the background image per the behavior established in #75859.

Two documentation files are regenerated to reflect the new supports: `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/latest-comments/README.md`.

## Contribution

Authored by @aaronrobertshaw with co-authorship from @andrewserong. The PR body notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no visible design debate or rejected alternatives in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
