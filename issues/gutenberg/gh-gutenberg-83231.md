# #83231: Math: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Math`
- **Merged:** [`3332a5c`](https://github.com/WordPress/gutenberg/commit/3332a5c3cceed8c491ba64e2e9604636ae141ee2)
- **Discussion:** [#83231](https://github.com/WordPress/gutenberg/pull/83231) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Math block (`core/math`) now supports background image, background size, and gradient via the Background panel in Global Styles and the block inspector. Previously the block supported color (including legacy gradients), border, shadow, spacing, and typography, but lacked the Background panel's image and gradient controls. This change brings the block in line with other core blocks as part of the ongoing design-tools consistency effort (related to #43241).

## Impact

- **Site owners / editors:** Can now set a background image (with `Contain`, `Cover`, etc. sizing) and a gradient on Math blocks through Global Styles → Blocks → Math → Background, or per-instance in the block inspector. Gradient layers over the image so both are visible simultaneously.
- **Plugin & theme developers:** No code changes required. The supports are declared in `block.json` and serialized automatically by `useBlockProps.save()` because Math is a static block. No PHP render-callback change.
- **No breaking changes.** The legacy `color.gradients` support remains alongside the new `background.gradient`; existing sites with gradient colors on Math blocks are unaffected.
- **No action required** for existing installations. The new controls simply appear in the Background panel.

## Technical details

The diff adds a `background` object to `packages/block-library/src/math/block.json`:

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

The existing `color.gradients` support is left in place alongside `background.gradient`, so both the legacy color-panel gradient and the new Background-panel gradient are available. Because `core/math` is a static block (no dynamic render callback), the supports are serialized onto the wrapper element by `useBlockProps.save()` at save time; no PHP change is needed.

Per #75859, the gradient renders as a layer over the background image, so both are visible together.

The core-blocks reference (`docs/reference-guides/core-blocks/README.md`) and the block's own `packages/block-library/src/math/README.md` are regenerated to list the new `background` supports.

## Contribution

Co-authored by @aaronrobertshaw and @andrewserong. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." The discussion contains only the automated performance-bot report and the co-author credit; no design debate or alternative approaches are recorded. It is part of the broader design-tools consistency work tracked under #43241.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
