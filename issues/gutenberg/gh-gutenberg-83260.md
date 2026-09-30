# #83260: Post Title: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Post Title`
- **Merged:** [`f36e6b3`](https://github.com/WordPress/gutenberg/commit/f36e6b3f1c29606a3d05fbbef75bb18bb2e70ff4)
- **Discussion:** [#83260](https://github.com/WordPress/gutenberg/pull/83260) · 2 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Post Title block (`core/post-title`) now supports background image, background size, and gradient via the `background` supports key in its `block.json`. This brings the block in line with other core blocks that already expose these controls, as part of the ongoing design-tools consistency effort (related to #43241). Because the block is server-rendered through `get_block_wrapper_attributes()`, the new supports serialize onto the heading element automatically with no PHP change required.

## Impact

- **Site owners / block-theme users:** The Post Title block now shows Background image, size, and gradient controls in both the block inspector and Global Styles (Blocks → Title → Background). The gradient layers over the image so both are visible simultaneously.
- **Theme developers:** No code changes required. The supports serialize as CSS custom properties on the rendered heading via `get_block_wrapper_attributes()`. If your theme applies its own background styling to `.wp-block-post-title`, the new block-level and Global Styles values will layer on top.
- **Plugin developers:** No action required. The legacy `color.gradients` support is retained alongside the new `background.gradient`, so existing gradient styling via the color panel continues to work.
- **No breaking changes.** The block still returns an empty string when no post or title exists, so no empty styled box is painted.

## Technical details

The diff adds a `background` object to `packages/block-library/src/post-title/block.json`:

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

The `__experimentalDefaultControls` key enables the image and gradient controls by default in the block inspector (size is opt-in). The pre-existing `color.gradients: true` support is left in place alongside `background.gradient`, matching the pattern on other blocks that have adopted the new background supports.

No PHP render-callback change is needed: `core/post-title` is server-rendered through `get_block_wrapper_attributes()`, which reads the registered supports and emits the corresponding CSS custom properties (`--wp--custom--background-image`, `--wp--custom--background-size`, `--wp--custom--gradient`) on the heading element. The block's render function returns an empty string when there is no post or title, preventing an empty styled box.

The gradient layers over the background image (per #75859), so both are visible together. Global Styles values for this block also apply to the Title block in the theme template itself.

Two documentation files are regenerated to list the new supports: `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/post-title/README.md`.

## Contribution

Opened by @aaronrobertshaw with @talldan as co-author. The PR notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, both from the GitHub Actions bot), with no visible design debate or rejected alternatives. It is part of the broader design-tools consistency work tracked in #43241.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
