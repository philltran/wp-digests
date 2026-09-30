# #83229: Login/out: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Login/out`
- **Merged:** [`59e5fc9`](https://github.com/WordPress/gutenberg/commit/59e5fc9e5ed916ce998d94ef212de7f2ac66942a)
- **Discussion:** [#83229](https://github.com/WordPress/gutenberg/pull/83229) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Login/out block (`core/loginout`) now supports background image, background size, and gradient via the Background panel in Global Styles and the block inspector. This is a `block.json` supports addition only — because the block is server-rendered through `get_block_wrapper_attributes()`, the new supports serialize onto the wrapper automatically with no PHP or JS rendering changes. The change brings the block in line with other core blocks as part of the ongoing design-tools consistency work (related to #43241).

## Impact

- **Site owners / theme developers:** Can now set a background image (with `contain`, `cover`, etc. sizing) and a gradient on the Login/out block from Global Styles → Blocks → Login/out → Background, or per-instance from the block inspector. Gradient layers over the image (per #75859), so both render together.
- **Plugin & theme developers:** No code changes required. The legacy `color.gradients` support remains alongside the new `background.gradient`, so existing gradient styling via the Color panel is unaffected.
- **No action required** for anyone not using the Login/out block or not setting background styles on it.

## Technical details

The sole functional change is in `packages/block-library/src/loginout/block.json`, which gains a `background` supports object:

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

The `__experimentalDefaultControls` key enables the image and gradient controls in the block inspector by default (not just via Global Styles). No PHP template or render-callback changes are needed: the block is server-rendered through `get_block_wrapper_attributes()`, which serializes all registered supports onto the wrapper element's inline styles.

The legacy `color.gradients` support (under the existing `color` key) is left in place alongside the new `background.gradient`, so both paths continue to work. The gradient renders as a layer above the background image, consistent with the behavior established in #75859.

Two documentation files are regenerated to reflect the new supports: `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/loginout/README.md`.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @ramonjd. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." The discussion is minimal (2 comments, 0 reactions) with no visible design debate or rejected alternatives; the approach of adding supports to `block.json` and relying on `get_block_wrapper_attributes()` for serialization is the standard pattern for this kind of change.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
