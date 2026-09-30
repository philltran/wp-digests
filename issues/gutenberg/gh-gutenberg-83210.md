# #83210: Comment Edit Link: Add background gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Comment Edit Link`
- **Merged:** [`c74076f`](https://github.com/WordPress/gutenberg/commit/c74076f5ac7b7cd8e926d5a91745a105d439ba52)
- **Discussion:** [#83210](https://github.com/WordPress/gutenberg/pull/83210) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `core/comment-edit-link` block now supports the Background panel's gradient controls (`background.gradient` in `block.json`), bringing it in line with other blocks that already expose this support. Previously the block only had the legacy `color.gradients` support, so the newer Background panel gradient picker was unavailable. Because the block is server-rendered through `get_block_wrapper_attributes()`, the new support serializes onto the wrapper automatically with no PHP change required.

## Impact

- **Theme & site builders:** The Comment Edit Link block inside the Comment Template can now receive a background gradient via Global Styles (Blocks → Comment Edit Link → Background) or a per-instance override in the block inspector, including responsive (mobile/tablet) variants. No code changes needed.
- **Plugin & theme developers:** No action required. The block's `block.json` gains a new `supports` key; any code that reads or merges supports for this block will now see `background.gradient: true` alongside the existing `color.gradients: true`.
- **No breaking changes.** The legacy `color.gradients` support is retained; both coexist.

## Technical details

The functional change is a single addition to `packages/block-library/src/comment-edit-link/block.json`:

```json
"supports": {
  "anchor": true,
  "html": false,
  "background": {
    "gradient": true
  },
  "color": {
    "link": true,
    "gradients": true,
    ...
  }
}
```

Because `core/comment-edit-link` is server-rendered via `get_block_wrapper_attributes()`, the `background.gradient` support is serialized onto the wrapper element's inline styles by the core block-supports machinery—no `render.php` or PHP template change is needed. The block's render callback returns an empty string when there is no comment or the current user lacks edit permission, so no empty gradient box is painted in those cases.

Two documentation files are regenerated to reflect the new support:
- `docs/reference-guides/core-blocks/README.md` — the supports line for `core/comment-edit-link` now reads `background (gradient), color (background, gradients, link, ~~text~~), …`.
- `packages/block-library/src/comment-edit-link/README.md` — a new `background.gradient: true` entry is listed under the supports section.

## Contribution

Opened by @aaronrobertshaw as part of the broader design-tools consistency effort tracked in #43241. The PR was implemented, built, and screenshotted by a Claude Code agent from a predefined task, with @shail-mehta listed as a co-author. The discussion is minimal (2 comments, 0 reactions) with no notable design debate or rejected alternatives in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
