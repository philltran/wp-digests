# #80852: Global Styles: Expose additional elements in Typography and Colors

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @MaggieCabrera
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `Global Styles`
- **Merged:** [`3bb6eb7`](https://github.com/WordPress/gutenberg/commit/3bb6eb7a08697ad16033851aa4b780be5ca0ac4f)
- **Discussion:** [#80852](https://github.com/WordPress/gutenberg/pull/80852) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `cite`, `textInput`, and `select` theme.json elements — already supported by theme.json parsing and CSS generation, but invisible in the editor — are now surfaced in the Global Styles UI under **Typography → Elements** and **Colors → Elements** as "Citations", "Inputs", and "Selects". Users can set font and text/background color for quote and pullquote citations, text inputs (Search block, comment form fields), and select dropdowns (Archives, Categories) without hand-editing theme.json. To make this possible the shared `ColorPanel` in `@wordpress/block-editor` gained a new `additionalElements` prop so the Global Styles colors screen can inject element rows beyond the built-in Button/Heading/Caption/Link set. No theme.json schema, CSS generation, or PHP behavior changed — this is purely UI exposure of existing element support.

## Impact

**Theme developers**
- `styles.elements.cite`, `styles.elements.textInput`, and `styles.elements.select` declared in `theme.json` now appear as *inherited* values in the UI and can be overridden by the user, which is written to the user global styles record. If your theme relied on these element styles being effectively unreachable from the editor, expect user overrides to start winning. Worth a regression pass on Search, comment form, Archives/Categories, Quote, and Pullquote rendering.
- Nothing to add to `theme.json` — no new schema keys, no version bump required.

**Plugin / package consumers**
- `ColorPanel` (exported through `blockEditorPrivateApis` and consumed via `unlock`) takes an optional `additionalElements` array of `{ name, label }`. It is a private API, so this is not a supported extension point for third-party code, but anyone already reaching into it via `unlock` should know the props changed additively.

**Site owners / editors**
- New controls only; no migration, no configuration. Existing global styles values are untouched.

No breaking changes, no deprecations, no action required for most sites.

## Technical details

Three surfaces changed in `packages/global-styles-ui/src/`, plus one shared component in `packages/block-editor/`.

`color-panel.js` gains an `additionalElements` prop that is spread into the memoized element list, each entry gated behind the panel's existing color-support check:

```js
...( additionalElements?.map( ( element ) => ( {
	...element,
	showPanel: hasSolidColors || hasGradientColors,
} ) ) ?? [] ),
```

The memo dependency array is widened to `[ additionalElements, hasGradientColors, hasSolidColors, showCaptionPanel, showButtonPanel, showHeadingPanel ]`. Note the gating differs from the built-ins: Caption/Button/Heading each have their own `showPanel` predicate, while injected elements render whenever solid or gradient color support exists.

`screen-colors.tsx` supplies the data:

```tsx
const ADDITIONAL_ELEMENTS = [
	{ name: 'cite', label: __( 'Citations' ) },
	{ name: 'textInput', label: __( 'Inputs' ) },
	{ name: 'select', label: __( 'Selects' ) },
];
const DEFAULT_CONTROLS = {
	link: true, heading: true, button: true, caption: true,
	cite: true, textInput: true, select: true,
};
// …
<StylesColorPanel
	additionalElements={ ADDITIONAL_ELEMENTS }
	defaultControls={ DEFAULT_CONTROLS }
	/* … */
/>
```

On the typography side, `typography-elements.tsx` adds three `<ElementItem>` rows (`cite` after Captions, `textInput` and `select` after Buttons); `screen-typography-element.tsx` adds matching `title`/`description` strings to its `elements` map; and `global-styles-ui.tsx` registers the routes `/typography/cite`, `/typography/textInput`, and `/typography/select`, each rendering `<ScreenTypographyElement element="…" />`.

A Jest case in `packages/block-editor/src/components/global-styles/test/color-panel.js` covers the new prop, asserting an `additionalElements={ [ { name: 'textInput', label: 'Inputs' } ] }` row renders. Changelogs were updated in both `@wordpress/block-editor` (the `additionalElements` capability) and `global-styles-ui` (the three new element screens).

## Contribution

The interesting part of the review was a design fork rather than the code. @jasmussen approved the incremental approach but asked whether elements belong as children of Typography/Colors at all, or whether a new top-level Styles section listing them would be better — since border, radius, and shadow are meaningful for inputs and have nowhere to live under Typography or Colors. @yjailin argued for a dedicated top-level "Form controls" section holding border, radius, fill, and label position together, while explicitly endorsing reuse of this PR's element controls for color and typography rather than duplicating them; @elizaan36 agreed both should exist. Separately, @fcoveram and @yjailin independently raised that listing Inputs and Selects as distinct color rows lets them diverge when most sites would want them to match, and suggested grouping them — a change not made here. The PR shipped as the incremental step with the broader form-controls direction deferred and tracked in #81648; @scruffian is also credited in the props list.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
