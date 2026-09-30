# #83018: Post Excerpt: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Block] Post Excerpt`, `[Feature] Design Tools`
- **Merged:** [`ab28edb`](https://github.com/WordPress/gutenberg/commit/ab28edb3b6aa9e1481bae67a37d92c37e00d786c)
- **Discussion:** [#83018](https://github.com/WordPress/gutenberg/pull/83018) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Post Excerpt block (`core/post-excerpt`) now declares the `shadow` block support, allowing site editors to apply box-shadow presets via Global Styles or the block inspector. This closes a gap in the block's design-tool coverage (it already supported color, border, spacing, and typography) as part of the broader design-tools consistency effort tracked in #43241. Because the block is server-rendered through `get_block_wrapper_attributes()`, the support is serialized onto the wrapper element automatically with no PHP render-callback change.

## Impact

- **Site owners / theme editors (block themes):** The Excerpt block now exposes a Shadow control in Global Styles (Blocks → Excerpt → Shadow) and in the block inspector. Block-level shadow overrides the Global Style value, consistent with other supported blocks.
- **Plugin & theme developers:** No code changes required. The `shadow` support is handled by the existing block-supports infrastructure; any theme that already ships shadow CSS custom properties (e.g. `--wp--preset--shadow--natural`) will work without modification.
- **Headless / REST consumers:** No schema or route changes. The block's `block.json` `supports` object gains a `"shadow": true` entry, which is visible if you inspect the block registration, but no new attribute or REST field is introduced.
- **No action required** for existing sites; the change is purely additive.

## Technical details

The functional change is a single line in `packages/block-library/src/post-excerpt/block.json`, adding `"shadow": true` to the `supports` object:

```json
"supports": {
  "color": { "background": true, "gradients": true, "link": true },
  "shadow": true,
  "spacing": { "margin": true, "padding": true },
  "typography": { "fontSize": true, "lineHeight": true, "textAlign": true, "textColumns": true }
}
```

Because `core/post-excerpt` renders server-side via `get_block_wrapper_attributes()`, the block-supports pipeline automatically emits the relevant `--wp--preset--shadow--*` CSS custom properties and the `box-shadow` declaration on the wrapper `<div>`. No change to the render callback or any PHP file is needed.

The remaining two files in the diff are regenerated documentation:
- `docs/reference-guides/core-blocks/README.md` — `shadow` added to the supports list for `core/post-excerpt`.
- `packages/block-library/src/post-excerpt/README.md` — a new bullet documenting the `shadow` support with a link to the block-supports reference.

## Contribution

Opened and authored by @aaronrobertshaw with co-author @yogeshbhutkar. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no design debate or rejected alternatives visible in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
