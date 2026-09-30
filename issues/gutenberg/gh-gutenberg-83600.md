# #83600: Post Title: Add minimum width and height support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Post Title`
- **Merged:** [`176d860`](https://github.com/WordPress/gutenberg/commit/176d8606d2a727af72bf076f5bbc9fc2816b9429)
- **Discussion:** [#83600](https://github.com/WordPress/gutenberg/pull/83600) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Post Title block (`core/post-title`) now supports `dimensions.minHeight` and `dimensions.minWidth` in its `block.json` `supports`, bringing it in line with Group, Quote, Verse, Pullquote, and Post Content, which already expose minimum-height, and Group, which also exposes minimum-width. This is part of the broader design-tools consistency effort tracked in #43241. No PHP or render-callback changes were needed because the block is server-rendered and its wrapper is produced by `get_block_wrapper_attributes()`, which serializes the support automatically.

## Impact

- **Theme & site builders:** Post Title can now be given a minimum width and minimum height via Global Styles (Blocks → Title → Dimensions) or the block inspector's Dimensions panel. Useful for post-grid layouts where variable title lengths cause ragged card rows.
- **Plugin & theme developers:** No code changes required. The support is declared in `block.json` and handled by the existing dimensions support infrastructure. If you build custom Post Title patterns or templates, the new CSS custom properties (`--wp--style--min-height`, `--wp--style--min-width`) will now apply to the block wrapper.
- **No breaking changes, deprecations, or migrations.** Width, height, and aspect-ratio supports are explicitly out of scope for this PR.

## Technical details

The change is confined to `packages/block-library/src/post-title/block.json`, where a new `dimensions` key is added to the `supports` object:

```json
"dimensions": {
  "minHeight": true,
  "minWidth": true
}
```

This mirrors the existing `dimensions` support on the Group block. Because Post Title is server-rendered, no PHP changes are needed: `get_block_wrapper_attributes()` reads the `supports` declaration and emits the corresponding CSS custom properties on the wrapper element. The render callback `render_block_core_post_title()` was verified to never produce an empty wrapper (it returns an empty string when there is no `postId` in context or when the title is empty), so the box-painting concern that applies to some other blocks does not arise here.

Two documentation files were regenerated to list the new support:
- `docs/reference-guides/core-blocks/README.md` — the Post Title entry's supports line now includes `dimensions (minHeight, minWidth)`.
- `packages/block-library/src/post-title/README.md` — a new `dimensions` section with `minHeight: true` and `minWidth: true` is inserted between the `color` and `shadow` entries.

## Contribution

Opened by @aaronrobertshaw as part of the design-tools consistency work (related #43241). The PR was co-authored with @andrewserong. The author notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions), with no notable design debate or rejected alternatives recorded.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
