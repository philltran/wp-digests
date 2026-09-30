# #83233: No Results: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`93ca030`](https://github.com/WordPress/gutenberg/commit/93ca0309743c24376719276273d50a50da8b7db3)
- **Discussion:** [#83233](https://github.com/WordPress/gutenberg/pull/83233) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The No Results block (`core/query-no-results`) now supports background image, background size, and gradient via the `background` block support, bringing it in line with other blocks that already expose these controls in the Background panel. Previously the block only supported color (including legacy `color.gradients`), border, shadow, spacing, and typography. Because the block is server-rendered through `get_block_wrapper_attributes()`, the new supports serialize onto the wrapper automatically with no PHP change required.

## Impact

- **Theme developers / Global Styles authors:** The No Results block now appears in the Background panel of Global Styles with image, size, and gradient controls. You can set a background image with `background-size` (e.g. `contain`, `cover`) and a gradient that layers over the image. No code changes needed — the supports are declarative in `block.json`.
- **Plugin & theme developers:** No breaking changes. The legacy `color.gradients` support remains alongside the new `background.gradient`; both are active simultaneously. No migration or code update required.
- **Site owners / editors:** No action required. The new controls are opt-in via Global Styles or the block inspector; existing No Results blocks render unchanged until a background is explicitly set.
- **Headless / REST consumers:** No REST schema or route changes. The block's serialized output gains `background-image`, `background-size`, and gradient CSS custom properties on the wrapper when set, but the REST block schema is unchanged.

## Technical details

The change is entirely in `packages/block-library/src/query-no-results/block.json`, adding a `background` supports object:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true
}
```

This sits alongside the existing `color` supports (which still include `gradients: true` for the legacy gradient path). Because `core/query-no-results` is server-rendered via `get_block_wrapper_attributes()`, the block-supports serialization pipeline emits the corresponding CSS custom properties (`--wp--custom--background-image`, `--wp--custom--background-size`, and the gradient variables) on the wrapper element without any PHP template change. The gradient layers over the background image per the behavior established in #75859.

Two documentation files are regenerated to reflect the new supports:
- `docs/reference-guides/core-blocks/README.md` — the supports line for `core/query-no-results` now lists `background (backgroundImage, backgroundSize, gradient)`.
- `packages/block-library/src/query-no-results/README.md` — a new `background` entry with the three sub-keys is added to the supports list.

No new hooks, filters, REST schema fields, or database changes are introduced.

## Contribution

Authored by @aaronrobertshaw with co-authorship from @andrewserong. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." The discussion is minimal (2 comments, 0 reactions) with no visible design debate or rejected alternatives; the change follows the established pattern from the broader design-tools consistency effort tracked in #43241.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
