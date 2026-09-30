# #83044: Footnotes: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Footnotes`
- **Merged:** [`d93a94d`](https://github.com/WordPress/gutenberg/commit/d93a94d1784aaa98de67cd803a23d148d04bc832)
- **Discussion:** [#83044](https://github.com/WordPress/gutenberg/pull/83044) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Footnotes block (`core/footnotes`) now declares the `shadow` block support, allowing site editors to apply box-shadow presets via Global Styles or the block inspector. This closes a gap in the block's design controls (it previously supported color, border, spacing, and typography but not shadow) as part of the broader design-tools consistency effort tracked in #43241. Because the block is server-rendered through `get_block_wrapper_attributes()`, the change is a single `block.json` addition with no PHP or JS logic changes.

## Impact

- **Site owners / editors:** The Footnotes block now exposes a Shadow control in Global Styles (Blocks → Footnotes → Shadow) and in the block inspector. Block-level shadow overrides Global Style shadow, consistent with other core blocks.
- **Plugin & theme developers:** No action required. The change is purely additive to `block.json`; no new hooks, filters, or REST schema fields are introduced. Themes that already handle `--wp--preset--shadow--*` custom properties will pick up the value automatically via the wrapper attributes.
- **Hosting & platform:** No migration, configuration, or code changes needed.

## Technical details

The diff touches three files, all additive:

1. **`packages/block-library/src/footnotes/block.json`** — adds `"shadow": true` to the `supports` object, alongside the existing `spacing`, `typography`, `color`, and `anchor` entries.
2. **`packages/block-library/src/footnotes/README.md`** — regenerated to list `shadow: true` in the supports table.
3. **`docs/reference-guides/core-blocks/README.md`** — regenerated to include `shadow` in the Footnotes block's supports summary line.

No PHP or JavaScript source files are modified. The Footnotes block renders server-side via `get_block_wrapper_attributes()`, which reads the `supports` declaration and serializes the resolved shadow CSS custom properties (e.g. `--wp--preset--shadow--natural`) onto the `<ol>` wrapper element. The block's existing `render.php` template already passes those attributes through, so the shadow value flows to the front end without template changes.

Before (no shadow support):
```json
"supports": {
  "anchor": true,
  "color": { "background": true, "link": true, "text": true },
  "spacing": { "margin": true, "padding": true },
  "typography": { "fontSize": true, "lineHeight": true }
}
```

After:
```json
"supports": {
  "anchor": true,
  "color": { "background": true, "link": true, "text": true },
  "shadow": true,
  "spacing": { "margin": true, "padding": true },
  "typography": { "fontSize": true, "lineHeight": true }
}
```

## Contribution

Opened by @aaronrobertshaw with @ramonjd as co-author. The PR body notes the implementation was produced by a Claude Code agent from a predefined task. The two comments on the PR are automated (performance metrics and contributor-credit bot); no design debate or alternative approaches are recorded. The work is part of the ongoing #43241 design-tools consistency initiative that has been incrementally adding missing supports to core blocks.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
