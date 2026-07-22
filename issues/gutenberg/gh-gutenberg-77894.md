# #77894: Block Editor: Reflect inherited Global Styles values in block inspector controls

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `Global Styles`, `[Feature] Design Tools`, `Backported to WP Core`
- **Merged:** [`cd89b1a`](https://github.com/WordPress/gutenberg/commit/cd89b1aa523677313d7e27ea95aa5d5be5522dff)
- **Discussion:** [#77894](https://github.com/WordPress/gutenberg/pull/77894) · 90 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The block editor inspector controls now display inherited Global Styles values when a block has no local override, resolving a long-standing UX gap where controls appeared empty despite the canvas showing styled content. Inherited values from theme.json, user edits, block types, or style variations are rendered as placeholders or preselected options, with a dotted underline on the label to indicate inheritance. When a local override is applied, a blue dot appears on the label that expands into a Reset to inherited value button on hover or focus.

## Impact

- **Plugin & theme developers:** No breaking changes or required code updates. The change is purely presentational within the block editor UI.
- **Site owners & editors:** Improved clarity in the inspector when blocks inherit styles from Global Styles, with a single-click reset to revert local overrides.
- **Hosting & platform teams:** No action required. The change ships in the block editor package.
- **Headless & REST consumers:** No impact; this is a client-side editor UI enhancement with no REST API or data structure changes.

## Technical details

The diff introduces inheritance-aware rendering across standard block-support panels. Key internal symbols include `useInheritedValue`, `buildInheritedValue`, `InheritanceToolsPanelItem`, `InheritanceResetButton`, and `getInheritanceProps`. Block support hooks now read merged Global Styles data via a single `useSelect`, resolve the Root › Block › Variation cascade, and pass the result to panels as `inheritedValue`. Panels accept a new optional prop `showInheritanceLabelIndicators` (default `true`) to toggle the dotted underline and override dot. The `BackgroundImagePanel` and `AspectRatioTool` were updated to conditionally render `InheritanceToolsPanelItem` and wire the reset affordance. `DimensionControl` gained `placeholder` and `className` props to support inherited value display. No public APIs were exposed; all new symbols remain internal to `@wordpress/block-editor`.

**Before (background size resolution):**
```js
const sizeValue =
	style?.background?.backgroundSize ||
	inheritedValue?.background?.backgroundSize;
```
**After (separated local/inherited reads for UI state tracking):**
```js
const localSizeValue = style?.background?.backgroundSize;
const inheritedSizeValue = inheritedValue?.background?.backgroundSize;
const sizeValue = localSizeValue || inheritedSizeValue;
```

## Contribution

The PR evolved through extensive review focusing on UX clarity and architectural boundaries. Reviewers pushed for a focused MVP, leading to the deliberate deferral of portalled label adornments, inheritance path tooltips, and the push block style to Global Styles feature to follow-ups. A notable design debate centered on using purple text versus a subtle dotted underline to mark inherited values; the team settled on the underline-only approach for better accessibility and reduced visual noise. The implementation builds on a prior color control relocation and was scoped to land without extending @wordpress/components.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
