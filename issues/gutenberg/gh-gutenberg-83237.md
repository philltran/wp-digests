# #83237: Post Template: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Post Template`
- **Merged:** [`02c5e5a`](https://github.com/WordPress/gutenberg/commit/02c5e5aeaa9188f93d109a3eadca61a29267708a)
- **Discussion:** [#83237](https://github.com/WordPress/gutenberg/pull/83237) · 2 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Post Template block (`core/post-template`) now supports background image, background size, and gradient via the `background` supports key in its `block.json`. Previously the block only supported color (including legacy `color.gradients`), border, layout, spacing, and typography, leaving the Background panel's image and gradient controls unavailable. This brings the block in line with other core blocks as part of the design tools consistency effort tracked in #43241. No PHP changes were needed because the block is server-rendered through `get_block_wrapper_attributes()`, which serializes the new supports onto the wrapper element automatically.

## Impact

- **Block theme developers / site builders:** The Post Template block (used inside Query Loop for single-post rendering) now accepts `backgroundImage`, `backgroundSize`, and `gradient` in both Global Styles (Blocks → Post Template → Background) and the block inspector. The gradient layers over the image, so both render together. Responsive (mobile) values are supported via the States menu.
- **Plugin & theme developers:** No code changes required. The legacy `color.gradients` support remains alongside the new `background.gradient`, so existing styles that target the old key continue to work. No new hooks, REST routes, or DB changes.
- **No action required** for existing sites; the change is purely additive. Existing Post Template blocks render identically until a user explicitly sets a background image or gradient.

## Technical details

The diff modifies three files:

1. **`packages/block-library/src/post-template/block.json`** — adds a `background` object under `supports`:

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

The `__experimentalDefaultControls` key enables the image and gradient controls in the block inspector by default (the size control is not shown by default). The existing `color.gradients: true` entry is left in place so legacy gradient styles continue to resolve.

2. **`packages/block-library/src/post-template/README.md`** — regenerated to document the new `background` supports sub-keys.

3. **`docs/reference-guides/core-blocks/README.md`** — the core blocks reference table is updated to list `background (backgroundImage, backgroundSize, gradient)` in the Post Template supports column.

No PHP render changes: the block's server-side rendering path calls `get_block_wrapper_attributes()`, which reads the registered supports and emits the corresponding inline styles (background-image, background-size, background gradient) on the wrapper `<div>`. When the query returns no posts the function returns an empty string, so no empty styled box is rendered. The gradient-over-image stacking behavior is inherited from the shared background supports implementation referenced in #75859.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @ramonjd. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." The discussion is minimal (2 comments, 0 reactions) with no recorded design debate or rejected alternatives; the approach of adding `background` supports alongside the existing `color.gradients` key was straightforward and consistent with how other core blocks handle the same transition.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
