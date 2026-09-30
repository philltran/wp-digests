# #83744: Comments Pagination: Add padding, margin, and block gap support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Comments Pagination`
- **Merged:** [`a4e46f4`](https://github.com/WordPress/gutenberg/commit/a4e46f48d5b294fd1163c55a9fa331ffee17dde0)
- **Discussion:** [#83744](https://github.com/WordPress/gutenberg/pull/83744) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Comments Pagination block (`core/comments-pagination`) now supports `spacing.padding`, `spacing.margin`, and `spacing.blockGap` in its `block.json`, allowing per-block and Global Styles control over the padding, margin, and inter-link gap of the pagination row. Previously, the space between the Previous, Numbers, and Next links was governed solely by the theme's global block gap and could not be overridden per block. The change is purely additive: no PHP or render-callback modification is needed because the block is server-rendered through `get_block_wrapper_attributes()`, which serializes the new supports automatically.

## Impact

- **Site owners / theme developers:** Can now set padding, margin, and block gap on Comments Pagination via Global Styles (Blocks → Comments Pagination → Dimensions) or the block inspector, with independent mobile values. No action required on existing sites — the block renders identically when no spacing styles are applied.
- **Plugin & theme developers:** If a theme or plugin was manually styling the gap between `core/comments-pagination-previous`, `core/comments-pagination-numbers`, and `core/comments-pagination-next` via CSS, the new native `blockGap` support will take precedence when set. No API or hook changes.
- **No breaking changes, deprecations, or removed APIs.**

## Technical details

Three files change in the block package:

1. **`packages/block-library/src/comments-pagination/block.json`** — a new `spacing` key is added to `supports`:

```json
"spacing": {
    "padding": true,
    "margin": true,
    "blockGap": true
}
```

2. **`packages/block-library/src/comments-pagination/style.scss`** — `box-sizing: border-box` is added to `.wp-block-comments-pagination` so that the now-customizable padding behaves predictably (matching the Comment Edit Link block).

3. **`docs/reference-guides/core-blocks/README.md`** and **`packages/block-library/src/comments-pagination/README.md`** — regenerated to list `spacing (blockGap, margin, padding)` in the supports table.

No PHP change: the block's render callback builds its `<nav>` wrapper with `get_block_wrapper_attributes()`, which picks up the new supports and emits the corresponding inline styles. The callback already returns an empty string when there is no inner content, so no empty wrapper is produced for padding to paint on.

Margin is deliberately **not** added to the three child blocks (`core/comments-pagination-previous`, `core/comments-pagination-numbers`, `core/comments-pagination-next`). The parent is a flex container, which resets child margins, so a Global Styles margin on a child would have no visual effect. Padding and margin on the parent cover the space inside and around the entire pagination row; `blockGap` controls the space between the three child elements.

## Contribution

The PR supersedes an earlier attempt (#66470) by @rinkalpagdar and is related to the broader spacing-supports tracking issue #43241. It was co-authored by @aaronrobertshaw and @andrewserong, and the PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." The discussion thread contains only the two automated bot comments (bundle-size/performance report and contributor-credit list) with no design debate or alternative approaches recorded.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
