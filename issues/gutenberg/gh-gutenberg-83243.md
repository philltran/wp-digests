# #83243: Query Total: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Query Total`
- **Merged:** [`99a443f`](https://github.com/WordPress/gutenberg/commit/99a443f0b0a2ca407dde18a29da062416f39d5be)
- **Discussion:** [#83243](https://github.com/WordPress/gutenberg/pull/83243) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Query Total block (`core/query-total`) now supports the Background panel's image, size, and gradient controls in the block editor and Global Styles. Previously the block supported color (including legacy `color.gradients`), border, shadow, spacing, and typography, but lacked the `background` supports group. This change adds `background.backgroundImage`, `background.backgroundSize`, and `background.gradient` to the block's `block.json`, bringing it in line with other core blocks as part of the design-tools consistency effort (related to #43241).

## Impact

- **Site owners / theme authors:** The Query Total block inside a Query Loop can now be styled with a background image, a background size (e.g. `cover`, `contain`), and a gradient overlay via Global Styles or the block inspector. No code changes required.
- **Plugin & theme developers:** No API or hook changes. The block is server-rendered through `get_block_wrapper_attributes()`, so the new supports serialize onto the wrapper automatically. No PHP template or render-callback update is needed.
- **No action required** for existing sites. The change is purely additive; blocks without these styles set render identically to before.

## Technical details

The sole functional change is in `packages/block-library/src/query-total/block.json`, which gains a `background` supports object:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true
}
```

The pre-existing `color.gradients` support (the legacy gradient mechanism) is retained alongside the new `background.gradient` key. Because `core/query-total` is server-rendered via `get_block_wrapper_attributes()`, the supports are serialized into the wrapper's inline styles and CSS custom properties without any PHP-side change.

Per the PR description, the gradient layers over the background image (consistent with the behavior established in #75859), so both are visible simultaneously.

Two documentation files are regenerated to reflect the new supports:
- `docs/reference-guides/core-blocks/README.md` — the Query Total entry's supports list now includes `background (backgroundImage, backgroundSize, gradient)`.
- `packages/block-library/src/query-total/README.md` — a new `background` section lists the three sub-keys as `true`.

## Contribution

Opened and merged by @aaronrobertshaw with co-authorship from @ramonjd. The PR notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no visible design debate or rejected alternatives; it is a straightforward application of an established supports pattern to a block that was missing it.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
