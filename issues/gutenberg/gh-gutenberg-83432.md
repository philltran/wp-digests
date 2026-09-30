# #83432: Terms List: Add text columns support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Categories`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`b31b2c7`](https://github.com/WordPress/gutenberg/commit/b31b2c7e3230dc9fe7b0d324ee39857393973cbd)
- **Discussion:** [#83432](https://github.com/WordPress/gutenberg/pull/83432) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Categories (Terms List) block gains `typography.textColumns` support, allowing its list-mode `<ul>` to flow into multiple CSS columns via the `column-count` property. This fills the last gap in the block's typography toolset, which already exposed font size, line height, font family, weight, style, transform, decoration, and letter spacing. The change is purely declarative in `block.json`; no PHP or JavaScript logic was added.

## Impact

- **Site owners / editors:** Can now set a column count on a Terms List block in list mode via Global Styles → Blocks → Terms List → Typography → Columns, or per-block in the Typography panel. The control is not a default; it must be enabled from the Typography panel's options menu (same as Post Excerpt).
- **Plugin & theme developers:** No code changes required. The support is serialized automatically by `get_block_wrapper_attributes()` in the block's dynamic render callback. No new hooks, filters, or REST schema changes.
- **Headless / REST consumers:** No change to block attributes or REST output; `textColumns` is a style support, not a block attribute.
- **No action required** for any audience. The feature is opt-in and additive.

## Technical details

The diff is three files, all adding the same key:

```json
// packages/block-library/src/categories/block.json
"typography": {
  "fontSize": true,
  "lineHeight": true,
  "textColumns": true,   // ← added
  "__experimentalFontFamily": true,
  "__experimentalFontWeight": true,
  "__experimentalFontStyle": true,
  ...
}
```

The two README files (`docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/categories/README.md`) are regenerated to list the new support.

Because the block is dynamic, its render callback builds the wrapper element with `get_block_wrapper_attributes()`, which reads the `typography.textColumns` support from the block's style context and emits the corresponding `style="column-count: N"` (or the equivalent CSS custom property) on the wrapper. No PHP change is needed.

The `WP_HTML_Tag_Processor` pass in the block's `index.php` render callback only adds a click directive to the individual term `<a>` links and does not touch the wrapper's `class` or `style` attributes, so it does not interfere with the serialized support.

The `appearanceTools` opt-in for `textColumns` in `lib/class-wp-theme-json-gutenberg.php` was already merged via the Paragraph block adoption (#74656), so no change to that file is required here.

In dropdown mode (`displayAsDropdown: true`), the wrapper is a `<div>` containing a `<label>` and a `<select>`. The `column-count` declaration is applied but has no visible effect on that layout, which is the expected behavior for a typography tool on a native form control.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @shail-mehta. The PR body notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. It is part of the broader design-tools consistency effort tracked in #43241 and mirrors the identical one-line `block.json` adoption already made for Post Excerpt (#75587) and Paragraph (#74656). The PR attracted only 2 comments and 0 reactions, with no design debate or alternative approaches discussed.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
