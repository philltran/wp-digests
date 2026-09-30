# #83584: No Results: Add minimum width and height support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] No Results`
- **Merged:** [`ef9e9d6`](https://github.com/WordPress/gutenberg/commit/ef9e9d600ad6032f4eb4f0a73f72c1d28ba7c444)
- **Discussion:** [#83584](https://github.com/WordPress/gutenberg/pull/83584) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The No Results block (`core/query-no-results`) now supports `dimensions.minWidth` and `dimensions.minHeight`, bringing it in line with other core blocks that already expose minimum width and height controls. The change is a `block.json` addition only; because the block is server-rendered through `get_block_wrapper_attributes()`, the new support serializes into the wrapper's inline styles automatically with no PHP modification. The render callback still returns an empty string when the block has no inner content, so the minimum dimensions never produce a visible box around nothing.

## Impact

- **Site owners / theme developers:** Minimum width and height for the No Results block are now available in Global Styles (Blocks → No Results → Dimensions) and in the block inspector, with independent mobile-state values. No code changes required.
- **Plugin & theme developers:** No action required. The support is declared in `block.json` and handled by the existing block-supports pipeline. If you build custom Query Loop templates that embed a No Results block, the new controls simply appear in the editor.
- **Headless / REST consumers:** No schema or route changes. The rendered HTML gains `min-width` / `min-height` inline styles on the wrapper when the values are set.

## Technical details

The diff touches three files, all declarative:

1. **`packages/block-library/src/query-no-results/block.json`** — adds the support entry:

```json
"dimensions": {
    "minHeight": true,
    "minWidth": true
}
```

2. **`packages/block-library/src/query-no-results/README.md`** — regenerated to list the new `dimensions` support with `minHeight: true` and `minWidth: true`.

3. **`docs/reference-guides/core-blocks/README.md`** — the core-blocks reference table for `core/query-no-results` now includes `dimensions (minHeight, minWidth)` in its supports column.

No changes to `render.php`, `index.js`, or any PHP file. The block's existing render callback calls `get_block_wrapper_attributes()`, which reads the `dimensions` support from the block's registered supports and emits the corresponding CSS custom properties / inline styles. The callback's early-return (`return '';`) when inner content is empty means the minimum dimensions are inert in that case.

## Contribution

Opened by @aaronrobertshaw as part of the broader design-tools consistency effort tracked in #43241. The PR was implemented, built, and screenshotted by a Claude Code agent from a predefined task, per the author's disclosure. @ramonjd is listed as a co-author on the merge commit. The discussion is minimal (2 comments, 0 reactions) with no recorded design debate or rejected alternatives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
