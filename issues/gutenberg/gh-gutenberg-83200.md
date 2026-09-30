# #83200: Post Author Name: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`9a48e8b`](https://github.com/WordPress/gutenberg/commit/9a48e8bc9d772d9d2519cc0f553092c1f97a80a9)
- **Discussion:** [#83200](https://github.com/WordPress/gutenberg/pull/83200) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Post Author Name block (`core/post-author-name`) now supports background image, background size, and gradient via the `background` supports key in its `block.json`. This closes a gap where the block already supported color, border, spacing, and typography but not the Background panel's image, size, and gradient controls, as part of the broader design-tools consistency effort (related to #43241). Because the block is server-rendered through `get_block_wrapper_attributes()`, the new supports serialize onto the wrapper element with no PHP change required.

## Impact

- **Theme developers / Global Styles users:** The Author Name block now accepts `backgroundImage`, `backgroundSize`, and `gradient` in both Global Styles (Blocks → Author Name → Background) and per-block inspector settings, including responsive (mobile) variants. The gradient layers over the image so both render together.
- **No breaking changes.** The legacy `color.gradients` support is retained alongside the new `background.gradient`; existing sites and themes are unaffected.
- **No action required** for existing installations. The change is purely additive to the block's supported design controls.

## Technical details

The diff adds a `background` object to the `supports` key in `packages/block-library/src/post-author-name/block.json`:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true
}
```

The block is server-rendered via `get_block_wrapper_attributes()`, so these supports are serialized onto the wrapper `<span>` automatically — no render-callback or PHP change is needed. When no author exists the block returns an empty string, so no empty box is painted with a background.

The pre-existing `color.gradients` support (legacy gradient) is kept in place alongside the new `background.gradient`; both coexist. Per #75859, the gradient renders as a layer over the background image rather than replacing it.

Two documentation files are regenerated to list the new supports: `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/post-author-name/README.md`.

## Contribution

Opened by @aaronrobertshaw with @andrewserong as co-author. The PR carries only 2 comments and 0 reactions, with no visible design debate or alternative approaches discussed. The author notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. Merged at `9a48e8b`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
