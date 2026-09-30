# #83051: Latest Posts: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Latest Posts`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`4ed0061`](https://github.com/WordPress/gutenberg/commit/4ed0061c1ab6d6ab19a97e5825c8c58ad08d217e)
- **Discussion:** [#83051](https://github.com/WordPress/gutenberg/pull/83051) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Latest Posts block (`core/latest-posts`) now supports the `shadow` block support, allowing shadow presets to be applied via Global Styles or the block inspector. This brings the block in line with other core blocks that already expose shadow as part of the design tools consistency effort (related to #43241). Because the block is server-rendered through `get_block_wrapper_attributes()`, the support is serialized onto the list wrapper automatically with no PHP template change required.

## Impact

- **Site owners / editors:** Can now apply a shadow preset to the Latest Posts block from Global Styles (Blocks → Latest Posts → Shadow) or from the block-level inspector. No code changes needed.
- **Theme developers:** If you override or extend the Latest Posts block's rendering, the wrapper element will now carry a `--wp--shadow` custom property and `box-shadow` declaration when a shadow is set. No action required unless you strip or override wrapper styles.
- **Plugin & theme developers:** No breaking changes, no new hooks, no new REST schema fields. The standard `shadow` support key is now present in the block's `block.json`.
- **No migration or configuration changes required.**

## Technical details

The change is a single-line addition to `packages/block-library/src/latest-posts/block.json`, inserting `"shadow": true` into the `supports` object (between the existing `spacing` and `interactivity` entries). Because the block's server-side render path calls `get_block_wrapper_attributes()`, the block editor's style engine serializes the chosen shadow preset onto the `<ul>` wrapper as inline CSS custom properties and a `box-shadow` declaration — no change to `render.php` or any PHP file is needed.

Two documentation files are regenerated to reflect the new support:

- `docs/reference-guides/core-blocks/README.md` — the Latest Posts entry's **Supports** line gains `shadow`.
- `packages/block-library/src/latest-posts/README.md` — a new bullet lists `shadow: true` under the supports section.

Before (block.json supports excerpt):
```json
"spacing": { "blockGap": true, "margin": true, "padding": true },
"interactivity": { "clientNavigation": true }
```

After:
```json
"spacing": { "blockGap": true, "margin": true, "padding": true },
"shadow": true,
"interactivity": { "clientNavigation": true }
```

No new hooks, filters, REST routes, or DB changes are introduced.

## Contribution

Authored by @aaronrobertshaw with co-authorship from @jorgefilipecosta. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no visible design debate or rejected alternatives; it was merged as a straightforward consistency addition under the broader #43241 design-tools tracking effort.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
