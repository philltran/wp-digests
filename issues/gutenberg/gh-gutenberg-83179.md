# #83179: RSS: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] RSS`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`c9010ef`](https://github.com/WordPress/gutenberg/commit/c9010ef7f7fb28d6da697723c56ba258342f8d9a)
- **Discussion:** [#83179](https://github.com/WordPress/gutenberg/pull/83179) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The RSS block (`core/rss`) now supports the `shadow` block support, allowing site owners to apply box-shadow presets via Global Styles or the block inspector. This is a `block.json`-only change: because the block is server-rendered through `get_block_wrapper_attributes()` on its `ul` wrapper, the shadow CSS custom properties serialize onto that element automatically with no PHP template modification. The change closes a gap in the design-tools consistency effort tracked in #43241, where the RSS block already supported color, border, and spacing but not shadow.

## Impact

- **Site owners / editors:** Can now set a shadow on the RSS block in Global Styles (Blocks → RSS → Shadow) or per-instance in the block inspector, including responsive (mobile/tablet/desktop) variants. No action required to adopt; the control simply appears.
- **Theme developers:** No code changes needed. The shadow serializes onto the block's `ul` wrapper via `get_block_wrapper_attributes()`. If a theme overrides the RSS block template, it must continue to call `get_block_wrapper_attributes()` on the wrapper element for the shadow to render.
- **Plugin developers:** No API change. The `shadow` key in `block.json` is the same mechanism used by every other core block that supports it.
- **No breaking changes, deprecations, or migrations.**

## Technical details

The functional change is a single line in `packages/block-library/src/rss/block.json`, adding `"shadow": true` to the `supports` object:

```json
"supports": {
  "spacing": { "margin": false },
  "shadow": true,
  "color": { "background": true, "text": true, "link": true, "gradients": true }
}
```

Because the RSS block is server-rendered and its wrapper `<ul>` receives its attributes from `get_block_wrapper_attributes()`, the shadow CSS custom properties (`--wp--custom--shadow--*`) are emitted onto that element with no change to the PHP render callback or the block's `render.php`.

Two documentation files are regenerated to reflect the new support:
- `docs/reference-guides/core-blocks/README.md` — the RSS entry's supports list gains `shadow`.
- `packages/block-library/src/rss/README.md` — a new bullet for `shadow: true` is added under the supports section.

## Contribution

Opened by @aaronrobertshaw and implemented by a Claude Code agent from a predefined task. @im3dabasia left a review but initially confused the demo video (it showed the Categories block rather than RSS); @aaronrobertshaw flagged the mix-up and @andrewserong provided a second review, after which the PR was merged. No design debate or alternative approaches are recorded in the four comments.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
