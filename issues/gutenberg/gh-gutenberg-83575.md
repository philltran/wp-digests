# #83575: Post Content: Add minimum width support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Post Content`
- **Merged:** [`91dc626`](https://github.com/WordPress/gutenberg/commit/91dc62670823d9ef784c49d2105e80e2bb18b0b7)
- **Discussion:** [#83575](https://github.com/WordPress/gutenberg/pull/83575) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `core/post-content` block now supports `dimensions.minWidth`, allowing a minimum width to be set via Global Styles or the block inspector. This closes a gap where the block already supported `minHeight` but not `minWidth`, as part of the broader design-tools consistency effort (related to #43241). Because the block is server-rendered and its wrapper is produced by `get_block_wrapper_attributes()`, the new support is serialized automatically with no PHP-side change.

## Impact

- **Theme developers / site builders:** The Post Content block now exposes a Minimum width control under Dimensions in both Global Styles (with mobile/desktop states) and the block inspector. No code changes required; the control appears automatically once the block is in a template.
- **Plugin & theme developers:** No API change. If you were manually applying `min-width` styles to the post-content wrapper, you can now rely on the block support instead. No deprecations or breaking changes.
- **No action required** for existing sites; the support is opt-in and paints nothing until a value is set.

## Technical details

The diff is confined to three files:

1. **`packages/block-library/src/post-content/block.json`** — the `supports.dimensions` object gains a second key:

```json
"dimensions": {
    "minHeight": true,
    "minWidth": true
}
```

2. **`packages/block-library/src/post-content/README.md`** — the generated supports list adds `minWidth: true` under `dimensions`.

3. **`docs/reference-guides/core-blocks/README.md`** — the core-blocks reference is regenerated to list `minWidth` alongside `minHeight` in the Post Content supports line.

No render-callback, PHP, or CSS changes are included. The block's existing render path calls `get_block_wrapper_attributes()`, which reads the `supports` declaration and emits the corresponding inline style (or class) for `min-width` when a value is present. The render callback returns an empty string when the post has no content, so the support can never produce a visible empty wrapper. Block-level values (set in the inspector) take precedence over Global Styles values at every viewport, consistent with the standard block-supports cascade.

## Contribution

Opened and merged by @aaronrobertshaw with co-author credit to @ramonjd. The PR body notes the implementation was produced by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no design debate or rejected alternatives recorded.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
