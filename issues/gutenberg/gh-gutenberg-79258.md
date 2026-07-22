# #79258: Cover block: add media editor modal

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Cover`
- **Merged:** [`64c7500`](https://github.com/WordPress/gutenberg/commit/64c7500ae141881de8052b2f22c396fcd7395d8f)
- **Discussion:** [#79258](https://github.com/WordPress/gutenberg/pull/79258) · 8 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Cover block now includes a "Crop background image" toolbar button that opens the shared media editor modal when an editable image attachment is selected. This allows authors to crop, rotate, or flip the cover image directly from the block toolbar without introducing Cover-specific transformation controls. The button automatically hides for videos, embeds, featured images, temporary uploads, or when the media editor is unavailable.

## Impact

- **Block authors & editors:** Gain direct access to image editing tools (crop, rotate, flip) within the Cover block toolbar.
- **Plugin & theme developers:** No breaking changes or required code updates. The feature relies on existing block-editor internals and only activates when standard conditions are met.
- **Hosting & platform teams:** No configuration or migration required.
- **Headless & REST consumers:** No impact on REST API responses or block serialization.

## Technical details

The change modifies `packages/block-library/src/cover/edit/index.js` and `packages/block-library/src/cover/edit/block-controls.js`. It unlocks `blockEditorPrivateApis` to access `openMediaEditorModalKey`, retrieves the modal opener via `getSettings()`, and wires it to a new `ToolbarButton` labeled "Crop background image". The button renders conditionally when `showEditMediaButton` evaluates to true (requires `hasNonContentControls`, `!useFeaturedImage`, `isImageBackground`, valid `id`/`url`, and `openMediaEditorModal` availability). When the modal closes after an edit, `onUpdate` triggers `setAttributes` with the updated `id`, `url`, `sizeSlug` (reset to `DEFAULT_MEDIA_SIZE_SLUG`), and recalculates `overlayColor` via `getMediaColor()` if the overlay wasn't manually set. It also updates `isDark` using `compositeIsDark()` and marks the change as non-persistent via `__unstableMarkNextChangeAsNotPersistent()` to group the attribute and color updates in the undo stack.

## Contribution

Opened by @ramonjd, the PR was reviewed and tested by @andrewserong and @jasmussen. During review, the team debated whether the Cover block’s automatic overlay color should recalculate after a crop operation. @andrewserong noted that recalculating might produce unexpected contrast shifts, while @ramonjd acknowledged the complexity but decided to implement the recalculation for non-user-set overlays, deferring edge-case refinement to a follow-up. The approach was merged as a focused enhancement that reuses the existing media editor rather than building custom transform controls.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
