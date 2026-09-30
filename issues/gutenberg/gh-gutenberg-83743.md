# #83743: Pagination: Add padding, margin, and block gap support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Query Pagination`
- **Merged:** [`80415d9`](https://github.com/WordPress/gutenberg/commit/80415d943739bbd68e6b030819c466f571d3af52)
- **Discussion:** [#83743](https://github.com/WordPress/gutenberg/pull/83743) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Query Pagination block (`core/query-pagination`) now supports padding, margin, and block gap through the standard `spacing` block supports. Previously, the space between the Previous, Numbers, and Next links could only be influenced by the theme's global block gap, with no per-block override. This brings the block in line with other flex-container blocks (Buttons, Social Icons, Navigation) that already expose these controls.

## Impact

- **Site builders / theme authors:** Padding, margin, and block gap can now be set on the Pagination block via Global Styles (with mobile/desktop states) or the block inspector's Dimensions panel. No code changes required.
- **Plugin & theme developers:** No action required. The block is server-rendered through `get_block_wrapper_attributes()`, so the new supports serialize into the wrapper's inline styles automatically. No PHP, JS, or CSS changes are needed in themes or plugins.
- **No breaking changes.** Existing Pagination blocks render identically when no spacing values are set.

## Technical details

Two files carry the functional change:

**`packages/block-library/src/query-pagination/block.json`** — a new `spacing` key is added to the `supports` object:

```json
"spacing": {
    "padding": true,
    "margin": true,
    "blockGap": true
}
```

**`packages/block-library/src/query-pagination/style.scss`** — `box-sizing: border-box` is added to `.wp-block-query-pagination` so that padding does not expand the block's total width/height beyond its layout allocation:

```scss
.wp-block-query-pagination {
    // This block has customizable padding, border-box makes that more predictable.
    box-sizing: border-box;
    // …existing flex rules unchanged
}
```

No PHP changes are made. The render callback calls `get_block_wrapper_attributes()`, which picks up the new supports and emits the corresponding inline `style` attribute. The callback also returns an empty string when there is nothing to paginate, so no empty wrapper element is produced for padding to paint on.

Margin is deliberately **not** added to the three child blocks (`core/query-pagination-previous`, `core/query-pagination-numbers`, `core/query-pagination-next`). The flex layout resets child margins, so a Global Styles margin on them would have no effect, and a block-level margin would override the `margin-inline-start: auto` that the "Space between" justification relies on.

The core-blocks reference (`docs/reference-guides/core-blocks/README.md`) and the block's own `README.md` are regenerated to list the new `spacing` supports. Bundle size impact is +101 B total.

## Contribution

The PR supersedes an earlier attempt (#66398) by @rinkalpagdar. It was co-authored by @aaronrobertshaw and @andrewserong, with only 3 comments and no reactions in the discussion. The author noted the work was "implemented, built and screenshotted by a Claude Code agent from a predefined task" and reflected that the block "should have" had these supports already, framing it as part of a broader design-tool consistency effort rather than a standalone feature.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
