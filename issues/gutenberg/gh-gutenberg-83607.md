# #83607: Home Link: Add padding and margin support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Home Link`
- **Merged:** [`afb3f67`](https://github.com/WordPress/gutenberg/commit/afb3f672f7a6873fe003f73513d543a7c2e8c13e)
- **Discussion:** [#83607](https://github.com/WordPress/gutenberg/pull/83607) · 3 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The Home Link block (`core/home-link`) now supports `spacing` (margin and padding), closing a gap with its sibling Custom Link block. The change is entirely declarative: a new `spacing` entry in the block's `block.json` is sufficient because the block is server-rendered and builds its `<li>` wrapper via `get_block_wrapper_attributes()`, which serializes spacing supports automatically. No PHP or JavaScript changes were required. This is part of the broader design-tools consistency work tracked in #43241.

## Impact

- **Site owners (block themes):** Padding and margin can now be set on the Home Link item via Global Styles → Blocks → Home Link → Dimensions, with independent mobile values. No action required; the controls appear automatically in the Site Editor.
- **Plugin & theme developers:** No code changes needed. The support is purely declarative in `block.json`. If you build custom Navigation patterns or child blocks, be aware that Home Link now emits `margin`/`padding` CSS custom properties through `get_block_wrapper_attributes()` when set.
- **Block inspector note:** The `__experimentalDefaultControls` entry sets both `margin` and `padding` to `false`, so the Dimensions panel does not show these controls by default in the block inspector. They remain settable via Global Styles.
- **Editor margin caveat (pre-existing):** A Global Styles margin on a Navigation child does not move the item in the editor canvas because the Navigation block's editor stylesheet resets margins on `.wp-block-navigation-item.wp-block` (since #37587). The margin does apply on the front end. This affects every Navigation child with margin support, not just Home Link.

## Technical details

The sole functional change is in `packages/block-library/src/home-link/block.json`, which adds:

```json
"spacing": {
    "margin": true,
    "padding": true,
    "__experimentalDefaultControls": {
        "margin": false,
        "padding": false
    }
}
```

Because the block's render callback calls `get_block_wrapper_attributes()` to build the `<li>` wrapper, the new spacing support is serialized into the output automatically (as inline CSS custom properties and the corresponding `:where()` rules) with no PHP modification. The render callback always emits a link with a label (falling back to "Home"), so there is no empty-wrapper edge case for padding.

Two documentation files are regenerated to list the new support:
- `docs/reference-guides/core-blocks/README.md` — the Home Link `Supports` line gains `spacing (margin, padding)`.
- `packages/block-library/src/home-link/README.md` — a new `spacing` section with `margin: true` and `padding: true` is inserted.

No new hooks, REST schema fields, or DB changes are introduced.

## Contribution

Authored by @aaronrobertshaw with co-authorship from @shail-mehta. The PR was implemented, built, and screenshotted by a Claude Code agent from a predefined task. @mrleemon reviewed and merged it, thanking the author and requesting that similar margin/padding support be extended to the Navigation, Navigation Link, and Navigation Submenu blocks for WP 7.2. The PR is related to the broader design-tools consistency tracking issue #43241.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
