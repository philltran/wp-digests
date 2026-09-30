# #83578: Post Excerpt: Add minimum width and height support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Block] Post Excerpt`, `[Feature] Design Tools`
- **Merged:** [`6b564d0`](https://github.com/WordPress/gutenberg/commit/6b564d0b17079d2874fdc2c5417b10a6bc9802c0)
- **Discussion:** [#83578](https://github.com/WordPress/gutenberg/pull/83578) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Post Excerpt block (`core/post-excerpt`) now declares `dimensions.minWidth` and `dimensions.minHeight` in its `block.json` supports, enabling minimum-size controls in Global Styles and the block inspector. This brings the block in line with other core blocks that already expose dimension controls, as part of the broader design-tools consistency effort tracked in #43241. No PHP changes were required because the block is server-rendered through `get_block_wrapper_attributes()`, which serializes the new support automatically.

## Impact

- **Site owners / block-theme builders:** A new Dimensions panel (minimum width, minimum height) appears under Global Styles → Blocks → Excerpt and in the block inspector. Values can be set per breakpoint (desktop, mobile) and per-block, following the standard cascade (block value > Global Styles).
- **Plugin & theme developers:** No code changes needed. If you build custom render callbacks for `core/post-excerpt`, be aware the wrapper `<p>` element is always present, so `min-width`/`min-height` CSS will apply to it. No new hooks, filters, or attributes are introduced.
- **No action required** for existing sites; the change is purely additive and only takes effect when a minimum dimension is explicitly set.

## Technical details

The diff makes three file changes:

1. **`packages/block-library/src/post-excerpt/block.json`** — adds a `dimensions` key to the existing `supports` object:

```json
"dimensions": {
    "minHeight": true,
    "minWidth": true
}
```

2. **`packages/block-library/src/post-excerpt/README.md`** — regenerated to list the new `dimensions` support with `minHeight: true` and `minWidth: true`.

3. **`docs/reference-guides/core-blocks/README.md`** — the Post Excerpt entry's supports line is updated to include `dimensions (minHeight, minWidth)`.

No changes to `render.php`, `index.js`, or any PHP file. The block's render callback always wraps the excerpt in a `<p>` element, so `get_block_wrapper_attributes()` has a wrapper to attach the serialized `min-width`/`min-height` inline styles to. The standard block-supports machinery handles the rest: Global Styles serialization, per-breakpoint values, and the block-inspector Dimensions panel are all driven by the `block.json` declaration alone.

## Contribution

Authored by @aaronrobertshaw with co-authorship from @ramonjd. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." The discussion is minimal — two comments (both from the GitHub Actions bot posting performance metrics and a flaky-test report) and zero reactions. No design debate or alternative approaches are visible in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
