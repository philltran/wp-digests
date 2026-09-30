# #83221: Post Excerpt: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Block] Post Excerpt`, `[Feature] Design Tools`
- **Merged:** [`87d68fb`](https://github.com/WordPress/gutenberg/commit/87d68fbe5556b2ade6e83572f0b79c070391f549)
- **Discussion:** [#83221](https://github.com/WordPress/gutenberg/pull/83221) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Post Excerpt block (`core/post-excerpt`) now supports background image, background size, and gradient through the `background` supports key in its `block.json`. This brings the block in line with other core blocks as part of the ongoing design-tools consistency work (tracked in #43241). No PHP or JavaScript changes were required because the block is server-rendered via `get_block_wrapper_attributes()`, which serializes the new supports onto the wrapper element automatically.

## Impact

- **Theme developers / site builders:** The Excerpt block now exposes Background Image, Background Size, and Gradient controls in both Global Styles (Blocks → Excerpt → Background) and the block inspector. No code changes needed; the controls appear automatically once the block is updated.
- **Plugin & theme developers:** If you build custom styling or overrides for the Excerpt block, the new `background` supports keys (`backgroundImage`, `backgroundSize`, `gradient`) are now available. The legacy `color.gradients` support is retained alongside `background.gradient`, so existing gradient styling via the color panel continues to work.
- **No action required** for existing sites. The change is purely additive; blocks without these supports set render identically to before.

## Technical details

The diff is confined to three files:

1. **`packages/block-library/src/post-excerpt/block.json`** — a `background` object is added to the `supports` key:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true
}
```

2. **`packages/block-library/src/post-excerpt/README.md`** — regenerated to document the three new sub-supports under the `background` entry.

3. **`docs/reference-guides/core-blocks/README.md`** — the Excerpt block's supports line is updated to include `background (backgroundImage, backgroundSize, gradient)`.

No PHP, JS, or CSS files are touched. Because `core/post-excerpt` is server-rendered through `get_block_wrapper_attributes()`, the supports serialize onto the wrapper `<div>` and the corresponding CSS classes are emitted by the block-supports machinery. The block returns an empty string when no post is present, so there is no empty box to paint. Per the related work in #75859, the gradient layers over the background image so both render together. The pre-existing `color.gradients` support is left in place alongside the new `background.gradient`.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @andrewserong. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The record shows only 2 comments (both from the GitHub Actions bot) and no design debate or alternative approaches discussed. It is one of a series of PRs under the #43241 design-tools consistency umbrella that progressively add the Background panel's image/size/gradient controls to core blocks that previously lacked them.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
