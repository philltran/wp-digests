# #83234: Page List: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Page List`
- **Merged:** [`1159170`](https://github.com/WordPress/gutenberg/commit/115917060c72dba01acb6347630eafbab8efee59)
- **Discussion:** [#83234](https://github.com/WordPress/gutenberg/pull/83234) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Page List block (`core/page-list`) now supports background image, background size, and gradient via the Background panel in Global Styles and the block inspector. Previously it supported color (including legacy `color.gradients`), border, shadow, spacing, and typography, but lacked the Background panel's image, size, and gradient controls. This brings it in line with other core blocks as part of the ongoing design tools consistency work (related to #43241).

## Impact

- **Site owners / content editors:** Can now set a background image (with `cover`, `contain`, etc. sizing) and a gradient on the Page List block through Global Styles → Blocks → Page List → Background, or per-instance in the block inspector. Gradient layers over the image so both are visible simultaneously.
- **Theme & plugin developers:** No code changes required. The block is server-rendered through `get_block_wrapper_attributes()`, so the new supports serialize onto the list wrapper automatically. The legacy `color.gradients` support remains alongside the new `background.gradient` for backward compatibility.
- **No action required** for existing sites; the change is purely additive to the block's supported design controls.

## Technical details

The change is confined to `packages/block-library/src/page-list/block.json` and two regenerated documentation files. The `supports` object gains a new `background` key:

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

The `__experimentalDefaultControls` sub-object enables the image and gradient controls by default in the block inspector (as opposed to hiding them behind a toggle). The existing `color` supports object (with `text`, `background`, `gradients`, `link`) is untouched, so the legacy `color.gradients` path continues to work alongside the new `background.gradient`.

Because Page List is server-rendered via `get_block_wrapper_attributes()`, the supports are serialized onto the `<ul>` wrapper element with no PHP template or render-callback modification. The block returns early when no pages exist; the one known edge case where every published page has an unpublished parent still renders an empty list, and the background styles paint that empty box the same way border, spacing, and shadow already do.

The gradient-over-image stacking behavior follows the pattern established in #75859. The core blocks reference (`docs/reference-guides/core-blocks/README.md`) and the block's own `README.md` are regenerated to list the new supports.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @ramonjd. The PR body notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no visible design debate or rejected alternatives; it is a straightforward application of an existing supports pattern to one block as part of the broader #43241 consistency effort.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
