# #83253: Term Count: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`17ddd61`](https://github.com/WordPress/gutenberg/commit/17ddd61f4df6d16a042c2e0adc9ace08d8c7bad7)
- **Discussion:** [#83253](https://github.com/WordPress/gutenberg/pull/83253) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `core/term-count` block now supports the Background panel's image, size, and gradient controls in addition to the color, border, shadow, spacing, and typography supports it already had. Three new keys — `background.backgroundImage`, `background.backgroundSize`, and `background.gradient` — are added to the block's `block.json` supports. Because the block is server-rendered through `get_block_wrapper_attributes()`, the new supports serialize onto the wrapper automatically with no PHP change. This closes a gap in the design-tools consistency work tracked in #43241.

## Impact

- **Theme & site builders:** The Term Count block (used inside a Terms Query block's Term Template) now accepts background image, size, and gradient values from Global Styles and the block inspector, matching the behavior of other core blocks. No code changes required; the controls appear in the existing Background panel.
- **Plugin & theme developers:** No API change. The legacy `color.gradients` support remains alongside the new `background.gradient` support, so existing gradient styling on Term Count blocks is unaffected. No migration needed.
- **No action required** for existing sites. The block renders identically when no background values are set (it returns an empty string when there is no term, so no empty box is painted).

## Technical details

The change is confined to `packages/block-library/src/term-count/block.json`, where a new `background` object is added to the `supports` key:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true
}
```

This sits alongside the existing `color.gradients: true` support. Because `core/term-count` is server-rendered via `get_block_wrapper_attributes()`, the block-supports serialization pipeline handles the CSS output (background-image, background-size, and the gradient layer) without any PHP template change. The gradient renders on top of the background image, consistent with the layering behavior introduced in #75859.

Two documentation files are regenerated to reflect the new supports:
- `docs/reference-guides/core-blocks/README.md` — the Term Count entry's supports list now reads `background (backgroundImage, backgroundSize, gradient), color (background, gradients, text), …`
- `packages/block-library/src/term-count/README.md` — a new `background` bullet with the three sub-keys is inserted before the existing `color` bullet.

## Contribution

Opened by @aaronrobertshaw and co-authored with @andrewserong. The PR body notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions), with no recorded design debate or rejected alternatives. It was merged as `17ddd61`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
