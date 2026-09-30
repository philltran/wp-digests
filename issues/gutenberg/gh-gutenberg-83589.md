# #83589: Site Tagline: Add minimum width and height support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Site Tagline`
- **Merged:** [`c2a1f9f`](https://github.com/WordPress/gutenberg/commit/c2a1f9f788a233936e7c1c26ebeb351d32a65068)
- **Discussion:** [#83589](https://github.com/WordPress/gutenberg/pull/83589) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Site Tagline block (`core/site-tagline`) now supports `dimensions.minHeight` and `dimensions.minWidth`, bringing it in line with other core blocks that already expose minimum-size controls. The change is a `block.json` addition only; because the block is server-rendered through `get_block_wrapper_attributes()`, the new CSS custom properties are serialized automatically with no PHP modification. This is part of the broader design-tools consistency effort tracked in #43241.

## Impact

- **Block theme developers / site builders:** The Site Tagline block now accepts minimum width and minimum height via Global Styles (desktop and mobile) and the block inspector. No code changes required; the controls appear in the existing Dimensions panel.
- **Plugin & theme developers:** No API change. If you build on top of `core/site-tagline` and read its `block.json` supports, the new `dimensions` key will be present. No migration or configuration needed.
- **No action required** for existing sites. The block renders identically when no minimum dimensions are set.

## Technical details

The sole functional change is in `packages/block-library/src/site-tagline/block.json`, which gains:

```json
"dimensions": {
  "minHeight": true,
  "minWidth": true
}
```

Because `core/site-tagline` is a server-rendered block whose wrapper is produced by `get_block_wrapper_attributes()`, the block-supports serialization pipeline emits the corresponding `--wp--min-height` and `--wp--min-width` CSS custom properties on the wrapper element without any change to the render callback. The render callback already returns early when `get_bloginfo( 'description' )` is empty, so no guard against an empty box is needed.

Two documentation files are regenerated to reflect the new support:
- `docs/reference-guides/core-blocks/README.md` — the Site Tagline supports line now lists `dimensions (minHeight, minWidth)`.
- `packages/block-library/src/site-tagline/README.md` — a new `dimensions` entry with `minHeight: true` and `minWidth: true` is added under the supports list.

## Contribution

Opened by @aaronrobertshaw and co-authored with @ramonjd. The PR body notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no design debate or rejected alternatives recorded.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
