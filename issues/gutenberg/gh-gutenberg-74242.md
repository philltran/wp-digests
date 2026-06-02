# #74242: Button: Migrate to width block support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Buttons`, `[Package] Block editor`, `[Feature] Design Tools`, `No Core Sync Required`
- **Merged:** [`5f647a0`](https://github.com/WordPress/gutenberg/commit/5f647a04861d35774c3863efc96dcd9144c82394)
- **Discussion:** [#74242](https://github.com/WordPress/gutenberg/pull/74242) · 35 comments · 0 reactions

## Summary

This PR migrates the core Button block away from its bespoke `width` attribute to the standard `dimensions.width` block support. Instead of a four-option toggle group (25/50/75/100%), the width control now renders in the Dimensions panel as a segmented slider backed by dimension presets, and accepts any value/unit (e.g. `150px`, `50vw`). Crucially, button width becomes addressable via theme.json and Global Styles, so themes and users can set a default button width globally and override it per instance. Existing buttons are migrated automatically through the block's deprecation chain.

## Impact

**Site owners / content editors**
- The width UI changes from preset toggle buttons to a dimension control (segmented slider) that also allows arbitrary values and units. Existing posts are migrated transparently — no manual editing needed.
- Button width can now be set globally in **Global Styles → Blocks → Button** and overridden on individual buttons.

**Theme developers**
- You can now define a default button width and dimension presets in `theme.json`, including overriding presets at `settings.blocks.core/button.dimensions.dimensionSizes`.
- Default 25/50/75/100 dimension sizes for the Button block are seeded in `lib/theme.json`.

**Plugin developers**
- The ad-hoc `width` attribute is removed from the Button `block.json`; width now lives in `style.dimensions.width`. Code that reads or writes the old numeric `width` attribute directly should move to the `dimensions.width` block support shape. Saved markup still emits the legacy `wp-block-button__width-*` classes for percentage values, so CSS targeting those classes continues to work.

**General**
- Labeled `No Core Sync Required`; this is a Gutenberg-side change. No action required for most sites — migration is automatic.

## Technical details

The change reworks the Button block definition and supports wiring:

- **`block.json`**: removes the custom `width` attribute and adds `dimensions.width` support with default controls enabled. Selectors are declared so that dimensions global styles and preset CSS custom properties target the outer `wp-block-button` wrapper rather than the inner `<a>` link element.
- **Editor**: the custom `WidthPanel` component is removed; the block now uses the shared `DimensionsPanel` provided by block supports.
- **Percentage tiling**: when a percentage width is applied via Global Styles or theme.json (as a direct value or a resolved dimension preset), both the PHP renderer and the JS style engine convert it into a `calc()` formula that subtracts a proportional share of the block gap — mirroring the existing block-instance behavior in `style.scss` — so a row of four 25% buttons tiles without overflowing.
- **Deprecations**: a new v14 deprecation migrates numeric `width` values to the `style.dimensions.width` shape, and all 14 prior deprecations were updated to include the width migration in their chain. Legacy `wp-block-button__width-25/50/75/100` classes are preserved for percentage widths for backward-compatible CSS.

Usage shift:

```js
// Before — ad-hoc attribute
{ "attributes": { "width": 25 } }

// After — dimensions.width block support
{ "style": { "dimensions": { "width": "25%" } } }
```

Bundle impact reported at +2.08 kB total, including ~+0.19% to `block-library` and small additions to the Button stylesheet. Depends on #75226.

## Contribution

Authored by **@aaronrobertshaw** and merged in commit `5f647a0`. Credited co-authors include **@SirLouen**, **@andrewserong**, **@ramonjd**, **@dmsnell**, and **@ciampo**. The PR drew 35 comments; a recurring design concern was preserving correct multi-button tiling for percentage widths, resolved by generating a gap-aware `calc()` value in both the PHP renderer and the JS style engine to match the prior instance-level CSS behavior. The work depends on #75226.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
