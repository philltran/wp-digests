# #83228: Latest Posts: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Latest Posts`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`4b79d74`](https://github.com/WordPress/gutenberg/commit/4b79d74dfb8dd20bf4358d2e235c86bccbf49b16)
- **Discussion:** [#83228](https://github.com/WordPress/gutenberg/pull/83228) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Latest Posts block (`core/latest-posts`) now supports background image, background size, and gradient via the Background panel in Global Styles and the block inspector. This closes a gap where the block already supported color, border, layout, shadow, spacing, and typography but not the Background panel's image/size/gradient controls, as part of the broader design-tools consistency effort tracked in #43241. No PHP changes were required because the block is server-rendered through `get_block_wrapper_attributes()`, which serializes the new supports onto the list wrapper automatically.

## Impact

- **Site owners / content editors:** Can now set a background image (with `cover`, `contain`, etc. sizing) and a gradient on the Latest Posts block from Global Styles or the block inspector, with responsive (mobile) overrides. No code changes needed.
- **Theme & plugin developers:** No action required. The new supports are additive and serialize through the existing `get_block_wrapper_attributes()` path. If you were applying custom CSS to the Latest Posts wrapper, the new `background-image`, `background-size`, and gradient properties will now appear alongside your rules.
- **No breaking changes.** The legacy `color.gradients` support is retained alongside the new `background.gradient`; both coexist in `block.json`.

## Technical details

The change is confined to `packages/block-library/src/latest-posts/block.json` and two regenerated README files. The `supports` object gains:

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

The existing `color.gradients: true` entry is left in place alongside the new `background.gradient`, so both the legacy and new gradient paths remain active.

Because Latest Posts is server-rendered via `get_block_wrapper_attributes()`, the supports are serialized onto the `<ul>` wrapper as inline styles / CSS custom properties with no PHP-side change. When zero posts match, the block renders an empty `<ul>`; the background paints that empty box, consistent with how the block's existing color, border, padding, and shadow supports already behave (the same approach taken in the shadow adoption PR #83051).

The gradient layers over the background image per the behavior established in #75859, so both are visible simultaneously.

`docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/latest-posts/README.md` are regenerated to list the new `background` supports.

## Contribution

The PR was implemented, built, and screenshotted by a Claude Code agent from a predefined task, per the author's disclosure. It was co-authored by @aaronrobertshaw and @talldan. The discussion contains only two automated bot comments (performance metrics and flaky-test report) with no visible design debate or alternative approaches considered.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
