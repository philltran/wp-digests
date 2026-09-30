# #83593: Site Title: Add minimum width and height support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Site Title`
- **Merged:** [`b685abd`](https://github.com/WordPress/gutenberg/commit/b685abd200b8799c3e323ea730031125df3c84c9)
- **Discussion:** [#83593](https://github.com/WordPress/gutenberg/pull/83593) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Site Title block (`core/site-title`) now supports `dimensions.minHeight` and `dimensions.minWidth` in its `block.json` supports, bringing it in line with other core blocks that already expose dimension controls. Because the block is server-rendered and its render callback builds the wrapper via `get_block_wrapper_attributes()`, the new supports are serialized into the output automatically with no PHP change. Width, height, and aspect ratio are explicitly deferred to a later pass in the design-tools consistency audit.

## Impact

- **Theme & site builders:** Site Title now accepts minimum width and minimum height through Global Styles (including a separate Mobile state) and the block inspector. No code changes required; the controls appear in the existing Dimensions panel.
- **Plugin & theme developers:** If you build custom styles or overrides targeting `core/site-title`, the block's wrapper can now carry `min-width` and `min-height` inline styles. No new hooks, REST fields, or attributes are introduced.
- **No action required** for existing sites. The block renders identically when no minimum dimensions are set.

## Technical details

The functional change is a single addition to `packages/block-library/src/site-title/block.json`:

```json
"dimensions": {
    "minHeight": true,
    "minWidth": true
}
```

This is inserted between the existing `color` and `spacing` support objects. Because `core/site-title` is a server-rendered block whose render callback calls `get_block_wrapper_attributes()`, the block-supports serialization pipeline picks up the new keys and emits `min-width` / `min-height` inline styles on the wrapper element with no PHP modification.

The render callback returns an empty string when the site has no title, so no empty wrapper is produced for the new minimums to apply to.

Two documentation files are regenerated to reflect the new support:
- `docs/reference-guides/core-blocks/README.md` — the Site Title supports line now lists `dimensions (minHeight, minWidth)`.
- `packages/block-library/src/site-title/README.md` — a new `dimensions` section with `minHeight: true` and `minWidth: true` is added.

No new block attributes, hooks, REST schema fields, or database changes are introduced. The `dimensions` support key itself is an existing block-supports mechanism; this PR simply opts the Site Title block into two of its sub-keys.

## Contribution

Opened by @aaronrobertshaw as part of the design-tools consistency audit (related to #43241), which determined that Site Title is the sibling of Site Tagline and should receive the same dimension answers. The PR was implemented, built, and screenshotted by a Claude Code agent from a predefined task, with co-authorship credited to @shail-mehta. The record carries no substantive design debate; the two comments are the automated PR-meta performance report and the co-author attribution bot.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
