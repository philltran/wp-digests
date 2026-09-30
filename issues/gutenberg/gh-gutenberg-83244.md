# #83244: RSS: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] RSS`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`85d5007`](https://github.com/WordPress/gutenberg/commit/85d5007f91c3a7a4050fa0c4d439a9f7c856e6b2)
- **Discussion:** [#83244](https://github.com/WordPress/gutenberg/pull/83244) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The RSS block (`core/rss`) now supports the Background panel's image, size, and gradient controls in Global Styles and the block inspector. Three new keys — `background.backgroundImage`, `background.backgroundSize`, and `background.gradient` — are added to the block's `block.json` supports, bringing the RSS block in line with other core blocks as part of the design-tools consistency effort (related to #43241). Because the block is server-rendered through `get_block_wrapper_attributes()` on its `<ul>` wrapper, the new supports serialize automatically with no PHP change.

## Impact

- **Site owners / editors:** The RSS block's Background panel in Global Styles and the block inspector now exposes image, size, and gradient controls. No code changes required; existing RSS blocks render unchanged until a background is applied.
- **Theme & plugin developers:** No breaking changes. The legacy `color.gradients` support remains alongside the new `background.gradient` key, so existing gradient styling via the color panel continues to work. If you build custom styling around the RSS block's wrapper, be aware that `background-image`, `background-size`, and gradient CSS may now appear on the `<ul>` element.
- **No action required** for existing sites or plugins. The change is purely additive to the block's supports declaration.

## Technical details

The diff modifies three files:

1. **`packages/block-library/src/rss/block.json`** — adds a `background` object under `supports`:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true
}
```

This sits alongside the existing `color` supports (which already included `background`, `text`, `link`, and `gradients`). The legacy `color.gradients` key is retained; `background.gradient` is the new, preferred path.

2. **`packages/block-library/src/rss/README.md`** — regenerated to document the new `background` supports entry.

3. **`docs/reference-guides/core-blocks/README.md`** — the core-blocks reference table for `core/rss` is updated to list `background (backgroundImage, backgroundSize, gradient)` in the supports column.

No PHP, JS, or CSS files are touched. The block's render callback already calls `get_block_wrapper_attributes()` on the `<ul>` wrapper, so the new supports serialize as inline styles or CSS custom properties on that element automatically. The wrapper only renders when the feed has items; error/empty states return placeholder markup without the wrapper, so no empty styled box appears. Per the PR description, the gradient layers over the background image (consistent with #75859), so both are visible simultaneously.

## Contribution

Authored by @aaronrobertshaw with co-authorship from @ramonjd. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." The discussion is minimal — two comments (both from the GitHub Actions bot posting performance metrics and the co-author list) and zero reactions — indicating a straightforward, uncontested merge with no design debate or rejected alternatives in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
