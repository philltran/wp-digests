# #80038: Media Inserter: Add pagination to core media inserter categories

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Enhancement`, `[Feature] Media`, `[Package] Editor`, `[Package] Block editor`
- **Merged:** [`cf215c1`](https://github.com/WordPress/gutenberg/commit/cf215c110389122105468a594861f4b9a98e5a57)
- **Discussion:** [#80038](https://github.com/WordPress/gutenberg/pull/80038) · 9 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The core media inserter now supports pagination for its categories, allowing editors to browse beyond the previous 10- or 20-item limit. This enhancement reuses the existing `BlockPatternsPaging` component and updates the media category registration API to optionally return pagination metadata. It enables deeper navigation through large media libraries without requiring a full redesign of the inserter UI.

## Impact

- **Plugin & theme developers:** If you register custom media categories via `registerInserterMediaCategory`, your `fetch` callback can now return an `InserterMediaResponse` object (`{ mediaItems, totalItems, totalPages }`) instead of a plain array to opt into pagination. Returning an array remains supported and disables pagination.
- **CSS/Theme authors:** The `.has-attach-footer` class on the media panel has been renamed to `.has-footer`. Custom themes or plugins overriding this selector will need to update their styles.
- **Site owners & editors:** No action required. The inserter automatically renders pagination controls when a category contains more than one page of results.

## Technical details

- The `fetch` signature for `registerInserterMediaCategory` now accepts `InserterMediaRequest` (which includes a new `page` parameter) and returns `Promise<InserterMediaItem[]|InserterMediaResponse>`.
- `packages/block-editor/src/components/inserter/media-tab/hooks.js` introduces `normalizeFetchResult()` to handle both array and object returns, and `useMediaResults()` now exposes `totalItems` and `totalPages` alongside `mediaList` and `isLoading`.
- `packages/block-editor/src/components/inserter/media-tab/media-panel.js` manages pagination state (`page`), clamps the page number when results shrink, and renders `BlockPatternsPaging` inside a new `<Stack>` footer wrapper. The default page size was increased from 10 to 20 (`MEDIA_ITEMS_PER_PAGE`).
- CSS in `packages/block-editor/src/components/inserter/style.scss` updates the `.has-attach-footer` selector to `.has-footer` and adjusts flex/padding rules for the new footer layout.
- JSDoc in `packages/block-editor/src/store/actions.js` and `docs/reference-guides/data/data-core-block-editor.md` defines the new `InserterMediaResponse` interface and updates the `fetch` type definition.

## Contribution

Opened and merged by @andrewserong with co-authors @ramonjd and @jasmussen. The PR was developed as a pragmatic solution for WordPress 7.1, reusing the existing `BlockPatternsPaging` component from the Pattern inserter rather than extracting a shared pagination component. @jasmussen suggested consolidating pagination into a reusable component, but the author deferred that to 7.2+ to meet the 7.1 Beta 1 timeline. The API change was explicitly designed to be backwards compatible while establishing a pattern for future media categories.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
