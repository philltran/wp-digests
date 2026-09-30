# #83218: Post Date: Add background gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Post Date`
- **Merged:** [`09b5d40`](https://github.com/WordPress/gutenberg/commit/09b5d40c5d415b1133554155bb0cfc0d639a056a)
- **Discussion:** [#83218](https://github.com/WordPress/gutenberg/pull/83218) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Post Date block (`core/post-date`) now supports the Background panel's gradient controls via the `background.gradient` block support. This closes a gap where the block already supported color (including legacy `color.gradients`), border, spacing, and typography, but not the newer Background panel gradient. The change is purely declarative in `block.json`; no PHP modification is needed because the block is server-rendered through `get_block_wrapper_attributes()`, which serializes the support onto the wrapper automatically.

## Impact

- **Site owners / theme builders:** The Date block now accepts background gradients from Global Styles (Blocks → Date → Background) and from the block-level inspector, including responsive (mobile/tablet/desktop) variants. No code changes required.
- **Plugin & theme developers:** No action required. The legacy `color.gradients` support is retained alongside the new `background.gradient`, so existing themes that target the older key continue to work. If you register custom styles for `core/post-date` via `wp_enqueue_block_style` or Global Styles overrides, the new `background.gradient` key is now available.
- **Headless / REST consumers:** No schema or route changes. The support is a rendering concern handled server-side.

## Technical details

The diff adds a single key to the `supports` object in `packages/block-library/src/post-date/block.json`:

```json
"background": {
  "gradient": true
}
```

This sits alongside the existing `color.gradients: true` entry, so both the legacy and new gradient mechanisms are active simultaneously. Because `core/post-date` is server-rendered via `get_block_wrapper_attributes()`, the support is serialized onto the wrapper element's `style` attribute without any PHP change. When no date is present the block returns an empty string, so no empty box is painted.

Two documentation files are regenerated to reflect the new support:
- `docs/reference-guides/core-blocks/README.md` — the supports line for `core/post-date` now reads `background (gradient), color (background, gradients, link, text), …`
- `packages/block-library/src/post-date/README.md` — a new `background.gradient: true` entry is listed under the supports section.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @ramonjd. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task," part of the broader design-tools consistency effort tracked in #43241. The record shows only 2 comments and no reactions; no design debate or alternative approaches are visible in the discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
