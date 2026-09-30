# #83259: Tag Cloud: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Tag Cloud`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`41db67a`](https://github.com/WordPress/gutenberg/commit/41db67a21f1eb77bf092d93c80ceb8b1ad82c82b)
- **Discussion:** [#83259](https://github.com/WordPress/gutenberg/pull/83259) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Tag Cloud block (`core/tag-cloud`) now supports the Background panel's image, size, and gradient controls in the block editor and Global Styles. Previously the block supported color (including legacy `color.gradients`), border, shadow, spacing, and typography, but not the newer `background.*` supports. This closes a gap in the design-tools consistency effort tracked in #43241. Because the block is server-rendered via `get_block_wrapper_attributes()`, the new supports serialize onto the wrapper `<p>` with no PHP change required.

## Impact

- **Theme & site builders:** The Tag Cloud block now exposes Background → Image, Size, and Gradient controls in both the block inspector and Global Styles (Blocks → Tag Cloud → Background). No code changes needed; the controls appear automatically once the updated block is loaded.
- **Plugin & theme developers:** No breaking changes. The legacy `color.gradients` support remains alongside the new `background.gradient` support, so existing themes that style tag-cloud gradients via the older path continue to work. No migration required.
- **Hosting & platform:** No action required. The change is confined to `block.json` supports and regenerated documentation.
- **Headless & REST consumers:** No new REST routes or schema changes. The serialized wrapper attributes will include the new CSS custom properties when set, but the block's REST representation is unchanged.

## Technical details

The sole functional change is in `packages/block-library/src/tag-cloud/block.json`, which adds a `background` key under `supports`:

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

The `__experimentalDefaultControls` sub-object enables the image and gradient controls by default in the block inspector (the size control is available but not shown by default). The existing `color` supports object (which includes `background`, `link`, and the legacy `gradients`) is untouched.

Because `core/tag-cloud` renders server-side through `get_block_wrapper_attributes()`, the block-supports serialization pipeline emits the corresponding CSS custom properties (`--wp--custom--background-image`, `--wp--custom--background-size`, `--wp--custom--gradient`) on the wrapper `<p>` element. No PHP template or render-callback change is needed. The block returns an empty string when no tags exist, so no empty background box is painted.

Per the PR description, the gradient layers over the background image (consistent with #75859), so both are visible simultaneously. The background fills the entire cloud container rather than individual tag links.

Two documentation files are regenerated to list the new supports: `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/tag-cloud/README.md`.

## Contribution

The PR was opened by @aaronrobertshaw and noted as implemented by a Claude Code agent from a predefined task. During review, @talldan flagged an older, stalled PR (#63592) that attempted to add color supports to Tag Cloud but was blocked by a double-classname bug; that bug was later fixed in #74228, making the current change straightforward. @ramonjd raised a follow-up idea about merging inherited Global Styles background values with block-level values (e.g., a block-level gradient change preserving an inherited background image), which was deferred to the broader interoperability issue #76525. The PR merged without further revision.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
