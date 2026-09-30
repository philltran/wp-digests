# #81604: DataViews: Add an aspect ratio toggle for grid layouts

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Feature] Media`, `[Type] Experimental`, `[Package] Media Utils`, `[Feature] DataViews`, `[Package] DataViews`
- **Merged:** [`c64d3b0`](https://github.com/WordPress/gutenberg/commit/c64d3b0a84e33959e1cee1e61c2ccc97421f09b9)
- **Discussion:** [#81604](https://github.com/WordPress/gutenberg/pull/81604) · 25 comments · 2 reactions
- **Usefulness:** 3/5

## Summary

DataViews grid layouts gain a `mediaFit` layout property (`'cover'` | `'contain'`). With `contain`, the media preview is fitted inside its box instead of cropped, so the item's own aspect ratio stays visible. Consumers can opt in to a user-facing "Original aspect ratio" toggle in the view options via `config.mediaFitControl`. It is first used in the experimental Media Upload Modal. The PR also fixes the `pickerGrid` layout not clipping previews to the rounded corners.

## Impact

- **Plugin & theme developers using `@wordpress/dataviews`:**
  - Additive and opt-in. Default remains `cover`, so existing grid and picker-grid views render as before.
  - To offer the toggle, pass `config={ { mediaFitControl: true } }` to `DataViews` or `DataViewsPicker`. It is off by default.
  - The toggle is hidden when the view renders no media field (`showMedia === false` or `mediaField` not among the fields).
  - `view.layout.mediaFit` is now part of persisted view state for `grid` and `pickerGrid`. Any code that serializes or compares `view.layout` will see this key.
- **Visual change in `pickerGrid`:** previews are now clipped to the media box's rounded corners (`overflow: hidden`), matching `grid`.
- **Site owners / editors:** only visible in the experimental Media Upload Modal (behind a Gutenberg experiment), where an "Original aspect ratio" toggle appears in the view options.
- **Headless / REST / hosting:** not affected.

## Technical details

**New layout property.** `mediaFit` is added to the `grid` and `pickerGrid` layouts (README table and types; the diff imports a `MediaFit` type from `../../types`). Any value other than `'contain'` is treated as `cover`.

**Rendering.** In `dataviews-layouts/grid/composite-grid.tsx` and `picker-grid/index.tsx`, `isMediaContain = view.layout?.mediaFit === 'contain'` adds a `has-media-fit-contain` class to the grid containers. The code comments say a class is used rather than a custom property (unlike `--wp-dataviews-media-aspect-ratio`) because the mode switches the box background token as well as `object-fit`. In each `style.scss`:

```scss
&.has-media-fit-contain .dataviews-view-grid__media {
  background-color: var(--wpds-color-background-surface-neutral);
  img { object-fit: contain; }
}
```

The picker-grid equivalent targets `.dataviews-view-picker-grid__media`. The box keeps its `aspectRatio` shape, so rows stay aligned.

**Toggle control.** New `utils/media-fit-control.tsx` renders a `ToggleControl` labelled "Original aspect ratio". It returns `null` unless `context.config?.mediaFitControl` is truthy and a media field is rendered. On change it calls `onChangeView` with `layout.mediaFit` set to `'contain'` or `'cover'`. It is added to `GridConfigOptions` between `DensityPicker` and `PreviewSizePicker`.

**Types.** The `config` type widens from `{ perPageSizes: number[] }` to `{ perPageSizes: number[]; mediaFitControl?: boolean }` in `DataViewsContextType` and `DataViewsPickerProps`.

**Picker-grid cleanup.** The diff removes the inline `gridTemplateColumns` style on the `GridItems` in the grouped branch of `ViewPickerGrid`. Nothing in the visible diff explains why, and it may be related to the corner-clipping fix. It is worth checking in the full diff.

**Other.** Storybook picker stories gain `mediaFit` and `mediaFitControl` args, and `CHANGELOG.md`/`README.md` are updated. The diff is truncated, so the `media-utils` changes that enable this in the Media Upload Modal are not visible here.

## Contribution

Opened by @andrewserong to address #81567, which was flagged in #79729, and the PR asked for feedback on the design before landing. @fcoveram supported the direction and wanted the toggle in the dropdown. Both noted that the selection checkbox sits awkwardly against 4:3 images. Follow-up ideas were an iCloud/macOS-style hover-only controls layout, with a question about accessibility if the ellipsis button relocates. @jasmussen suggested a grey background behind the photo; @fcoveram countered that this fails where the checkbox overlaps both the photo and the empty area. It was merged as opt-in and experimental, with the checkbox and layout polish left for follow-ups in the 7.2 cycle.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
