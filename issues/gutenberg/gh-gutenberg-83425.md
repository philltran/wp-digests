# #83425: Group: Add text columns support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Group`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`e91711d`](https://github.com/WordPress/gutenberg/commit/e91711d5e23779c48af3dc34f0e20ca3a3304edd)
- **Discussion:** [#83425](https://github.com/WordPress/gutenberg/pull/83425) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds `typography.textColumns` support to the Group block, enabling the CSS `column-count` property on the Group wrapper. Paragraph and Post Excerpt already exposed this support, but Group did not, leaving no way to set a column count on a Group from Global Styles or the block inspector. This closes that gap as part of the design-tools consistency work tracked in #43241.

## Impact

- **Site owners / content editors:** Can now set a column count on any Group block via Global Styles (Blocks → Group → Typography → Columns) or the block inspector's Typography panel. The control is hidden by default and must be enabled from the Typography options menu.
- **Theme & block-theme developers:** No code changes required. The support is purely additive to `block.json`; existing Group markup is unaffected unless a column count is explicitly set. If you previously worked around this by wrapping paragraphs in a Columns block or applying `column-count` via custom CSS, you can now use the native support.
- **Plugin developers:** No breaking changes, no new hooks, no API surface change beyond the `block.json` entry.
- **No migration or configuration changes needed.**

## Technical details

The diff is minimal. In `packages/block-library/src/group/block.json`, the `typography` supports object gains one key:

```json
"typography": {
  "fontSize": true,
  "lineHeight": true,
  "textColumns": true,
  "__experimentalFontFamily": true,
  "__experimentalFontWeight": true,
  "__experimentalFontStyle": true
}
```

Because Group is a static block whose `save.jsx` builds the wrapper element from `useBlockProps.save()`, the `column-count` style is serialized into the saved markup automatically — no PHP render-callback change is needed. The `column-count` CSS property is applied to the Group wrapper element itself, so all inner blocks flow through the columns as a single content unit rather than each inner block receiving its own column layout.

Two documentation files are regenerated to reflect the new support: `packages/block-library/src/group/README.md` and `docs/reference-guides/core-blocks/README.md` (the latter now lists `textColumns` in Group's supports line).

## Contribution

Opened by @aaronrobertshaw with @andrewserong as co-author. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." The record carries no human review discussion — the only two comments are automated bot posts (performance metrics and a flaky-test report for an unrelated Cover block spec). Merged as `e91711d`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
