# #82003: Enable Gallery columns and crop controls in viewport states

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Gallery`, `[Package] Block editor`, `[Feature] Style States`
- **Merged:** [`b03cafc`](https://github.com/WordPress/gutenberg/commit/b03cafc9d1d12a502e9e3c84d4b14ec6aac20e4f)
- **Discussion:** [#82003](https://github.com/WordPress/gutenberg/pull/82003) · 3 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The Gallery block's Flex layout now supports per-viewport `columns` and `imageCrop` ("Crop images to fit") values when a viewport style state is selected in the editor. To let blocks place their own responsive controls, the PR adds a new `viewport` group to `InspectorControls`, rendered only while a viewport (non-pseudo) style state is active. The remaining logic is internal to the Gallery block. It is part of the Style States effort (#80388).

## Impact

- **Plugin & theme developers:**
  - Blocks can now render controls for viewport style states with `<InspectorControls group="viewport">`. The slot is only rendered when the selected style state has a viewport and no pseudo state.
  - No deprecations or removed public APIs.
  - Code that targeted the Gallery editor wrapper via `#block-{clientId}` for gap styles no longer matches; the editor wrapper now also carries `wp-block-gallery-{clientId}`, which the gap and responsive styles use.
- **Site owners / editors:**
  - With responsive styles enabled, selecting a viewport on a Flex-layout Gallery exposes only Columns and Crop images to fit. Source, Resolution, Randomize order, Open in new tab, Aspect ratio and lightbox navigation controls are hidden in that state.
  - The author notes this does not fix #41781: mobile wrapping comes from block styles, not the columns setting, and a "stack on mobile" option may be a follow-up.
- **Headless & REST consumers:** Per-viewport values are stored in the block's `style` attribute under the viewport key, and are read via `getViewportGalleryStyle`. The truncated diff does not show the exact serialized shape or front-end render changes, so verify against the source.

## Technical details

**Block editor**
- `block-inspector/index.jsx`: `showLayoutControls` is renamed to `isViewportStyleState`. It is true when `hasViewportBlockStyleState()` holds and `hasPseudoBlockStyleState()` does not. It now also gates a new `<InspectorControls.Slot group="viewport" />`.
- `inspector-controls/groups.js`: adds `InspectorControlsViewport = createSlotFill( 'InspectorControlsViewport' )` and registers it as `viewport` in `groups`.

**Gallery** (`packages/block-library/src/gallery/`)
- `edit.jsx`:
  - Reads `getSelectedBlockStyleState( clientId )` via `unlock( select( blockEditorStore ) )`.
  - When a non-default viewport is selected, it derives `activeColumns` and `activeImageCrop` from `getViewportGalleryStyle( attributes.style, viewport )`, falling back to the base `columns` and `imageCrop` attributes.
  - Writes go through the new `setGalleryFlexSettings()`. Outside viewport states it calls `setAttributes( settings )`. In a viewport state it writes `style` via `getUpdatedGalleryStyle( { style, viewport, baseSettings, settings } )`.
  - `<InspectorControls>` uses `group="viewport"` when in a viewport state with a Flex layout, otherwise `default`.
  - The Settings `ToolsPanel` reset in a viewport state clears only `columns` and `imageCrop`.
  - The editor wrapper gets a `wp-block-gallery-${ clientId }` class.
  - `MAX_COLUMNS` moves to `constants.js`.
- `gap-styles.js` is renamed to `flex-styles.js`, and `GalleryGapCustomProperties` becomes `GalleryFlexStyles`. Its selector changes from `#block-${ clientId }` to `.wp-block-gallery-${ clientId }`, and it imports `getGalleryResponsiveFlexCSS` from the new `responsive-styles` module. The CSS accumulator is renamed from `gap` to `css`.
- The new `responsive-styles` module exports `getViewportGalleryStyle`, `getUpdatedGalleryStyle`, `isValidGalleryColumns` and `getGalleryResponsiveFlexCSS`. Its source is not visible in the truncated diff.

**Changelogs:** entries were added to the `block-editor` and `block-library` packages.

The `viewport` group is consumed only through the block editor's unlocked style-state machinery, so it should be treated as experimental.

## Contribution

The PR is stacked on #81909 because both touch the same files, though this one affects only the Flex layout. The author disclosed AI tooling use (codex). The one open design question is how to handle #41781, which would need a possible "stack on mobile" setting and is left for a follow-up. Discussion on the PR is limited to bot comments.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
