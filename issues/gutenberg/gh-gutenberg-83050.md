# #83050: Login/out: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Login/out`
- **Merged:** [`0644c1d`](https://github.com/WordPress/gutenberg/commit/0644c1d12247ff548ed87f09a1bcbed1cb3a17b2)
- **Discussion:** [#83050](https://github.com/WordPress/gutenberg/pull/83050) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Login/out block (`core/loginout`) now supports the `shadow` block support, allowing designers to apply box-shadow presets via Global Styles or the block inspector. The change is a single-line addition of `"shadow": true` to the block's `block.json` supports object; because the block is server-rendered through `get_block_wrapper_attributes()`, the shadow CSS custom properties are serialized onto the wrapper automatically with no PHP template change. This closes a gap in the design-tools consistency effort tracked in #43241, where the block already supported color, border, spacing, and typography but not shadow.

## Impact

- **Site owners / theme editors (block themes):** Can now set a shadow on the Login/out block in Global Styles (Blocks → Login/out → Shadow) or per-instance in the block inspector. No code changes required.
- **Plugin & theme developers:** No action required. The support is additive; existing themes and plugins that render `core/loginout` will simply receive the new CSS custom properties on the wrapper when a shadow is applied. No new hooks, filters, or attributes are introduced.
- **Headless / REST consumers:** No schema or route changes. The block's REST representation is unchanged; shadow is a presentation-layer concern handled by the block-supports CSS pipeline.

## Technical details

The functional change is one line in `packages/block-library/src/loginout/block.json`, inside the existing `supports` object:

```json
"supports": {
  "spacing": { "margin": true, "padding": true },
  "color": { "background": true, "text": false, "gradients": true, "link": true },
  "typography": { "fontSize": true, "lineHeight": true },
  "border": { "color": true, "radius": true, "style": true, "width": true, "style": true },
  "shadow": true,          // ← added
  "interactivity": { "clientNavigation": true }
}
```

Because `core/loginout` is a server-rendered block whose markup is produced by `get_block_wrapper_attributes()`, the block-supports CSS pipeline (the `--wp--preset--shadow--*` custom properties and the corresponding `box-shadow` declaration) is emitted onto the wrapper `<div>` with no change to the block's PHP render callback or template. The two README files (`docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/loginout/README.md`) are regenerated to list `shadow` in the supports table.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @shail-mehta. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. @aaronrobertshaw left a brief thank-you for responsive-styles testing. The discussion is minimal (3 comments, 0 reactions) with no design debate or rejected alternatives visible in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
