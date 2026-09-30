# #83246: Site Tagline: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Site Tagline`
- **Merged:** [`bc66749`](https://github.com/WordPress/gutenberg/commit/bc667490d371bd7c26d61dcf188e68d0bbe3dc4d)
- **Discussion:** [#83246](https://github.com/WordPress/gutenberg/pull/83246) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Site Tagline block (`core/site-tagline`) now supports background image, background size, and gradient via the `background` supports key in its `block.json`. This closes a gap where the block already supported color, border, shadow, spacing, and typography but not the Background panel's image/size/gradient controls, as part of the broader design-tools consistency effort (related to #43241). Because the block is server-rendered through `get_block_wrapper_attributes()`, the new supports serialize onto the wrapper element automatically with no PHP change.

## Impact

- **Site owners / block-theme users:** Can now set a background image, size (e.g. Contain, Cover), and gradient on the Site Tagline block from Global Styles (Blocks → Site Tagline → Background) or the block inspector, with responsive (mobile) overrides. No action required.
- **Theme developers:** The site-tagline wrapper will now emit `background-image`, `background-size`, and gradient-related CSS custom properties when those supports are set. If a theme applies its own background styling to `.wp-block-site-tagline`, it may need to account for the new properties. No code changes are mandatory.
- **Plugin developers:** No API change. The legacy `color.gradients` support remains alongside the new `background.gradient`; both can be active simultaneously. No migration or code update needed.
- **Headless / REST consumers:** No schema or route changes. The block's serialized HTML will include the new inline styles when set.

## Technical details

The diff adds a single `background` object to `packages/block-library/src/site-tagline/block.json`:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true
}
```

This sits alongside the existing `color.gradients: true` support, which is retained for backward compatibility. The block's `render.php` calls `get_block_wrapper_attributes()`, so the Block Editor's supports serialization pipeline outputs the corresponding CSS custom properties and inline styles on the wrapper `<p>` element with no PHP modification.

Per the PR description, the gradient layers over the background image (consistent with the behavior established in #75859), so both are visible simultaneously. The block returns an empty string when no tagline is set, so no empty styled box is rendered.

Two documentation files are regenerated to list the new supports: `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/site-tagline/README.md`.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @ramonjd. The PR carries only 2 comments and 0 reactions, indicating a straightforward, uncontested merge. The author notes the implementation was produced by a Claude Code agent from a predefined task. No alternative approaches or design debates are visible in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
