# #81601: Fix viewport state values set by grid resizer

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `[Feature] Layout`, `[Feature] Style States`
- **Merged:** [`aa50808`](https://github.com/WordPress/gutenberg/commit/aa50808db60acba7c51eef8e738cc7c61937ac05)
- **Discussion:** [#81601](https://github.com/WordPress/gutenberg/pull/81601) · 3 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

Fixes a bug in the Grid block's drag-handle resizer where setting a column or row span while a viewport was selected, and no default (desktop) span existed, wrote the value to the base `style.layout` instead of the viewport's style state. The span was then output without a media query and applied to all viewports. Resizer changes are now stored under the selected viewport and emitted inside the matching media query, consistent with the inspector controls.

## Impact

- **Site owners / editors:** Anyone using the Grid resizer handles with responsive styles enabled on a child that had no desktop span could get a viewport-specific span that leaked to desktop on the front end and in the editor. After this fix the value stays scoped to the selected viewport. Children already saved with the leaked values are not migrated; the stored attributes would need to be corrected manually (for example, by resetting the span on desktop).
- **Plugin & theme developers:** No API, hook, or block.json changes. The new `getUpdatedChildLayoutStyle` is exported from `layout-child.js` only for testing inside the package. No action required.
- **Release note:** The PR author describes this as a 7.1 regression and considers it safe to defer to 7.1.1. The easy workaround is to use the inspector span controls instead of the resizer.

## Technical details

All changes are in `packages/block-editor/src/hooks/layout-child.js`.

- **New helper `getUpdatedChildLayoutStyle( style, layout, selectedState )`.** If `hasViewportBlockStyleState( selectedState )` is false, it keeps the old behavior and merges `layout` into `style.layout`. Otherwise it builds a layout state `{ viewport: selectedState.viewport, pseudo: DEFAULT_BLOCK_STYLE_STATE.pseudo }`, reads the existing style with `getStyleForState`, merges the new `layout` into that state's `layout`, and writes it back with `setStyleForState`. The pseudo state is deliberately forced to the default, so a selected pseudo state such as `:hover` does not affect where layout is stored.
- **`GridTools`.** The `useSelect` now reads `getSelectedBlockStyleState` from `unlock( select( blockEditorStore ) )` and returns `selectedState: getSelectedBlockStyleState( clientId )`. `updateLayout` now calls `setAttributes( { style: getUpdatedChildLayoutStyle( style, layout, selectedState ) } )` instead of always spreading into `style.layout`.

```js
// Before
setAttributes( { style: { ...style, layout: { ...style?.layout, ...layout } } } );
// After
setAttributes( { style: getUpdatedChildLayoutStyle( style, layout, selectedState ) } );
```

New unit tests in `hooks/test/layout-child.js` cover two cases. With `@mobile` selected and no default layout, `getChildLayoutStyles` returns `''` and `getResponsiveChildLayoutStyles` returns the `@media (width <= 480px)` rule containing `grid-column: span 2; grid-row: span 1;`. With `@mobile` plus `:hover` selected, the result is `{ '@mobile': { layout: { columnSpan: 2, rowSpan: 1 } } }`. A CHANGELOG entry was added under Bug Fixes for `@wordpress/block-editor`.

Per a review comment, the experimental manual Grid placement `GridItemMovers` still use the base `layout` object rather than the viewport-specific one. This PR does not touch them.

## Contribution

Found while reviewing #81560. The author judged it a niche 7.1 regression with an easy workaround, so it was slated for 7.1.1 rather than the imminent 7.1 release. A reviewer (apparently @andrewserong, per the props list) noted that the experimental manual grid placement movers still use the base layout. The author replied that the experiment is already broken overall and needs broader work if it is revisited, so that is out of scope here. The PR disclosed that AI tooling (Codex) was used.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
