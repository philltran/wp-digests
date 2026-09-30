# #83217: Comments Title: Add background gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`3c27aaa`](https://github.com/WordPress/gutenberg/commit/3c27aaab9382d75ee4df9bac79225f5fa620985a)
- **Discussion:** [#83217](https://github.com/WordPress/gutenberg/pull/83217) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Comments Title block (`core/comments-title`) now supports the Background panel's gradient controls via the `background.gradient` block support. This closes a gap where the block already supported color (including legacy `color.gradients`), border, spacing, and typography, but not the newer Background panel gradient. The change is part of the broader design-tools consistency effort tracked in #43241. No PHP changes are required because the block is server-rendered through `get_block_wrapper_attributes()`, which serializes the support onto the wrapper automatically.

## Impact

- **Theme developers / Global Styles editors:** The Comments Title block now exposes a Background → Gradient control in the Site Editor and block inspector. Gradients can be set at the Global Styles level (including responsive/mobile states) and overridden per block instance.
- **No breaking changes.** The legacy `color.gradients` support is retained alongside the new `background.gradient`, so existing themes that style the block via the older color-gradients path are unaffected.
- **No action required** for existing sites or plugins. The block returns early for password-protected posts and posts with zero comments, so no empty gradient box is rendered in those cases.

## Technical details

The functional change is a single addition to `packages/block-library/src/comments-title/block.json`:

```json
"supports": {
  "anchor": true,
  "align": true,
  "html": false,
  "background": {
    "gradient": true
  },
  "__experimentalBorder": { "radius": true, "color": true },
  "color": { "background": true, "gradients": true, "text": true },
  "spacing": { "margin": true, "padding": true },
  "typography": { "fontSize": true, "lineHeight": true, "textAlign": true }
}
```

Because `core/comments-title` is server-rendered via `get_block_wrapper_attributes()`, the new support serializes onto the wrapper element without any PHP-side change. The legacy `color.gradients: true` entry is kept in parallel with the new `background.gradient: true`.

Two documentation files are regenerated to reflect the new support:
- `docs/reference-guides/core-blocks/README.md` — the supports list for `core/comments-title` now reads `background (gradient), color (background, gradients, text), …`
- `packages/block-library/src/comments-title/README.md` — a new `background` → `gradient: true` entry is added under the supports table.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @ramonjd. The PR carries only 2 comments and 0 reactions, indicating a straightforward, uncontested merge. The author notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task, consistent with the routine nature of adding a standard block support to a single core block as part of the #43241 consistency sweep.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
