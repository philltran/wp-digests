# #83426: Tab Panel: Add typography support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`c72e82f`](https://github.com/WordPress/gutenberg/commit/c72e82f2d92a4147ad5990801eddd2d7b330d7ba)
- **Discussion:** [#83426](https://github.com/WordPress/gutenberg/pull/83426) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Tab Panel block (`core/tab-panel`) gains six new typography supports — `lineHeight`, `__experimentalFontWeight`, `__experimentalFontStyle`, `__experimentalTextTransform`, `__experimentalTextDecoration`, and `__experimentalLetterSpacing` — alongside the existing `fontSize` and `__experimentalFontFamily`. This closes a gap where Tab Panel was the only block in the Tabs family missing these controls, as part of the broader design-tools consistency effort (related to #43241). No PHP or render-callback changes are required because the block's Tag Processor only sets `id`, `aria-labelledby`, and Interactivity API attributes, leaving the class and style attributes emitted by `useBlockProps.save()` untouched.

## Impact

- **Site owners / editors:** Tab Panel now exposes line height, font weight, font style (italic), letter spacing, text transform, and text decoration in both the block inspector (Typography panel) and Global Styles → Blocks → Tab Panel → Typography. The `__experimentalDefaultControls` object is unchanged, so only `fontSize` and `__experimentalFontFamily` appear by default in the inspector; the other five must be enabled via Global Styles or by selecting the block.
- **Plugin & theme developers:** No code changes required. If you build custom styles targeting `.wp-block-tab-panel`, be aware that the element can now carry `font-weight`, `font-style`, `letter-spacing`, `text-transform`, `text-decoration-line`, and `line-height` inline styles from the block editor. No new hooks, filters, or REST schema changes.
- **No action required** for existing sites; the change is purely additive to `block.json` supports.

## Technical details

The sole functional change is in `packages/block-library/src/tab-panel/block.json`, under the `supports.typography` object:

```json
"typography": {
  "fontSize": true,
  "lineHeight": true,
  "__experimentalFontFamily": true,
  "__experimentalFontWeight": true,
  "__experimentalFontStyle": true,
  "__experimentalTextTransform": true,
  "__experimentalTextDecoration": true,
  "__experimentalLetterSpacing": true,
  "__experimentalDefaultControls": {
    "fontSize": true,
    "__experimentalFontFamily": true
  }
}
```

`lineHeight` is a stable key and is the only new entry that appears in the regenerated docs (`docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/tab-panel/README.md`); the five `__experimental*` keys are skipped by both docs generators.

The PR description notes that the equivalent Tabs and Tab List blocks deliberately omit `__experimentalFontStyle` and `__experimentalTextDecoration` because `tab-list/style.scss` resets `text-decoration` on tab buttons and excludes `font-style` from its re-inherit list. Tab Panel renders a plain `<section>` that inherits normally, so the full set applies here, matching the Group block's behavior.

## Contribution

Opened by @aaronrobertshaw and implemented via a Claude Code agent from a predefined task. @shail-mehta reviewed and flagged a merge conflict; @ramonjd resolved it and merged. The PR carries three co-author credits (aaronrobertshaw, ramonjd, shail-mehta). No design debate or rejected alternatives are recorded in the three comments.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
