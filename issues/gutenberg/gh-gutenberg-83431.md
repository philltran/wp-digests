# #83431: Tabs: Add typography support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Tabs`
- **Merged:** [`c6001fe`](https://github.com/WordPress/gutenberg/commit/c6001fef34f00ebbf3f30b8e8342ba73922d4b39)
- **Discussion:** [#83431](https://github.com/WordPress/gutenberg/pull/83431) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Tabs block (`core/tabs`) gains five new typography supports: `lineHeight`, `__experimentalFontStyle`, `__experimentalFontWeight`, `__experimentalTextTransform`, and `__experimentalLetterSpacing`. Previously the block only exposed `fontSize` and `__experimentalFontFamily`. Because the Tabs block wraps both the tab row and the panel area in a single container, one set of typography values covers tab labels and panel content simultaneously. This brings the block in line with other container blocks as part of the design-tools consistency effort (related to #43241).

## Impact

- **Theme & site builders (Global Styles):** The Tabs block now appears with line-height, font-style, font-weight, letter-spacing, and text-transform controls under Global Styles → Blocks → Tabs → Typography. No code changes needed; the controls are available immediately after updating.
- **Block developers:** The `core/tabs` `block.json` `supports.typography` object now lists five additional keys. Any tooling that reads block supports (e.g., style-engine serializers, block-directory metadata) will pick up the new entries automatically.
- **No action required** for existing sites. The new supports are additive; blocks without explicit values render identically to before.
- **Known limitation:** `__experimentalFontStyle` (italic) does not reach the tab-label buttons because the UA `font` shorthand on `<button>` resets `font-style` to `normal` and the tab-list button reset does not re-inherit it. Italic applies to panel content only. The PR notes this fix belongs in a separate change.

## Technical details

The diff modifies three files:

1. **`packages/block-library/src/tabs/block.json`** — the `supports.typography` object expands from two keys to seven:

```json
"typography": {
  "fontSize": true,
  "lineHeight": true,
  "__experimentalFontFamily": true,
  "__experimentalFontWeight": true,
  "__experimentalFontStyle": true,
  "__experimentalTextTransform": true,
  "__experimentalLetterSpacing": true
}
```

The key order mirrors the Group block's typography supports (minus `textDecoration`, which is deliberately excluded because the tab-list button reset sets `text-decoration: none` and an inherited underline on every tab label and panel link is not a useful container-level control).

2. **`packages/block-library/src/tabs/README.md`** — adds a `lineHeight: true` line under the typography supports list. The four `__experimental` keys are omitted because the docs generator skips them.

3. **`docs/reference-guides/core-blocks/README.md`** — the Tabs entry's supports string changes from `typography (fontSize)` to `typography (fontSize, lineHeight)`.

No PHP or render-callback changes are needed: the wrapper element comes from `useBlockProps.save()`, so the style-engine serializes the new supports into the block's `style` attribute automatically. The render callback only adds Interactivity API attributes via `WP_HTML_Tag_Processor` and never writes `class` or `style`.

The PR description notes that `tab-list/style.scss` already re-inherits `font-size`, `font-family`, `font-weight`, `line-height`, `letter-spacing`, and `text-transform` onto the tab buttons as part of its button reset, so those five properties flow into tab labels as well as panel content. That CSS is not part of this diff.

## Contribution

Authored by @aaronrobertshaw with co-authorship from @ramonjd. The PR body states the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no visible design debate or rejected alternatives beyond the stated decision to exclude text decoration and defer the `font-style` button-reset fix to a separate change.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
