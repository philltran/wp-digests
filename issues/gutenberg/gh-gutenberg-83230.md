# #83230: Home Link: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Home Link`
- **Merged:** [`22e529f`](https://github.com/WordPress/gutenberg/commit/22e529f5e60d98c5d7536b447b9bdf7c36c132a8)
- **Discussion:** [#83230](https://github.com/WordPress/gutenberg/pull/83230) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Home Link block (`core/home-link`) now supports the Background panel's image, size, and gradient controls in the block editor and Global Styles. Previously it only supported shadow and typography, making it inconsistent with other blocks in the design tools suite. The change is a `block.json` supports addition; because the block is server-rendered via `get_block_wrapper_attributes()`, the new supports serialize onto the menu item automatically with no PHP change.

## Impact

- **Theme & site builders:** The Home Link block inside a Navigation block now exposes Background → Image, Size, and Gradient controls in both the block inspector and Global Styles (including responsive/mobile states). No code changes required.
- **Plugin & theme developers:** No new hooks, filters, or REST schema changes. If you build custom Navigation or Home Link rendering, the serialized attributes on the menu item will now include `background-image`, `background-size`, and gradient CSS custom properties when set. No migration needed.
- **No action required** for existing sites; the change is purely additive and only takes effect when a user explicitly sets a background on the block.

## Technical details

The sole functional change is in `packages/block-library/src/home-link/block.json`, adding a `background` object under `supports`:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true
}
```

This enables the existing `@wordpress/block-library` background support infrastructure (image upload, `background-size` control, and gradient overlay) for the Home Link block. Because the block renders server-side through `get_block_wrapper_attributes()`, the supports are serialized as inline styles / CSS custom properties on the `<a>` element without any template or PHP modification.

The gradient layers over the background image (per the pattern established in #75859), so both are visible simultaneously.

Two documentation files are regenerated to reflect the new supports:
- `docs/reference-guides/core-blocks/README.md` — the Home Link supports line now reads `background (backgroundImage, backgroundSize, gradient)`.
- `packages/block-library/src/home-link/README.md` — a new `background` entry with the three sub-keys is added to the supports list.

## Contribution

Opened and merged by @aaronrobertshaw with co-authorship from @andrewserong. The PR body notes the implementation was produced by a Claude Code agent from a predefined task. The only substantive discussion was @aaronrobertshaw thanking a reviewer for overlay-menu testing. No design debate or rejected alternatives are recorded in the three comments.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
