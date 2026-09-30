# #83428: Tab Panels: Add typography support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`b2e8b03`](https://github.com/WordPress/gutenberg/commit/b2e8b03fd2734cd9dccf628edc1592d7d53804cc)
- **Discussion:** [#83428](https://github.com/WordPress/gutenberg/pull/83428) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Tab Panels block (`core/tab-panels`) now supports six additional typography controls — line height, font style, font weight, letter spacing, text transform, and text decoration — bringing it in line with other core blocks as part of the design-tools consistency effort (related to #43241). Previously only `fontSize` and `__experimentalFontFamily` were available. Because the block is static and its `save` uses `useBlockProps.save()`, the new supports serialize onto the wrapper element automatically with no PHP or render change.

## Impact

- **Site owners / editors:** New typography controls appear in the block inspector and under Global Styles → Blocks → Tab Panels → Typography. Values set at the Tab Panels level apply to every panel and persist as the reader switches tabs.
- **Plugin & theme developers:** No code changes required. The new supports are declared in `block.json` and handled by the standard block-supports machinery. If you were previously applying these properties via custom CSS on `.wp-block-tab-panels`, the built-in controls now cover them.
- **No breaking changes.** Existing saved markup is unaffected; the new supports are additive. No action required.

## Technical details

The change is confined to `packages/block-library/src/tab-panels/block.json`. The `typography` supports object gains six keys alongside the existing `fontSize` and `__experimentalFontFamily`:

```json
"typography": {
  "fontSize": true,
  "lineHeight": true,
  "__experimentalFontFamily": true,
  "__experimentalFontWeight": true,
  "__experimentalFontStyle": true,
  "__experimentalTextTransform": true,
  "__experimentalTextDecoration": true,
  "__experimentalLetterSpacing": true
}
```

Key order mirrors the Group block, which carries the same set. Only `lineHeight` is a stable (non-experimental) key, so it is the only one that appears in the regenerated docs (`docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/tab-panels/README.md`); the five `__experimental*` keys are skipped by both docs generators.

No stylesheet ships with Tab Panels, so all six properties inherit normally into the panels and their content. This contrasts with the sibling Tabs and Tab List blocks, where `tab-list/style.scss` resets `text-decoration` on tab buttons and omits `font-style` from its re-inherit list — the reason those two properties were excluded from the equivalent Tabs/Tab List changes.

Because the block's `save` calls `useBlockProps.save()`, the supports are serialized onto the wrapper `<div>` automatically; no PHP render function change is needed.

## Contribution

Opened by @aaronrobertshaw and co-authored with @ramonjd. The PR notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no visible design debate or rejected alternatives; the approach of mirroring Group's key order and omitting font-style/text-decoration from the sibling Tabs/Tab List blocks (due to their stylesheet resets) is stated as rationale in the PR body rather than arising from review.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
