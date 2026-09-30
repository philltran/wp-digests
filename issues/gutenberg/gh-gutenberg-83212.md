# #83212: Comments: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Comments`
- **Merged:** [`d0b677f`](https://github.com/WordPress/gutenberg/commit/d0b677f159538bba803fa9a149b2cbc802f72650)
- **Discussion:** [#83212](https://github.com/WordPress/gutenberg/pull/83212) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Comments block (`core/comments`) now supports background image, background size, and gradient via the Background panel in Global Styles and the block inspector. This closes a gap where the block already supported color, border, spacing, and typography but not the Background panel's image/size/gradient controls, as part of the broader design-tools consistency effort (related #43241). No PHP changes are required because the block is server-rendered through `get_block_wrapper_attributes()`, which serializes the new supports onto the wrapper automatically.

## Impact

- **Site owners / editors:** Can now set a background image (with `cover`, `contain`, etc. sizing) and a gradient on the Comments block from Global Styles → Blocks → Comments → Background, or per-instance from the block inspector. The gradient layers over the image so both render together.
- **Theme developers:** The Comments block's `block.json` now declares `background.backgroundImage`, `background.backgroundSize`, and `background.gradient`. If a theme overrides or restyles the Comments wrapper, it should account for these new CSS properties. The legacy `color.gradients` support remains alongside `background.gradient` (same pattern as Post Content), so existing gradient styles via the color panel are unaffected.
- **No action required** for plugin or REST consumers; no new hooks, REST routes, or DB changes are introduced.

## Technical details

The functional change is entirely in `packages/block-library/src/comments/block.json`. A new `background` key is added to the `supports` object:

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

The `__experimentalDefaultControls` sub-object enables the image and gradient controls by default in the block inspector (the size control is available but not shown by default). The existing `color.gradients: true` entry is left in place alongside the new `background.gradient`, matching the dual-support pattern used by the Post Content block.

No PHP render changes are needed: the Comments block is server-rendered and its wrapper attributes are produced by `get_block_wrapper_attributes()`, which reads the `supports` declaration and emits the corresponding CSS custom properties and classes. The gradient layers over the background image per the behavior established in #75859.

Two documentation files are regenerated to list the new supports: `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/comments/README.md`.

## Contribution

Opened by @aaronrobertshaw with @andrewserong as co-author. The PR body notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no visible design debate or rejected alternatives; the approach of adding `block.json` supports and relying on `get_block_wrapper_attributes()` serialization is the standard pattern for this class of change.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
