# #83240: Previous Page: Add background gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`2ac769c`](https://github.com/WordPress/gutenberg/commit/2ac769c2c6311085d1aa1e3e5ef133bfa6832f7e)
- **Discussion:** [#83240](https://github.com/WordPress/gutenberg/pull/83240) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Previous Page block (`core/query-pagination-previous`) now supports the Background panel's gradient controls in Global Styles and the block inspector. Previously it only had the legacy `color.gradients` support, meaning the newer `background.gradient` pipeline (with responsive states and the Background panel UI) was unavailable. This closes a gap in the design-tools consistency effort tracked in #43241.

## Impact

- **Theme developers / Global Styles authors:** The Previous Page block now appears under Background → Gradient in the Global Styles editor, alongside other blocks that already supported it. No code changes required; the support is declared in `block.json` and serialized automatically.
- **Plugin developers:** No new hooks, filters, or REST schema changes. The block is server-rendered via `get_block_wrapper_attributes()`, so the gradient CSS custom properties are emitted on the link element with no additional PHP.
- **Site owners:** No action required. Existing sites are unaffected; the new control is opt-in through Global Styles or the block inspector.
- **No breaking changes.** The legacy `color.gradients` support remains alongside the new `background.gradient` entry.

## Technical details

The functional change is a single addition to `packages/block-library/src/query-pagination-previous/block.json`:

```json
"background": {
  "gradient": true
}
```

This sits alongside the existing `color.gradients: true` entry. Because `core/query-pagination-previous` is server-rendered through `get_block_wrapper_attributes()`, the block-supports machinery serializes the gradient CSS custom properties (e.g. `--wp--preset--gradient--*`) onto the `<a>` element automatically. When there is no previous page the block returns an empty string, so no empty box is painted.

The remaining diff is documentation regeneration:
- `docs/reference-guides/core-blocks/README.md` — the supports line for `core/query-pagination-previous` now lists `background (gradient)`.
- `packages/block-library/src/query-pagination-previous/README.md` — a new `background.gradient: true` entry is added to the supports list.

No changes to `render.php`, `index.js`, or any PHP template file are present in the diff.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @ramonjd. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no visible design debate or rejected alternatives; it was merged straightforwardly as a consistency follow-up to #43241.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
