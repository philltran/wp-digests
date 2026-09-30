# #83615: Heading: Add text indent support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Heading`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`d5cde8f`](https://github.com/WordPress/gutenberg/commit/d5cde8f58a338a2f3b2af67d53cccdbf98a3496c)
- **Discussion:** [#83615](https://github.com/WordPress/gutenberg/pull/83615) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Heading block now supports the `text-indent` CSS property via the `typography.textIndent` block support, enabling it to be set through Global Styles (including responsive/mobile states) and the block inspector. This closes a gap in the Heading block's typography toolset, which previously covered font size, line height, alignment, family, style, weight, letter spacing, transform, decoration, writing mode, shadow, and fit text but not indent. The change is part of the broader design-tools consistency effort tracked in #43241.

## Impact

- **Site owners / content editors:** A new "Line indent" control appears under Typography in both the Global Styles panel (Blocks → Heading) and the block inspector. No migration or configuration required; existing headings render unchanged until a value is set.
- **Theme & plugin developers:** No code changes needed. The support serializes automatically through `useBlockProps.save()`. If a theme applies its own `text-indent` to `.wp-block-heading`, the block-level or Global Styles value will override it via the inline `style` attribute.
- **Headless / REST consumers:** No new REST schema fields or attributes are introduced; the value is stored in the block's serialized `style` attribute as with all other typography supports.
- **No action required** for existing sites or plugins.

## Technical details

The functional change is a single line in `packages/block-library/src/heading/block.json`, adding `"textIndent": true` to the `typography` supports object:

```json
"typography": {
  "fontSize": true,
  "lineHeight": true,
  "textAlign": true,
  "textIndent": true,
  "__experimentalFontFamily": true,
  "__experimentalFontStyle": true,
  "__experimentalFontWeight": true,
  ...
}
```

No `selectors` entry is provided. The Paragraph block (the only other block with this support) uses the selector `.wp-block-paragraph + .wp-block-paragraph` so that only a paragraph *following* another paragraph is indented. A heading is a single, non-nested text element, so the plain adoption applies `text-indent` to the block's own wrapper with no sibling combinator.

No PHP change is required. The Heading block's `index.php` post-processes saved markup with `WP_HTML_Tag_Processor`, but it only calls `add_class()` to append `wp-block-heading`; it does not touch the `class` or `style` attributes that the typography support writes, so the serialized `text-indent` declaration passes through intact.

Two documentation files are regenerated to list the new support: `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/heading/README.md`.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @shail-mehta. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The discussion thread contains only two automated comments from `github-actions[bot]` (co-author attribution and PR performance/flaky-test metadata); no human review comments or design debate are visible in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
