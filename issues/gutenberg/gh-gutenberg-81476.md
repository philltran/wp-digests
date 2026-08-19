# #81476: Enable axial gap for Group and restrict to flex and grid layouts.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Package] Block editor`, `[Feature] Layout`
- **Merged:** [`4061b86`](https://github.com/WordPress/gutenberg/commit/4061b86a009cb95477dffbc4f6e92c0fcd61a620)
- **Discussion:** [#81476](https://github.com/WordPress/gutenberg/pull/81476) · 9 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Group block can now take separate row and column values for `blockGap`, but the split (axial) control is only rendered when the block instance resolves to a `flex` or `grid` layout — flow and constrained Groups keep the single input, since those layouts only ever set `margin-block-start` on children. A new helper, `isAxialBlockGapAllowed()`, makes that decision, and the shared global-styles `DimensionsPanel` gained an `allowAxialBlockGap` prop (defaulting to `true`, so Global Styles keeps axial controls for the Group block regardless of variation). The PR also corrects two server-side bugs: `WP_Theme_JSON` previously dropped an axial `blockGap` entirely unless *both* `top` and `left` were set, and flow/constrained layouts were writing the full `"<row> <column>"` shorthand into `margin-block-start`. Responsive grid column math now reads the horizontal (`left`) value instead of the concatenated shorthand.

## Impact

**Block/theme developers**
- Group blocks in Row, Stack, and Grid variations now expose split top/bottom and left/right block-spacing controls in the inspector; flow/constrained Groups do not. Existing content is unaffected — axial gap was previously only reachable from flex layouts.
- `theme.json` authors: a `spacing.blockGap` object with only `top` **or** only `left` now produces output (the missing axis falls back to `0`). Previously such a partial value was silently discarded, so any theme relying on that silence to suppress a gap will start emitting one.
- Themes that set an axial `blockGap` on a block whose layout is `default` (flow) or `constrained` will now get only the `top` value applied to `margin-block-start`, rather than the two-value shorthand.
- Blocks other than Group that already support axial gap (Gallery, Buttons) were regression-tested and are unchanged. The Navigation block was checked and does **not** pick this up.

**Plugin developers consuming `@wordpress/block-editor`**
- `packages/block-editor/src/hooks/gap.js` exports a new `isAxialBlockGapAllowed( layout, defaultLayout )`; the internal global-styles `DimensionsPanel` accepts a new `allowAxialBlockGap` prop.

**Everyone else:** no action required. The change is queued for backport to Core via `backport-changelog/7.2/13011.md`.

## Technical details

`packages/block-editor/src/hooks/gap.js` adds:

```js
export function isAxialBlockGapAllowed( layout, defaultLayout ) {
	const usedLayout =
		layout?.inherit || layout?.contentSize || layout?.wideSize
			? { ...layout, type: 'constrained' }
			: layout || defaultLayout || {};

	return [ 'flex', 'grid' ].includes( usedLayout?.type );
}
```

Note the normalisation: a layout carrying `inherit`, `contentSize`, or `wideSize` is treated as `constrained` even without an explicit `type`, and a block with no `layout` attribute falls back to its `layout` support `default`.

`packages/block-editor/src/hooks/dimensions.js` now pulls `attributes.layout` in its `useSelect` and reads `getBlockSupport( name, 'layout' ).default`, passing the result of `isAxialBlockGapAllowed()` down as `allowAxialBlockGap`.

In `components/global-styles/dimensions-panel.js`, `isAxialGap` is gated on the new prop. When axial is disallowed but the stored value is an object, the single `UnitControl`/placeholder path now reads `.top` via new `localGapForSingleInput` / `inheritedGapForSingleInput` values instead of handing a non-string object to a control that cannot render it. `SpacingSizesControl` gets `key={ isAxialGap ? 'axial-gap' : 'single-gap' }` so it remounts when axial support toggles — covered by a new test asserting the control stays visible (2 sliders → 1) across that transition.

Server side, `WP_Theme_JSON_Gutenberg::get_layout_styles()` tracks `$block_gap_row_value` alongside `$block_gap_value`:

```php
// Before: both sides required, otherwise nothing is emitted.
if ( isset( $block_gap_value['top'] ) && isset( $block_gap_value['left'] ) ) { ... }
else { $block_gap_value = null; }

// After: either axis is enough; the missing one becomes '0'.
if ( $has_block_gap_row_value || $has_block_gap_column_value ) {
	$block_gap_row_value    = $has_block_gap_row_value ? ... : '0';
	$block_gap_column_value = $has_block_gap_column_value ? ... : '0';
	$block_gap_value = $block_gap_row_value === $block_gap_column_value
		? $block_gap_row_value
		: $block_gap_row_value . ' ' . $block_gap_column_value;
}
```

When substituting into `spacingStyles` rules, layout definitions keyed `default` and `constrained` now receive `$block_gap_row_value`; every other layout gets the shorthand `$block_gap_value`. Matching this, `layouts/flow.js` and `layouts/constrained.js` no longer run `style?.spacing?.blockGap` through `getGapCSSValue()` before use.

In `lib/block-supports/layout.php`, `gutenberg_get_layout_style()` sets `$responsive_gap_value` from the `left` side of an axial gap during the side loop rather than assigning it the trimmed two-value shorthand afterwards — the shorthand was invalid inside the responsive-grid `calc()`. The adjacent comment describing `left` as "the vertical gap" was corrected to "horizontal".

## Contribution

Opened by **@tellthemachines** to close the long-standing issue #47084, with review and manual testing from **@ramonjd** (who confirmed Gallery and Buttons were unregressed and flagged confusion between global-styles and per-instance `wp-container-core-group-is-layout-*` overrides, later attributed to a pre-existing trunk bug with section-style gap output) and **@andrewserong**. Review surfaced a real bug — both axes were being written into `margin-block-start` for flow/constrained layouts — which the author fixed in a follow-up commit before merge. **@talldan** suggested extending the same treatment to the Navigation block; it was acknowledged as a candidate but left out of scope. @andrewserong also noted, without a diagnosis, that toggling a Group → Row → Group via the variation switcher left the Block Spacing controls rendering empty.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
