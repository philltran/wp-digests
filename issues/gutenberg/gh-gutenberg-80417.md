# #80417: Components: Align legacy form control focus rings with WPDS

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] DataViews`
- **Merged:** [`2b71453`](https://github.com/WordPress/gutenberg/commit/2b714533eda001f180733f979bdc13202bda1114)
- **Discussion:** [#80417](https://github.com/WordPress/gutenberg/pull/80417) · 12 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The shared focus treatment for legacy `InputBase`-derived form controls (`InputControl`, `SelectControl`, `CustomSelectControl` and everything built on them) moves from an Emotion-generated border + box-shadow combination to the WPDS outset focus ring, so `@wordpress/components` controls visually match their `@wordpress/ui` replacements ahead of consumer migration. The old style — a darkened 1px border plus a 0.5px box-shadow to fake a 1.5px stroke, with a transparent `outline` as a Windows High Contrast Mode fallback — is replaced by a single `outset-ring__focus()` mixin call in package Sass that uses the `--wpds-border-width-focus` and `--wpds-color-stroke-focus` tokens. Because the new ring paints outside the control, DataViews' filter operator `SelectControl` had to stop clipping it: `overflow: hidden` becomes `min-width: 0`. Validated (invalid-state) controls also switch the custom property they override to recolor the focus ring, from `--wp-components-color-accent` to `--focus-color`.

## Impact

**Plugin & theme developers using `@wordpress/components`**
- Focus rings on `InputControl`, `SelectControl`, `CustomSelectControl` and all composites built on them (`NumberControl`, `SearchControl`, `UnitControl`, `RangeControl`, `AnglePickerControl`, `TreeSelect`, `ColorPicker`, `ColorPalette`, `CustomGradientPicker`, `GradientPicker`, `TimePicker`, `DateTimePicker`, `PaletteEdit`, `FontSizePicker`, `BorderControl`, `QueryControls`, and the `Validated*` variants) now render as an outset outline rather than a thickened border + shadow. No action required if you use the default styling.
- **If you override the old focus treatment**, re-check it. Overrides that neutralise or restyle `box-shadow`/`border-color` on `.components-input-control__backdrop`, or that rely on the specificity of the Emotion-generated `Root` class, will no longer control the visible ring — the ring is now an outline delivered from the components stylesheet.
- **Focus styling now depends on the stylesheet being loaded.** The rule left the CSS-in-JS bundle (`input-control-styles.tsx`) for `packages/components/src/input-control/style.scss`, which is aggregated into `src/style.scss`. Contexts that load the components JS without the built `wp-components` CSS will not get a focus ring. The build-size report shows the shift: components JS −69 B, components CSS +65–70 B.
- **Recolouring the invalid-state focus ring changed variable**: setting `--wp-components-color-accent` on `.components-input-control__backdrop` no longer drives the focus colour; the ring reads `--focus-color`. The error `border-color: $alert-red` is unchanged.

**DataViews consumers**
- `.dataviews-filters__summary-operators-filter-select` no longer sets `overflow: hidden`; it uses `min-width: 0` to keep flex sizing while letting the ring paint outside. Any CSS relying on that element clipping its children is affected.

**Accessibility / forced colors**
- The `outline: 2px solid transparent; outline-offset: -2px;` High Contrast Mode shim (added in #50772 because box-shadows are invisible under forced colors) is removed — the real outline now provides forced-colors visibility directly.

**Site owners:** no action required; this is a visual change only.

## Technical details

The focus rule is deleted from the Emotion `Root` styled component in `packages/components/src/input-control/styles/input-control-styles.tsx` and re-implemented as Sass in a new `packages/components/src/input-control/style.scss`, registered via `@use "./input-control/style.scss" as *;` in `packages/components/src/style.scss`.

Before (Emotion, in `input-control-styles.tsx`, interpolating the `Prefix`/`Suffix`/`BackdropUI` styled components):

```js
export const Root = styled( Flex )`
	// Focus within, excluding cases where auxiliary controls in prefix or suffix have focus.
	&:focus-within:not( :has( :is( ${ Prefix }, ${ Suffix } ):focus-within ) ) {
		${ BackdropUI } {
			border-color: ${ COLORS.ui.borderFocus };
			box-shadow: ${ CONFIG.controlBoxShadowFocus };
			// Windows High Contrast mode will show this outline, but not the box-shadow.
			outline: 2px solid transparent;
			outline-offset: -2px;
		}
	}
`;
```

After (`input-control/style.scss`, keyed to the public class names rather than Emotion-generated ones):

```scss
@use "@wordpress/base-styles/mixins";

.components-input-control__container {
	// Don't show the input's focus ring when an auxiliary control in a prefix
	// or suffix has focus.
	&:focus-within:not(:has(:is(.components-input-control__prefix, .components-input-control__suffix):focus-within)) {
		.components-input-control__backdrop {
			@include mixins.outset-ring__focus();
		}
	}
}
```

The prefix/suffix suppression semantics are preserved verbatim — focusing an interactive prefix/suffix (including the `UnitControl` unit selector) still cancels the surrounding ring. Per the PR description, `outset-ring__focus()` is the same mixin `@wordpress/ui` uses, resolving `--wpds-border-width-focus` and `--wpds-color-stroke-focus` with generated fallbacks.

Validated controls: both `packages/components/src/validated-form-controls/style.scss` and `packages/dataviews/src/components/validated-form-controls/style.scss` introduce a `%red-focus-ring` placeholder selector that sets `--focus-color: #{$alert-red}`, `@extend`-ed in place of the previous `--wp-components-color-accent: #{$alert-red}` declaration, in both the `input`/`select` `:invalid[data-validity-visible]` branch and the `CustomSelectControl` branch. The `border-color: $alert-red` error border is untouched, so an invalid focused control keeps a red border and now also gets a red WPDS ring.

DataViews clipping fix, in `packages/dataviews/src/components/dataviews-filters/style.scss`:

```diff
 .dataviews-filters__summary-operators-filter-select {
 	width: 100%;
+	min-width: 0;
 	white-space: nowrap;
-	overflow: hidden;
 }
```

No JS API, hooks, block.json fields, REST schema, or DB changes; both `@wordpress/components` and `@wordpress/dataviews` changelogs are updated under Unreleased (Enhancements and Bug Fix respectively).

## Contribution

This landed as Phase 0 of the broader `@wordpress/components` → `@wordpress/ui` alignment effort (#80412), authored by @ciampo. The main design discussion was @jasmussen weighing in that he had liked the old convention — darkening and thickening a control's own 1px border to a 1.5px stroke as an "elegant" treatment that didn't complicate the silhouette — but endorsed the universal outset ring on maintainability grounds, noting it makes text, outline, and solid buttons share one focus style; @ciampo agreed that a universal ring also simplifies the components→ui transition. @mirka pushed commit `9821f3a` during review to handle the invalid-state (red) focus ring, and explicitly deferred the question of promoting the red focus colour to a public WPDS token, suggesting it may not need to be public at all — so this PR ships a Sass placeholder rather than a token. @ciampo disclosed that Codex was used to survey existing focus-ring patterns and history, audit affected consumers, implement the change, and draft the PR description, with the diff and generated CSS reviewed manually.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
