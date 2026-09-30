# #83238: Post Terms: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Post Terms`
- **Merged:** [`15f7c56`](https://github.com/WordPress/gutenberg/commit/15f7c56ed7b52ea123cbd069d68c634cfc99aac9)
- **Discussion:** [#83238](https://github.com/WordPress/gutenberg/pull/83238) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Post Terms block (`core/post-terms`) now supports the Background panel's image, size, and gradient controls in Global Styles and the block inspector. Previously the block supported color (including legacy `color.gradients`), border, shadow, spacing, and typography, but lacked the `background` supports group. This change brings it in line with other core blocks as part of the ongoing design-tools consistency effort (related to #43241).

## Impact

- **Theme & site builders:** The Post Terms block now exposes `backgroundImage`, `backgroundSize`, and `gradient` in the Global Styles → Blocks → Post Terms → Background panel and in the block-level inspector. No code changes required; the controls appear automatically once the updated block library ships.
- **Plugin & theme developers:** No breaking changes. The legacy `color.gradients` support remains alongside the new `background.gradient` support, so existing gradient styling via the color panel continues to work. No PHP changes are needed because the block is server-rendered through `get_block_wrapper_attributes()`.
- **No action required** for existing sites. The change is purely additive; posts without terms still render an empty string (no empty box to paint).

## Technical details

The diff adds a `background` object to the `supports` key in `packages/block-library/src/post-terms/block.json`:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true
}
```

This sits alongside the existing `color` supports (which retains `gradients: true` for the legacy gradient path). Because `core/post-terms` is server-rendered via `get_block_wrapper_attributes()`, the new supports serialize onto the wrapper element's inline styles and CSS custom properties without any PHP template change. The gradient layers over the background image (per the behavior established in #75859), so both render together.

Two documentation files are regenerated to reflect the new supports:
- `docs/reference-guides/core-blocks/README.md` — the Post Terms supports line now reads `background (backgroundImage, backgroundSize, gradient), color (background, gradients, link, text), …`
- `packages/block-library/src/post-terms/README.md` — a new `background` entry with the three sub-keys is inserted before the existing `color` entry.

## Contribution

Opened and merged by @aaronrobertshaw with co-authorship from @ramonjd. The PR body notes the implementation was produced by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions), with no visible design debate or rejected alternatives in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
