# #83258: Post Time to Read: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Time to Read`
- **Merged:** [`81abd6b`](https://github.com/WordPress/gutenberg/commit/81abd6b54ad8644e91ed621ee1bf5916ebe34df6)
- **Discussion:** [#83258](https://github.com/WordPress/gutenberg/pull/83258) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Time to Read block (`core/post-time-to-read`) now supports the Background panel's image, size, and gradient controls in Global Styles and the block inspector. Previously the block supported color (including legacy `color.gradients`), border, shadow, spacing, and typography, but lacked the newer `background.*` supports that other core blocks already had. This closes a gap in the design-tools consistency effort tracked under #43241.

## Impact

- **Theme developers / site builders:** The Time to Read block now accepts `backgroundImage`, `backgroundSize`, and `gradient` in Global Styles (Blocks → Time to Read → Background) and in the block inspector. No code changes required; the supports serialize automatically through `get_block_wrapper_attributes()`.
- **Plugin & theme developers:** No breaking changes. The legacy `color.gradients` support remains alongside the new `background.gradient` support, so existing gradient styling continues to work. No migration needed.
- **No action required** for existing sites; the change is purely additive.

## Technical details

The change is confined to `packages/block-library/src/post-time-to-read/block.json`, where a new `background` key is added to the `supports` object:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true
}
```

Because `core/post-time-to-read` is server-rendered via `get_block_wrapper_attributes()`, the supports are serialized onto the wrapper element's inline styles with no PHP template change. The pre-existing `color.gradients` support is left in place alongside `background.gradient`; per #75859 the gradient renders as a layer over the background image so both are visible simultaneously.

Two documentation files are regenerated to reflect the new supports: `docs/reference-guides/core-blocks/README.md` (the supports line for the block) and `packages/block-library/src/post-time-to-read/README.md` (the per-block reference).

## Contribution

Authored by @aaronrobertshaw with co-authorship from @talldan. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." The discussion is minimal (2 comments, 0 reactions) and the record carries no design debate or rejected alternatives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
