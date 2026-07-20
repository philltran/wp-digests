# #76949: Block Supports: Add min width support to dimensions

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Blocks`, `[Package] Block library`, `[Package] Block editor`, `[Feature] Design Tools`, `[Package] Style Engine`, `has dev note`
- **Merged:** [`d701f12`](https://github.com/WordPress/gutenberg/commit/d701f12d6f97d560043397444296ffd04930f564)
- **Discussion:** [#76949](https://github.com/WordPress/gutenberg/pull/76949) · 10 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

Adds `minWidth` as a new dimension block support, completing parity with the existing `minHeight` support. This enhancement allows blocks to enforce a minimum width constraint, preventing containers and layout elements from collapsing below a usable size on smaller viewports. It integrates across the Style Engine, block supports system, block editor UI, and `theme.json` schema.

## Impact

- **Theme developers:** Can now define `minWidth` values in `theme.json` under `settings.dimensions` or `styles.*.dimensions`. The property is added to the valid styles and appearance tools feature list.
- **Block developers:** Can opt into `minWidth` support for custom blocks via the block supports system. The control is hidden by default in the block inspector and must be enabled via `__experimentalDefaultControls`.
- **Editors & Site Owners:** Can set minimum width values for supported blocks (e.g., Group) in the block inspector or global styles panel.
- **No action required** for existing sites or plugins; this is a backward-compatible enhancement.

## Technical details

The implementation mirrors the existing `minHeight` support across all layers:
- **Style Engine & PHP:** Introduces a `minWidth` CSS rule generator and maps it to the `min-width` property, supporting `dimensionSizes` presets via CSS variables.
- **Block Supports:** Registers `minWidth` as a dimension feature and adds it to the `__EXPERIMENTAL_STYLE_PROPERTY` constants.
- **Block Editor UI:** Adds a `DimensionControl` for `minWidth` to the dimensions panel. Settings filtering is wired up in `useSettingsForBlockElement` and `useSettings`. The control is **not** visible by default in the post editor and must be opted into via `__experimentalDefaultControls`, though it appears by default in the global styles panel.
- **Schemas & Theme JSON:** Updates `theme.json` and `block.json` schemas to validate `minWidth` in settings and styles.
- **Block Defaults:** Enables `minWidth` support on the core Group block.
- **Linting:** Renames internal `useHas*` helper functions in the dimensions panel to drop the `use` prefix to avoid `react-hooks/rules-of-hooks` false positives, and reorders a `useCallback` call to satisfy hook rules.

## Contribution

Opened and merged by @aaronrobertshaw, this enhancement addresses a gap in the dimensions API (related to #76525). The implementation mirrors the established `minHeight` pattern across JS and PHP layers. Review focused on ensuring consistent preset handling via `dimensionSizes` and correcting React hook linting violations in the dimensions panel before merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
