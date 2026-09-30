# #83056: Page List: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Page List`
- **Merged:** [`790ad67`](https://github.com/WordPress/gutenberg/commit/790ad676c9d0758a1e50dbf8919afc9aee9207b0)
- **Discussion:** [#83056](https://github.com/WordPress/gutenberg/pull/83056) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Page List block (`core/page-list`) now supports the `shadow` block support, allowing shadows to be applied via Global Styles or the block inspector. This is a one-line addition to the block's `block.json` supports object; because the block is server-rendered through `get_block_wrapper_attributes()`, the support is serialized onto the wrapper automatically with no PHP template change. The change closes a gap in the design tools consistency effort (related #43241), where Page List already supported color, border, spacing, and typography but not shadow.

## Impact

- **Site owners / block theme authors:** Can now set a shadow on the Page List block in Global Styles (Blocks → Page List → Shadow) or per-instance in the block inspector. No code changes required.
- **Plugin & theme developers:** No action required. The support is handled entirely by the existing `get_block_wrapper_attributes()` serialization path. No new hooks, filters, or template changes.
- **Navigation block fallback:** A Navigation block with no menu items renders as a Page List, so a Global Styles shadow applied to Page List will also appear on that fallback (e.g. in a theme header). Worth noting if you rely on the Navigation fallback and have Page List shadow styles set.
- **No breaking changes, deprecations, or migrations.**

## Technical details

The diff adds a single key to the `supports` object in `packages/block-library/src/page-list/block.json`:

```json
"supports": {
  // …existing color, spacing, typography, etc.…
  "shadow": true,
  "contentRole": true
}
```

Because `core/page-list` is server-rendered via `get_block_wrapper_attributes()`, the shadow support is automatically serialized as a CSS custom property on the list wrapper element. No change to the block's `render.php` or any PHP template is needed.

Two documentation files are regenerated to list the new support:
- `docs/reference-guides/core-blocks/README.md` — `shadow` added to the Page List supports line.
- `packages/block-library/src/page-list/README.md` — a new bullet under the supports section: `[shadow](…): true`.

## Contribution

Opened by @aaronrobertshaw and reviewed by @shail-mehta, who is credited as co-author. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. It is part of the broader design-tools consistency work tracked in #43241. Merged at `790ad67` with no notable design debate in the three comments on record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
