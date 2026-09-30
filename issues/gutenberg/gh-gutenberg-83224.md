# #83224: Footnotes: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Footnotes`
- **Merged:** [`7f0c39e`](https://github.com/WordPress/gutenberg/commit/7f0c39e716162f1248a059d254113b04560307ce)
- **Discussion:** [#83224](https://github.com/WordPress/gutenberg/pull/83224) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Footnotes block (`core/footnotes`) now supports the Background panel's image, size, and gradient controls in the block editor and Global Styles. Previously the block only supported color (including legacy `color.gradients`), border, shadow, spacing, and typography. This change brings the block in line with other core blocks as part of the ongoing design-tools consistency effort (related to #43241). No PHP changes are required because the block is server-rendered through `get_block_wrapper_attributes()`, which serializes the new supports onto the wrapper automatically.

## Impact

- **Site owners / editors:** The Footnotes block now accepts a background image (with `backgroundSize`), a `background.gradient`, and the corresponding `__experimentalDefaultControls` in both Global Styles and the block inspector. The gradient layers over the image (per #75859). No action required; existing posts render unchanged.
- **Theme developers:** If you style `.wp-block-footnotes` in your theme, be aware that the block can now emit `background-image`, `background-size`, and `background` (gradient) CSS via the wrapper attributes. The legacy `color.gradients` support remains alongside the new `background.gradient`, so both code paths can produce gradient output.
- **Plugin developers:** No API change. The new supports use the standard block-supports infrastructure; no new hooks, filters, or REST schema fields are introduced.
- **No breaking changes or migrations required.**

## Technical details

The diff adds a `background` object to the `supports` key in `packages/block-library/src/footnotes/block.json`:

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

Because the Footnotes block is server-rendered via `get_block_wrapper_attributes()`, these supports serialize directly onto the wrapper `<div>` as inline styles or CSS custom properties—no PHP render-callback change is needed. The block returns an empty string when the post has no footnotes, so no empty styled box is painted.

The legacy `color.gradients` support (already present in the block's `supports.color` object) is retained alongside the new `background.gradient`, meaning both the old and new gradient paths coexist.

Two documentation files are regenerated to list the new supports:
- `docs/reference-guides/core-blocks/README.md` — the Footnotes entry's supports line now reads `background (backgroundImage, backgroundSize, gradient)`.
- `packages/block-library/src/footnotes/README.md` — a new `background` bullet with the three sub-keys is added.

## Contribution

Opened by @aaronrobertshaw and co-authored with @ramonjd. The PR notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions), with no visible design debate or rejected alternatives. Merged as `7f0c39e`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
