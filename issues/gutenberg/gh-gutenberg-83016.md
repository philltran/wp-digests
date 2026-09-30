# #83016: Post Content: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Post Content`
- **Merged:** [`89629b2`](https://github.com/WordPress/gutenberg/commit/89629b2738602a0ae59ea38b953bf12c965cc630)
- **Discussion:** [#83016](https://github.com/WordPress/gutenberg/pull/83016) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `core/post-content` block now supports the `shadow` block support, closing a gap where it already supported color, background, border, spacing, and typography but not shadow. The change is a single-line addition to the block's `block.json` `supports` object; because the block is server-rendered through `get_block_wrapper_attributes()`, no PHP changes are required. This is part of the broader design-tools consistency effort tracked in #43241.

## Impact

- **Block theme developers:** The Post Content block in templates (e.g. `single.html`) can now receive a shadow via Global Styles (Blocks → Content → Shadow) or a per-instance override in the block inspector. No code changes needed.
- **Plugin developers:** No new hooks, filters, or REST schema changes. The existing `shadow` support machinery handles serialization.
- **Site owners / hosting:** No action required. Existing sites are unaffected; the shadow only appears if explicitly applied in Global Styles or a template.

## Technical details

The diff touches three files, all in the `core/post-content` block package:

1. **`packages/block-library/src/post-content/block.json`** — adds `"shadow": true` to the `supports` object, between the existing `dimensions` and `spacing` entries:

```json
"supports": {
  "dimensions": { "minHeight": true },
  "shadow": true,
  "spacing": { "blockGap": true, "padding": true, "margin": true }
}
```

2. **`packages/block-library/src/post-content/README.md`** — regenerated to list `shadow: true` under the supports table.
3. **`docs/reference-guides/core-blocks/README.md`** — regenerated; the Post Content entry's supports line now includes `shadow`.

No PHP, JS, or CSS changes. The block's render path calls `get_block_wrapper_attributes()`, which already serializes any declared `shadow` support onto the wrapper element's `style` attribute, so the new support is picked up automatically.

## Contribution

Opened by @aaronrobertshaw with @ramonjd as co-author. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." The discussion is minimal (2 comments, 0 reactions) with no visible design debate or rejected alternatives; it was a straightforward consistency addition under the #43241 design-tools umbrella.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
