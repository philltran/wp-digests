# #83186: Tabs: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Tabs`
- **Merged:** [`2e4c9f9`](https://github.com/WordPress/gutenberg/commit/2e4c9f9764e4f24f4bca5ddabfaa52ea3fef0f76)
- **Discussion:** [#83186](https://github.com/WordPress/gutenberg/pull/83186) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Tabs block (`core/tabs`) now declares the `shadow` block support, allowing shadow styling through Global Styles and per-block overrides, including responsive variants. This closes a gap in the Tabs block's design-tool coverage (it previously supported color, layout, spacing, and typography but not shadow) as part of the broader design-tools consistency effort tracked in #43241.

## Impact

- **Site owners / block-theme users:** Can now pick a shadow preset (Natural, Deep, Sharp, Crisp, etc.) for the Tabs block in Global Styles → Blocks → Tabs → Shadow, or override it per instance in the block inspector. Responsive shadow variants work via the States menu.
- **Plugin & theme developers:** No action required. The change is purely additive to the block's `supports` declaration; no new hooks, REST routes, or PHP render logic were introduced.
- **Headless / REST consumers:** No schema or serialization change beyond the standard `class`/`style` attributes that the shadow support already produces on the saved block markup.

## Technical details

The functional change is a single line in `packages/block-library/src/tabs/block.json`, adding `"shadow": true` to the `supports` object:

```json
"supports": {
  "spacing": {
    "blockGap": true,
    "margin": true,
    "padding": true
  },
  "shadow": true,
  "typography": { "fontSize": true, "__experimentalFontFamily": true }
}
```

Because the Tabs block's wrapper element is emitted from `save.js`, the shadow support is serialized directly into the block's saved markup as `class` and `style` attributes. The PHP render callback only injects Interactivity API attributes via `WP_HTML_Tag_Processor`, so those saved attributes pass through untouched — no PHP change was needed.

Two documentation files are regenerated to reflect the new support: `docs/reference-guides/core-blocks/README.md` (adds `shadow` to the Tabs supports list) and `packages/block-library/src/tabs/README.md` (adds a `shadow: true` entry under the supports section).

## Contribution

Authored by @aaronrobertshaw with co-authorship from @ramonjd. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The record carries no design debate or alternative-approach discussion beyond the two bot comments (performance metrics and flaky-test report).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
