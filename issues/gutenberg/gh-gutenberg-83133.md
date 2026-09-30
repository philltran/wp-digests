# #83133: Featured image field: support the `editor.PostFeaturedImage` filter in the post summary

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Enhancement`, `[Feature] Extensibility`, `[Feature] DataViews`, `[Package] Fields`
- **Merged:** [`1354417`](https://github.com/WordPress/gutenberg/commit/1354417f86dc29dc1730abe72820947907e51466)
- **Discussion:** [#83133](https://github.com/WordPress/gutenberg/pull/83133) · 2 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The featured image field in the post summary (DataViews/Fields API) now applies the `editor.PostFeaturedImage` filter around its edit control, matching the behavior of the classic `PostFeaturedImage` panel. When the DataForm inspector experiment is enabled and the item is the post in the surrounding entity context, plugins that extend the featured image panel via that filter will see their injected markup in the post summary. The filter is deliberately not applied in Quick Edit (site editor) because the two contexts load different editor APIs.

## Impact

- **Plugin developers using `editor.PostFeaturedImage`:** Your filter callbacks will now fire in the post summary (DataViews) when the DataForm inspector experiment is active. The callback receives the same props the classic panel passed: `currentPostId`, `featuredImageId`, `media`, `postType`, and `onRemoveImage`. No code changes needed on your side.
- **Plugin developers targeting Quick Edit:** The filter is **not** applied there. The PR notes that plugins should use the Fields API (aimed for 7.2) for Quick Edit extensibility instead.
- **Site owners / theme developers:** No action required. Behavior is unchanged unless the DataForm inspector experiment is enabled.
- **No breaking changes.** This is purely additive; the classic panel and the post summary now both honor the same filter.

## Technical details

A new file `packages/fields/src/fields/featured-image/edit.tsx` introduces three components:

1. **`FilteredMediaEdit`** — created via `withFilters( 'editor.PostFeaturedImage' )` wrapping a thin `PostFeaturedImage` component that renders `<MediaEdit { ...props } isExpanded />`. The type is cast to include the classic-panel props (`currentPostId`, `featuredImageId`, `media`, `postType`, `onRemoveImage`).

2. **`FilteredFeaturedImageEdit`** — derives `featuredImageId` from `field.getValue()`, fetches `media` (via `getEntityRecords( 'postType', 'attachment', { include: [ featuredImageId ] })`) and `postType` (via `getPostType( data.type )`) from the `core` store using `useSelect`, then renders `FilteredMediaEdit` with all five classic props plus the DataForm `onChange`-backed `onRemoveImage`.

3. **`FeaturedImageEdit`** (default export) — the gate. It calls `useEntityId( 'postType', data.type )` and compares the result to `data.id`. If they match (the item *is* the post in the entity context, as in the post editor), it renders `FilteredFeaturedImageEdit`. Otherwise (Quick Edit, list views, etc.) it renders `<MediaEdit { ...props } isExpanded />` directly, bypassing the filter.

In `packages/fields/src/fields/featured-image/index.tsx`, the field's `Edit` property changes from an inline arrow to the new `FeaturedImageEdit` component:

```tsx
// before
Edit: ( props ) => <MediaEdit { ...props } isExpanded />,

// after
Edit: FeaturedImageEdit,
```

`@wordpress/hooks` is added as a devDependency to `packages/fields/package.json` (and `package-lock.json`) to support the new jsdom test. The test file `packages/fields/src/fields/featured-image/test/edit.jsdom.test.tsx` verifies three cases: filter applied when the item matches the entity context, filter skipped with no entity context, and filter skipped when a *different* post is in context.

## Contribution

Opened by @ntsekouras as part of the broader Fields/DataViews extensibility effort (issue #76076, related #82770). @oandregal is credited as co-author. The PR notes that Fable 5.1 (an AI tool) was used with direction, changes, and review. The discussion is minimal — two bot comments (co-author attribution and PR meta with bundle/perf data) and no substantive design debate in the record. The decision to exclude Quick Edit from the filter is documented in the PR body and code comments rather than in a review thread.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
