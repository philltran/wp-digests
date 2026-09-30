# #64601: Query No Results: Add Border & Spacing Support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @shail-mehta
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Feature] Design Tools`, `[Block] No Results`
- **Merged:** [`4828b9f`](https://github.com/WordPress/gutenberg/commit/4828b9f2ed37a6fbdf7399a1c1f75acf840649a5)
- **Discussion:** [#64601](https://github.com/WordPress/gutenberg/pull/64601) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `core/query-no-results` block now supports spacing (margin and padding) and border (radius, color, width, style) controls in the block editor and Global Styles. The change is declared in `block.json`, and a small stylesheet is added so padding and borders behave predictably.

## Impact

**Site owners / theme authors**
- The Query No Results block gets margin, padding, and border controls in the inspector and under Appearance > Editor > Styles > Blocks. Border and spacing values can also be set in `theme.json` under `styles.blocks['core/query-no-results']`.
- Controls are hidden by default in the UI (`__experimentalDefaultControls` is `false` for margin and padding) and are toggled on via the tools panel menu.

**Plugin & theme developers**
- No API removals or deprecations. Existing content is unaffected until a user sets values.
- The block now enqueues its own stylesheet, `wp-block-query-no-results`. Themes that style `.wp-block-query-no-results` with their own `box-sizing` or padding may want to check the result, though the rule uses `:where()` (zero specificity) and is easy to override.

**Headless & REST consumers**
- No REST or schema changes. Saved markup will carry the standard border and spacing classes and inline styles once values are set.

## Technical details

Changes in `packages/block-library/src/query-no-results/block.json`:

```json
"spacing": {
  "padding": true,
  "margin": true,
  "__experimentalDefaultControls": { "margin": false, "padding": false }
},
"__experimentalBorder": {
  "radius": true, "color": true, "width": true, "style": true
},
"style": "wp-block-query-no-results"
```

- A new `query-no-results/style.scss` sets `box-sizing: border-box` inside `:where(.wp-block-query-no-results)`. The comment in the file explains the low specificity is so a parent layout can still enforce width, and border-box makes padding more predictable.
- The stylesheet is registered via `@use "./query-no-results/style.scss" as *;` in `packages/block-library/src/style.scss`, and it is also built as a standalone per-block file (`build/styles/block-library/query-no-results/style.css`), which is why the `style` field was added.
- The `README.md` files (block-level and `docs/reference-guides/core-blocks/README.md`) are updated to list `spacing (margin, padding)`. The block-level README lists only spacing, not the border support.
- Border uses the `__experimentalBorder` key rather than the stable `border` key.

## Contribution

Part of the broader effort to add design-tool support across core blocks (issue #43247). @aaronrobertshaw reviewed the PR. Review feedback led @shail-mehta to rebase on trunk and to add the `box-sizing: border-box` rule under a low-specificity `:where()` selector, so the block can still honor a fixed width set by a parent.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
