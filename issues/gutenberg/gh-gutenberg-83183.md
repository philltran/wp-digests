# #83183: Tab Panel: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`3fe6137`](https://github.com/WordPress/gutenberg/commit/3fe61370701c9835258053af6bb674df1e634e81)
- **Discussion:** [#83183](https://github.com/WordPress/gutenberg/pull/83183) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Tab Panel block (`core/tab-panel`) now declares the `shadow` block support, allowing site editors to apply box-shadow presets via Global Styles or the block inspector. This closes a gap where the block already supported color, layout, spacing, and typography but had no shadow control, as part of the broader design-tools consistency effort tracked in #43241. The change is a single-line addition to `block.json` plus regenerated documentation; no PHP or JavaScript logic changes were required.

## Impact

- **Site owners / block-theme users:** Can now set a shadow on Tab Panel blocks in Global Styles (Blocks → Tab Panel → Shadow) or per-instance in the block inspector, including responsive (mobile) overrides. No action required.
- **Plugin & theme developers:** No code changes needed. The `shadow` support is a standard block-support key; any theme or plugin that already handles `shadow` via the block supports pipeline will pick it up automatically for this block.
- **No breaking changes, deprecations, or migrations.**

## Technical details

The functional change is one line in `packages/block-library/src/tab-panel/block.json`, adding `"shadow": true` to the `supports` object alongside the existing `layout`, `spacing`, `color`, and `typography` entries:

```json
"supports": {
  "anchor": true,
  "color": { "background": true, "text": true },
  "layout": true,
  "shadow": true,
  "spacing": { "blockGap": true, "padding": true },
  "typography": { "fontSize": true }
}
```

The two README files (`docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/tab-panel/README.md`) are regenerated to list `shadow` in the supports table.

No PHP change is needed: the Tab Panel render callback uses `WP_HTML_Tag_Processor` to inject only `id`, `aria-labelledby`, and Interactivity API attributes, so the `class` and `style` attributes that carry the serialized shadow (written by `save.js`) pass through untouched to the front end.

## Contribution

Opened by @aaronrobertshaw with @ramonjd as co-author. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The discussion contains only two bot comments (performance metrics and contributor attribution) with no design debate or alternative approaches raised.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
