# #83232: Next Page: Add background gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`f6ac31a`](https://github.com/WordPress/gutenberg/commit/f6ac31a011b039d9741c3cc777fe7896e9f603fe)
- **Discussion:** [#83232](https://github.com/WordPress/gutenberg/pull/83232) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds `background.gradient` to the `core/query-pagination-next` block's `supports` in `block.json`, enabling the Background panel's gradient controls for the Next Page pagination block. The block previously supported color (including legacy `color.gradients`) and typography but not the newer Background panel gradient controls. This brings it in line with other core blocks as part of the design tools consistency work (related to #43241). No PHP change is required because the block is server-rendered through `get_block_wrapper_attributes()`, which serializes the new support onto the link element automatically.

## Impact

- **Block theme developers:** The Next Page block now responds to background gradient settings in Global Styles (Blocks → Next Page → Background) and in the block inspector. No code changes needed; the support is purely declarative in `block.json`.
- **Plugin developers:** No action required. The change is additive to a core block's supports; no new hooks, REST routes, or public APIs are introduced.
- **Site owners / editors:** If using a block theme with a Query Loop and pagination, the Next Page link can now be given a gradient background via Global Styles or per-block overrides, including responsive (mobile/desktop) variants.
- **No breaking changes.** The legacy `color.gradients` support is retained alongside the new `background.gradient` support, so existing gradient styling via the old path continues to work.

## Technical details

The functional change is a single addition to `packages/block-library/src/query-pagination-next/block.json`:

```json
"supports": {
  "anchor": true,
  "reusable": false,
  "html": false,
  "background": {
    "gradient": true
  },
  "color": {
    "gradients": true,
    "text": false,
    ...
  }
}
```

Because `core/query-pagination-next` is server-rendered through `get_block_wrapper_attributes()`, the new support serializes onto the `<a>` element in both the inherited and custom query paths without any PHP modification. When there is no next page, the block returns an empty string, so no empty box is painted.

The legacy `color.gradients` support is kept in place alongside `background.gradient`, preserving backward compatibility for themes that set gradients through the older color panel path.

Two documentation files are regenerated to reflect the new support:
- `docs/reference-guides/core-blocks/README.md` — the supports line for `core/query-pagination-next` now lists `background (gradient)`.
- `packages/block-library/src/query-pagination-next/README.md` — a new `background.gradient: true` entry is added under the supports list.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @ramonjd. The PR carries only 2 comments and 0 reactions, indicating a straightforward, uncontested merge. The author notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. It is part of the broader design tools consistency effort tracked in #43241, which aims to bring remaining core blocks up to parity with the Background panel's gradient controls.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
