# #83424: Post Author Biography: Add text columns support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`c9c8fd6`](https://github.com/WordPress/gutenberg/commit/c9c8fd6830dec2ae8b458ee896e784420dae0442)
- **Discussion:** [#83424](https://github.com/WordPress/gutenberg/pull/83424) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Post Author Biography block (`core/post-author-biography`) now supports the `typography.textColumns` block support, allowing the author bio to be rendered in multiple CSS columns. This brings the block in line with Paragraph and Post Excerpt, which already expose the same support, and enables column count to be set from Global Styles or the block inspector's Typography panel.

## Impact

- **Theme & site builders:** The Author Biography block now accepts a column count via Global Styles (Blocks → Author Biography → Typography → Columns) or the block inspector. No code changes required; the support is opt-in through the existing Typography panel (enable "Columns" from the panel's options menu).
- **Plugin & theme developers:** No API change. The block is server-rendered and its render callback passes through `get_block_wrapper_attributes()`, which serializes the `textColumns` support automatically. No PHP or JS changes are needed in themes or plugins.
- **No action required** for existing sites. The support defaults to off (single column) until explicitly set.

## Technical details

The change is a single-line addition to `packages/block-library/src/post-author-biography/block.json`, adding `"textColumns": true` to the `supports.typography` object alongside the existing `fontSize`, `lineHeight`, and `textAlign` entries:

```json
"typography": {
  "fontSize": true,
  "lineHeight": true,
  "textAlign": true,
  "textColumns": true,
  "__experimentalFontFamily": true,
  "__experimentalFontWeight": true,
  "__experimentalFontStyle": true
}
```

Because the block is server-rendered and its render callback calls `get_block_wrapper_attributes()`, the `textColumns` support is serialized into the block wrapper's inline styles automatically — no PHP render-callback change is needed. The two README files (`docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/post-author-biography/README.md`) are regenerated to list the new support. This mirrors the identical support already present on Paragraph (added in #74656) and Post Excerpt (added in #75587).

## Contribution

Opened by @aaronrobertshaw as part of the design-tools consistency work tracked under #43241. The PR was implemented, built, and screenshotted by a Claude Code agent from a predefined task. @shail-mehta and @ramonjd are listed as co-authors. @ramonjd reviewed and merged the PR with a brief "LGTM" comment; no design debate or alternative approaches are recorded in the three comments.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
