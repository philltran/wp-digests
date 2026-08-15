# #77750: Block Editor: Fix blockGap fallback handling for nested var() fallback values

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mustafabharmal
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `Global Styles`, `[Feature] Layout`, `Backported to WP Core`
- **Merged:** [`8dd0b92`](https://github.com/WordPress/gutenberg/commit/8dd0b92513885b7d337f95f3f82ab6cdf1de1b4c)
- **Discussion:** [#77750](https://github.com/WordPress/gutenberg/pull/77750) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Layout gap fallback resolution in the block editor no longer splits a processed gap string on whitespace, which produced malformed CSS when `styles.spacing.blockGap` in `theme.json` used nested `var()` fallbacks such as `var(--a, var(--b, 1.2rem))`. Splitting on spaces treated the whitespace *inside* the nested `var()` expression as a row/column separator, so editing row-gap or column-gap on a Columns (flex) or Grid block emitted a gap value with unbalanced parentheses. The flex and grid layout definitions now read the row/column values structurally via `getGapBoxControlValueFromStyle()` and resolve them through `getSpacingPresetCssVar()`. Fixes Gutenberg issue #76896.

## Impact

- **Theme developers:** If your `theme.json` sets `styles.spacing.blockGap` to a value containing a nested `var()` fallback (e.g. `"var(--wp--custom--gap, var(--fallback, 1.2rem))"`), the invalid gap CSS generated after a user tweaks row/column gap on flex or grid layout blocks is fixed. No `theme.json` change is required — the previously broken value shape is now handled correctly.
- **Block/plugin developers:** No public API change. `getGapCSSValue()` is unchanged and still exported; only the internal fallback-derivation path inside `packages/block-editor/src/layouts/flex.js` and `grid.js` was rewritten. Blocks whose `block.json` skips serialization for `spacing.blockGap` are the ones that exercise this fallback path.
- **Site owners:** No action required beyond updating Gutenberg/WordPress; affected sites should see gap values render correctly again without editing content.
- The hardcoded defaults are unchanged: `0.5em` for flex layout, `1.2rem` for grid layout.

## Technical details

Both `packages/block-editor/src/layouts/flex.js` and `packages/block-editor/src/layouts/grid.js` computed `fallbackGapValue` from the *global* blockGap value by running it through `getGapCSSValue()` and then string-splitting the result on a space, assuming a two-value `"<row> <column>"` shorthand and taking the second token as the column value. That assumption breaks for any single value containing internal whitespace — precisely what a nested `var(--a, var(--b, 1.2rem))` fallback is — yielding fragments like `var(--b,` and unbalanced parentheses in the emitted rule.

The fix imports `getGapBoxControlValueFromStyle` from `../hooks/gap` (alongside the existing `getGapCSSValue`) and `getSpacingPresetCssVar` from `../components/spacing-sizes-control/utils`, then parses the raw style value into a box object and reads `left` (column) with `top` (row) as a secondary fallback:

```js
// Before (grid.js)
let fallbackGapValue = '1.2rem';
if ( globalBlockGapValue ) {
	const processedGap = getGapCSSValue( globalBlockGapValue, '0.5em' );
	const gapParts = processedGap.split( ' ' );
	fallbackGapValue =
		gapParts.length > 1 ? gapParts[ 1 ] : gapParts[ 0 ];
}

// After (grid.js)
let fallbackGapValue = '1.2rem';
if ( globalBlockGapValue ) {
	const gapBox = getGapBoxControlValueFromStyle( globalBlockGapValue );
	fallbackGapValue =
		getSpacingPresetCssVar( gapBox?.left ) ||
		getSpacingPresetCssVar( gapBox?.top ) ||
		'1.2rem';
}
```

`flex.js` receives the identical treatment with `0.5em` as the terminal default. Preset handling is preserved because `getSpacingPresetCssVar()` still converts `var:preset|spacing|xx` style values into their `var( --wp--preset--spacing--xx )` form — the change is that resolution happens per-axis on structured values rather than by re-parsing a serialized string. No hooks, `block.json` fields, REST schema, or stored data change; this is purely CSS generation in the layout definitions.

To reproduce the original bug: set `styles.spacing.blockGap` to `"var(--a, var(--b, 1.2rem))"`, insert a Columns block, then change only column-gap (and then row-gap) in block settings and inspect the generated rule for balanced parentheses.

## Contribution

Opened by @Mustafabharmal as a targeted fix for issue #76896, with props-bot crediting @karthikeya-io, @Swanand01, and @inc2734 for work on the PR and linked issues. After merge, @t-hamano flagged it as a candidate for the 7.0 minor release (citing discussion on #78743), @cbravobernal cherry-picked it to the `wp/7.0` branch as `e1577c0e6f5`, and @t-hamano confirmed it landed in 7.0.1. No alternative approaches were debated in the thread — the author explicitly framed the change as minimal and scoped to fallback resolution, reusing existing gap helpers rather than hardening the string parsing.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
