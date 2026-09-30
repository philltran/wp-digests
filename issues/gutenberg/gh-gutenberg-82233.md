# #82233: Enable viewport state values for Gallery aspect ratio

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Gallery`, `[Feature] Style States`
- **Merged:** [`f4e4afb`](https://github.com/WordPress/gutenberg/commit/f4e4afbbfd3554905af6f05a430bf92dfb24aedf)
- **Discussion:** [#82233](https://github.com/WordPress/gutenberg/pull/82233) · 8 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The Gallery block's aspect ratio control now supports viewport-specific overrides (tablet and mobile) in both the Flex and Grid layout variations, including dynamic galleries. Previously the aspect ratio `SelectControl` was hidden when the editor was in a viewport style state; it is now always visible and writes to the block's `viewportStyle` attribute. The change also refactors the Gallery's editor-side style generation from a Flex-only component into a layout-agnostic one so that responsive aspect-ratio CSS is emitted for every Gallery layout.

## Impact

- **Site owners / content editors:** Can now set different aspect ratios for desktop, tablet, and mobile on any Gallery block (Flex, Grid, or dynamic). No code changes required; the control appears in the block's Settings panel when a viewport is selected.
- **Theme developers:** Viewport-specific aspect-ratio rules are emitted via `useStyleOverride` with `!important` to override the inline `aspect-ratio` style on each `<img>`. Themes that previously relied on the inline style being the sole source of aspect-ratio CSS should be aware that a media-query rule now takes precedence at smaller viewports.
- **Plugin developers:** No new public API or hook is introduced. The internal `setGalleryFlexSettings` function was renamed to `setGallerySettings` in `edit.jsx`, but this is not a public symbol. The `flex-styles.js` module was deleted and replaced by `gallery-styles.js`; any code importing the old path will break, though it was never a public export.
- **No action required** for most developers. The change is additive to the editor UI and the generated CSS.

## Technical details

The diff makes several coordinated changes across the Gallery block's editor and server-rendering code:

**Editor (`edit.jsx`):**
- A new `baseAspectRatio` constant normalises the attribute: `aspectRatio || 'auto'`, so that "no ratio" and "Original" compare equal and a viewport override matching either is dropped rather than stored.
- `hasViewportAspectRatio` and `activeAspectRatio` follow the same pattern already used for `columns` and `imageCrop`.
- The `setAspectRatio` function now branches: in a viewport style state it delegates to `setGallerySettings` (which writes into `viewportStyle`); otherwise it sets the base attribute and propagates `aspectRatio` and a new `scale: 'cover'` attribute to every inner Image block via `updateBlockAttributes`.
- The aspect-ratio `ToolsPanelItem` is no longer gated behind `! isViewportStyleState`. Its `onDeselect` passes `undefined` in a viewport state (clearing the override) versus `'auto'` at the base level.
- The viewport-group visibility check changes from `isFlexLayout` to `hasViewportSettings = isFlexLayout || aspectRatioOptions.length > 1`, so a non-Flex Gallery with aspect-ratio presets still shows the viewport panel.

**Style generation (`flex-styles.js` → `gallery-styles.js`):**
- The old `GalleryFlexStyles` component is deleted. The new `GalleryStyles` component accepts an `isFlexLayout` prop. The gap custom-property CSS (`--wp--style--unstable-gallery-gap`) is only emitted when `isFlexLayout` is true. The responsive CSS call changes from `getGalleryResponsiveFlexCSS` to `getGalleryResponsiveCSS` (both from `./responsive-styles`), and that call now runs unconditionally so aspect-ratio media queries are generated for Grid and other layouts as well.

**Server rendering (`index.php`):**
- A new `isValidGalleryAspectRatio()` helper (truncated in the diff) validates that a stored aspect-ratio value is one of the numeric preset forms or `auto` before it is interpolated into the generated stylesheet, preventing CSS injection from malformed saved content.

**Inner Image blocks:**
- When a non-auto aspect ratio is set, each inner Image block now also receives `scale: 'cover'` so the image fills the constrained box rather than letterboxing.

## Contribution

Opened by @tellthemachines as part of the broader #80388 effort to roll viewport style states out to more block controls. @ramonjd reviewed, initially reported what looked like a regression where the block inspector's Styles panel closed when switching from a viewport back to Desktop, but after @tellthemachines confirmed the behaviour was identical on trunk and 7.1, @ramonjd retracted the report. @ramonjd then rebased the PR, retested across new and existing galleries, and confirmed it was ready. @andrewserong is listed as a co-author in the merge commit.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
