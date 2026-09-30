# #83192: Term Template: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`fbc587a`](https://github.com/WordPress/gutenberg/commit/fbc587a471d63edaf82976fecbe40af516420c48)
- **Discussion:** [#83192](https://github.com/WordPress/gutenberg/pull/83192) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Term Template block (`core/term-template`) now supports the `shadow` block support, allowing a box-shadow to be applied to the entire term list via Global Styles or the block inspector. This closes a gap in the block's design-tool coverage — it previously supported color, border, layout, spacing, and typography but had no shadow control. The change is a single `"shadow": true` entry in the block's `block.json`; no PHP or JS logic was added because the block is server-rendered through `get_block_wrapper_attributes()`, which already serializes shadow CSS custom properties onto the wrapper element.

## Impact

- **Block theme developers:** The Term Template block (used inside `core/terms-query`) now exposes a Shadow control in the Site Editor's Global Styles panel and in the block inspector. No code changes are required to take advantage of it; it works out of the box in any theme that uses the block.
- **Plugin developers:** No action required. No new hooks, filters, or REST schema changes are introduced.
- **Site owners / editors:** Can now pick a shadow preset (Natural, Deep, Sharp, Crisp, etc.) for the term list in Global Styles or per-block, including responsive variants.
- **No breaking changes, deprecations, or migrations.**

## Technical details

The diff touches three files, all of which are declarative or generated:

1. **`packages/block-library/src/term-template/block.json`** — the functional change. A single key is added to the existing `supports` object:

```json
"supports": {
  "align": ["full", "wide"],
  "anchor": true,
  "color": true,
  "width": true,
  "style": true,
  "shadow": true
}
```

2. **`packages/block-library/src/term-template/README.md`** — regenerated to list the new `shadow` support under the block's supports table.

3. **`docs/reference-guides/core-blocks/README.md`** — the core-blocks reference is regenerated; the Term Template entry's supports line now includes `shadow`.

No PHP, JS, or CSS files are modified. The block's server-side render path calls `get_block_wrapper_attributes()` on the `<ul>` wrapper, which already emits `--wp--custom--shadow` and the corresponding `box-shadow` declaration when the support is active. The shadow applies to the whole list container, not to individual `<li>` items.

## Contribution

Opened by @aaronrobertshaw as part of the broader design-tools consistency effort tracked in #43241. The PR notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. @im3dabasia is listed as a co-author. The record carries no substantive design debate or alternative-approach discussion — the two comments are the automated PR-meta performance report and the co-author attribution bot.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
