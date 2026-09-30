# #83176: Query Total: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Query Total`
- **Merged:** [`75ce5f0`](https://github.com/WordPress/gutenberg/commit/75ce5f0cc230a058cfdeaebd7208bb6e671bfa1c)
- **Discussion:** [#83176](https://github.com/WordPress/gutenberg/pull/83176) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Query Total block (`core/query-total`) now supports the `shadow` block support, allowing box-shadow styles to be applied via Global Styles or the block inspector. This closes a gap in the block's design controls (it previously supported color, border, spacing, and typography but not shadow) as part of the broader design-tools consistency effort tracked in #43241. The change is a single-line addition to the block's `block.json`; no PHP or JavaScript logic changes are required because the block is server-rendered through `get_block_wrapper_attributes()`, which serializes shadow styles onto the wrapper automatically.

## Impact

- **Site owners / editors:** Can now assign a shadow preset (Natural, Deep, Sharp, Crisp, etc.) to Query Total blocks in Global Styles or per-instance in the block inspector, including responsive (mobile/tablet) variants.
- **Theme & plugin developers:** No code changes required. The support is declarative in `block.json`; any theme or plugin that renders `core/query-total` via `get_block_wrapper_attributes()` will pick up the shadow styles automatically. If you build custom Query Total renderers that bypass that function, you will need to handle the `--wp--shadow` custom property yourself.
- **No breaking changes, deprecations, or migrations.** Existing Query Total blocks render identically until a shadow is explicitly applied.

## Technical details

The diff touches three files:

1. **`packages/block-library/src/query-total/block.json`** — adds `"shadow": true` to the `supports` object, alongside the existing `color`, `spacing`, `typography`, and `interactivity` entries.

2. **`packages/block-library/src/query-total/README.md`** — regenerated to list `shadow: true` under the supports section.
3. **`docs/reference-guides/core-blocks/README.md`** — regenerated to include `shadow` in the Query Total supports summary line.

Because `core/query-total` is a server-rendered block whose markup is produced by `get_block_wrapper_attributes()`, the shadow support serializes as a `--wp--shadow` custom property and a `box-shadow` declaration on the wrapper `<div>`. No changes to `render.php`, `index.js`, or any PHP template are needed.

Before (no shadow control available):
```json
"supports": {
  "color": { "background": true, "text": true, "gradients": true, "style": true },
  "spacing": { "margin": true, "padding": true },
  "typography": { "fontSize": true, "lineHeight": true },
  "interactivity": { "clientNavigation": true }
}
```

After:
```json
"supports": {
  "color": { "background": true, "text": true, "gradients": true, "style": true },
  "spacing": { "margin": true, "padding": true },
  "typography": { "fontSize": true, "lineHeight": true },
  "shadow": true,
  "interactivity": { "clientNavigation": true }
}
```

## Contribution

Opened by @aaronrobertshaw with @andrewserong as co-author. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The discussion contains only the two automated bot comments (performance metrics and co-author list); no design debate or alternative approaches were raised. Merged as `75ce5f0`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
