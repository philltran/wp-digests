# #83247: Site Title: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Site Title`
- **Merged:** [`6df1e3c`](https://github.com/WordPress/gutenberg/commit/6df1e3cb909a2662d1e7975120b320708a08d602)
- **Discussion:** [#83247](https://github.com/WordPress/gutenberg/pull/83247) · 2 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Site Title block (`core/site-title`) now supports the Background panel's image, size, and gradient controls in Global Styles and the block inspector. Previously the block supported color (including legacy `color.gradients`), border, shadow, spacing, and typography, but not the newer `background.*` supports. The change is a pure `block.json` supports declaration; because the block is server-rendered through `get_block_wrapper_attributes()`, the new supports serialize onto the wrapper automatically with no PHP change.

## Impact

- **Theme & site builders (block themes):** The Site Title block now accepts `backgroundImage`, `backgroundSize`, and `gradient` in Global Styles (Blocks → Site Title → Background) and in the block inspector. No code changes required; the controls appear automatically once the block is updated.
- **Plugin & theme developers:** No breaking changes. The legacy `color.gradients` support remains alongside the new `background.gradient` support, so existing gradient styling on the Site Title block is unaffected. If you were manually applying background styles to `.wp-block-site-title` via CSS, be aware the block can now emit `background-image`, `background-size`, and gradient CSS custom properties from the editor.
- **No action required** for existing sites unless you want to use the new controls.

## Technical details

The sole functional change is in `packages/block-library/src/site-title/block.json`, which adds a `background` object to the existing `supports` key:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true
}
```

This sits alongside the pre-existing `color` supports (`gradients`, `link`, `text`, `background`). Because `core/site-title` is server-rendered via `get_block_wrapper_attributes()`, the block-supports serialization pipeline picks up the new keys and emits the corresponding CSS custom properties and inline styles on the wrapper element—no PHP template or render-callback change is needed.

The gradient layers over the background image (per the behavior established in #75859), so both render simultaneously. When the site title is empty the block returns an empty string, so no empty box is painted.

Two documentation files were regenerated to reflect the new supports: `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/site-title/README.md`.

## Contribution

Opened by @aaronrobertshaw with @ramonjd as co-author. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." Discussion was minimal (2 comments, 0 reactions), with no recorded design debate or rejected alternatives. It references the broader design-tools consistency effort tracked in #43241.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
