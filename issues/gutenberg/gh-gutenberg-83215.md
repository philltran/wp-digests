# #83215: Comments Next Page: Add background gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`607ff0b`](https://github.com/WordPress/gutenberg/commit/607ff0b6f2e1fead927298f7db19f82b443f192c)
- **Discussion:** [#83215](https://github.com/WordPress/gutenberg/pull/83215) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Comments Next Page block (`core/comments-pagination-next`) now supports the Background panel's gradient controls via the `background.gradient` block support. This closes a gap where the block already supported color (including legacy `color.gradients`) and typography but not the newer Background gradient API, as part of the broader design-tools consistency effort tracked in #43241. No PHP change was required because the block is server-rendered through `get_block_wrapper_attributes()`, which serializes the support onto the link element automatically.

## Impact

- **Theme developers / Global Styles authors:** The Comments Next Page block now appears under Background in the Global Styles panel with gradient controls, and block-level gradient overrides work in the block inspector. Responsive (mobile/tablet) gradient overrides are also supported.
- **Plugin developers:** No action required. The change is additive to a core block's `block.json` supports; no new hooks, REST routes, or PHP APIs are introduced.
- **Site owners:** No visible change unless a gradient is explicitly applied to the block via Global Styles or the block inspector.
- **No breaking changes.** The legacy `color.gradients` support is retained alongside the new `background.gradient` key, so existing gradient assignments continue to work.

## Technical details

The diff adds a single entry to the `supports` object in `packages/block-library/src/comments-pagination-next/block.json`:

```json
"background": {
  "gradient": true
}
```

This sits alongside the existing `"color": { "gradients": true, "text": false, … }` entry. Because `core/comments-pagination-next` is server-rendered, the support is picked up by `get_block_wrapper_attributes()` and serialized as inline styles on the `<a>` element — no `render.php` or PHP template change is needed. When the block has no next page to link to, it returns an empty string, so no empty gradient box is painted.

The two README files (`docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/comments-pagination-next/README.md`) are regenerated to list the new `background (gradient)` support.

## Contribution

Authored by @aaronrobertshaw with co-authorship from @shail-mehta. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." The record carries no design debate or review discussion beyond the automated performance metrics; it merged cleanly with no substantive back-and-forth.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
