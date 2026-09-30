# #83251: Table of Contents: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Table of contents (experimental)`
- **Merged:** [`70eedab`](https://github.com/WordPress/gutenberg/commit/70eedab17e00ddb588d4d4b8ebdb0cb0f4dd7a84)
- **Discussion:** [#83251](https://github.com/WordPress/gutenberg/pull/83251) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The experimental Table of Contents block now supports the Background panel's image, size, and gradient controls in Global Styles and the block inspector. Three new keys — `background.backgroundImage`, `background.backgroundSize`, and `background.gradient` — were added to the block's `block.json` supports. Because the block is server-rendered via `get_block_wrapper_attributes()`, the supports serialize onto the wrapper automatically with no PHP change. This brings the block in line with other core blocks as part of the ongoing design-tools consistency effort (related #43241).

## Impact

- **Site owners / editors:** The Table of Contents block (still experimental) now accepts a background image, a size setting (e.g. Contain, Cover), and a gradient from the Background panel in Global Styles or the block inspector. The gradient layers over the image so both are visible simultaneously.
- **Plugin & theme developers:** No code changes required. The new supports are purely additive in `block.json`; the block's render path (`get_block_wrapper_attributes()`) picks them up automatically. The legacy `color.gradients` support remains alongside the new `background.gradient` key, so existing gradient styling is unaffected.
- **No action required** for existing sites or themes. The block is experimental and the change is backward-compatible.

## Technical details

The diff adds a single `background` object to the `supports` key in `packages/block-library/src/table-of-contents/block.json`:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true
}
```

This sits alongside the existing `color` supports (which already include `gradients` for the legacy gradient API). Because the Table of Contents block is server-rendered through `get_block_wrapper_attributes()`, the new supports are serialized onto the wrapper element's inline styles and CSS custom properties without any change to the block's `render.php` or PHP render callback. When the block has no headings it returns an empty string, so no empty box is painted with the new background.

The gradient layers over the background image (behavior established in #75859), so both render together. The core-blocks reference (`docs/reference-guides/core-blocks/README.md`) and the block's own `README.md` were regenerated to list the new supports.

## Contribution

Authored by @aaronrobertshaw with co-authorship from @ramonjd. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." Discussion was minimal (2 comments, 0 reactions) with no visible design debate; the change follows the established pattern of adding Background-panel supports to core blocks. The gradient-over-image stacking behavior is inherited from the work in #75859 rather than introduced here.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
