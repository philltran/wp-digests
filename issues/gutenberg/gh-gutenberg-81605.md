# #81605: Global Styles: Allow editing duotone palettes

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @annezazu
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] Block editor`, `Global Styles`
- **Merged:** [`7cea1c7`](https://github.com/WordPress/gutenberg/commit/7cea1c72e1e1e88ee0ee865bae34bcd4aa042bfb)
- **Discussion:** [#81605](https://github.com/WordPress/gutenberg/pull/81605) · 11 comments · 3 reactions
- **Usefulness:** 3/5

## Summary

The Global Styles "Edit palette" screen gains a third Duotone tab next to Color and Gradient, so theme, default and user-defined duotone presets can be edited, renamed, removed, reset, and custom ones added. The read-only duotone swatch list that used to sit under the Gradient tab is removed. Along the way, `DuotonePicker` now selects presets by slug rather than by color, fixing a bug where two presets sharing the same color pair both appeared selected and the wrong one was saved. Closes the long-standing #36541.

## Impact

**Site owners / editors**
- Appearance → Editor → Styles → Colors → Edit palette now offers Color, Gradient and Duotone tabs. Custom duotones persist under `settings.color.duotone.custom` and can be applied to Image blocks, previewing on the canvas and rendering on the front end.
- The Gradient tab no longer shows the read-only duotone list.

**Plugin & theme developers**
- Changes are additive on the public surface: `DuotonePicker` and `DuotoneControl` gain an optional `selectedSlug` prop, and `DuotonePicker`'s `onChange` now receives `(value, index?, slug?)`. Existing callers that only read the first argument keep working.
- `PaletteEdit` gains a `duotones` prop (with optional `colorPalette`). Internally, the private `isGradient` boolean became a `variant` discriminator.
- Behavior change for block duotone selection: a picked preset is now saved by its slug (`var:preset|duotone|<slug>`) instead of being matched back by colors, which previously resolved to the first preset with that pair.
- If you pass `selectedSlug` to `DuotonePicker`, selection is strictly by slug, so presets with a non-matching slug won't appear selected even if their colors match `value`. An empty string falls back to color matching.
- Palette colors that don't resolve to concrete values (e.g. `color-mix()`) are excluded from the duotone pickers, and the rest are normalized to hex.

**Hosting / platform / PHP**
- No PHP changes.

No action required; the feature targets Gutenberg 7.2.

## Technical details

**Components (`packages/components`)**
- `PaletteEdit` gets a third variant, `'duotone'`. The duotone branches reuse `CustomDuotoneBar` and `ColorListPicker` (Shadows / Highlights) in the edit popover, `DuotonePicker` for the collapsed preview, and `getGradientFromCSSColors( colors, '135deg' )` and `getDefaultColors( colorPalette )` from `duotone-picker/utils` for swatches and newly added values.
- `DuotonePicker` adds `selectedSlug` and calls `onChange( value, index, slug )` when a preset is picked. Index and slug are omitted for custom, unset and clear controls, and when deselecting the current preset (which reports `undefined` alone). It also no longer renders the custom controls wrapper when `disableCustomDuotone` is set, removing trailing padding on read-only pickers.
- The swatch picker in `PaletteEdit` is now labelled by its palette heading (Theme / Default / Custom).

**Block editor (`packages/block-editor`)**
- `DuotoneControl` forwards `selectedSlug` to `DuotonePicker`.
- New helper `getDuotoneSlugFromPreset()` in `components/global-styles/utils.js` parses `var:preset|duotone|<slug>`. It lives there because `hooks/duotone` imports the filters panel, which would create a circular import.
- `FiltersPanel` derives `duotoneSlug` from the raw (undecoded) value, and `setDuotone( newValue, slug )` prefers the slug over a color-match lookup.
- `hooks/duotone.js` now passes the raw `duotoneStyle` into `StylesFiltersPanel` instead of the resolved colors, and in `onChange( newDuotone, index, slug )` uses the slug when present, falling back to `getDuotonePresetFromColors`.

```js
// Before: preset resolved by colors, first match wins
const maybePreset = getDuotonePresetFromColors( newDuotone, duotonePalette );

// After: slug is authoritative when a preset was picked
const maybePreset = slug
	? `var:preset|duotone|${ slug }`
	: getDuotonePresetFromColors( newDuotone, duotonePalette );
```

Per the PR description (the diff shown was truncated, so these parts are not verified against code here): a new `DuotonePalettePanel` mirrors `GradientPalettePanel` (Theme/Default sections with `canOnlyChangeValues`, Custom with `slugPrefix="custom-"`, Default gated on `color.defaultDuotone`), and the editor now generates SVG filters for user-defined duotones so custom ones preview on the canvas. Custom duotones work without PHP changes because `WP_Duotone::get_all_global_styles_presets()` already iterates every origin. Tests were added for `DuotoneControl`, `FiltersPanel` and `getDuotoneSlugFromPreset`.

## Contribution

@annezazu opened the PR as an exploratory, Claude Code-assisted prototype following @jasmussen's three-tab proposal on #36541, explicitly inviting someone to take it over and questioning whether three tabs fit the available space. @jasmussen and @fcoveram approved the UX and visuals, with Jasmussen noting the remaining blocker was code review and flagging concern about AI-generated test volume. @ramonjd then pushed substantial follow-up work: slug/index-based selection, editor SVG filters for custom duotones, filtering unparseable palette colors, and hex normalization to match the front-end parser. @andrewserong adjusted spacing to match the gradient picker on TT3. Two pre-existing accessibility issues (#81799, #81800) were deliberately deferred, and one review thread was left open for a possible follow-up.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
