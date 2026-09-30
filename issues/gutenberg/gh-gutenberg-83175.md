# #83175: Post Terms: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Post Terms`
- **Merged:** [`5a49f7f`](https://github.com/WordPress/gutenberg/commit/5a49f7fc4bd9c749cc8aaa79c5e27d982d855e65)
- **Discussion:** [#83175](https://github.com/WordPress/gutenberg/pull/83175) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Post Terms block (rendered as Categories or Tags) now supports the `shadow` block support, allowing shadow styling via Global Styles and per-block overrides. This brings the block in line with other core blocks that already expose shadow as part of the design-tools consistency effort. No PHP changes were required because the block is server-rendered through `get_block_wrapper_attributes()`, which serializes the shadow CSS custom properties onto the wrapper automatically.

## Impact

- **Theme & site developers:** The Post Terms block now accepts shadow presets (Natural, Deep, Sharp, Crisp, etc.) in Global Styles and in the block inspector. No code changes needed; the support is opt-in via the existing shadow UI.
- **Plugin developers:** No new hooks, filters, or API surface. If you build on top of `core/post-terms` and inspect its `block.json` supports, `shadow` will now appear.
- **No action required** for existing sites. The change is purely additive; posts without a shadow applied render identically to before.

## Technical details

The diff adds a single key to the `supports` object in `packages/block-library/src/post-terms/block.json`:

```json
"shadow": true,
```

Because the Post Terms block is server-rendered, the shadow CSS custom properties (`--wp--preset--shadow--*`) are emitted onto the wrapper element by `get_block_wrapper_attributes()`. No render-callback or PHP template change is needed.

One behavioral note: when a post has no terms of the requested type, the render callback returns an empty string rather than an empty wrapper element, so no shadow box is painted in that case.

The two README files (`docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/post-terms/README.md`) are regenerated to list `shadow` in the block's supports.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @im3dabasia. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. It is part of the broader design-tools consistency work tracked in #43241. The discussion is minimal (2 comments, 0 reactions) with no notable design debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
