# #82244: DataViews: clamp page after delete

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @oandregal
- **Labels:** `[Type] Bug`, `[Package] Core data`, `[Feature] Site Editor`, `[Feature] DataViews`, `[Package] DataViews`
- **Merged:** [`7b4305f`](https://github.com/WordPress/gutenberg/commit/7b4305f23889a6d5e50e6cd9623cf448d6ffb0a0)
- **Discussion:** [#82244](https://github.com/WordPress/gutenberg/pull/82244) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

DataViews now moves the view back to the last existing page when `view.page` points past the end of the collection, such as after trashing the only item on the last page. Previously the list stayed on a nonexistent page and the REST request returned a 400. To make this work, `@wordpress/core-data` now decrements a query's `meta.totalItems` when records are removed from it, so the page count is correct immediately after a deletion rather than only after the next fetch.

## Impact

- **Plugin & theme developers using `@wordpress/dataviews`:** `DataViews` and `DataViewsPicker` automatically call `onChangeView` with a clamped `page` when `view.page > totalPages` and loading has finished. Controlled-view consumers will see an extra `onChangeView` call in that situation. Consumers that maintain their own page-clamp effect become redundant (the Guidelines route dropped its own in this PR).
- **Consumers of `@wordpress/core-data`:** After `removeItems`, affected queries have `meta.totalItems` reduced by the number of removed IDs the query listed, and `meta.totalPages` set to `null` until the next fetch. Code that reads `meta.totalPages` directly (rather than through the selectors) may now see `null` after a deletion. Queries that set `per_page` are unaffected, since the selectors derive the page count from `totalItems`.
- **Site owners / editors:** In Site Editor → Pages (both the standard and `gutenberg-extensible-site-editor` experiment editors), trashing the last item of the last page, or bulk-trashing, now lands on the new last page instead of an empty page.
- **Not fixed:** Loading a URL with an out-of-range page (e.g. `?p=%2Fpage&pageNumber=5` with only 2 pages) is still unsolved. The server returns a 400 without `X-WP-Total`/`X-WP-TotalPages` headers, core-data's resolver swallows the error, and DataViews receives `totalPages: null`, so the clamp deliberately does nothing.
- No migration required.

## Technical details

**core-data** (`packages/core-data/src/queried-data/reducer.js`): the `REMOVE_ITEMS` branch of the `queries` reducer now calls a new `removeQueryItems( queryItems, removedItems )` helper instead of only filtering `itemIds`. It filters `itemIds`, computes `removedCount`, and returns the original object if nothing was removed. If `meta.totalItems` is a finite number, it sets `totalItems = max( 0, totalItems - removedCount )` and `totalPages = null`. `totalPages` is nulled because the reducer doesn't know the page size behind the `X-WP-TotalPages` header. Queries that didn't list the removed item keep their meta unchanged (covered by the `s=a` case in the new reducer test).

**dataviews**: a new hook, `packages/dataviews/src/hooks/use-page-clamp.ts` (also exported from `hooks/index.ts`), is used in both `dataviews/index.tsx` and `dataviews-picker/index.tsx`:

```ts
usePageClamp( {
	view,
	onChangeView,
	isLoading,
	totalPages: paginationInfo.totalPages,
} );
```

It derives `lastPage = max( totalPages, 1 )` when `totalPages` is a finite number, otherwise `null`. An effect calls `onChangeView( { ...view, page: lastPage } )`, via `useEvent` from `@wordpress/compose`, when `! isLoading`, `lastPage !== null`, `view.page` is set, and `view.page > lastPage`. It is skipped while loading and when totals are unknown (`null`). An empty collection falls back to page 1.

**Guidelines route**: `routes/guidelines/components/block-guidelines.tsx` removes its local `useEffect` that clamped `view.page` against `paginationInfo.totalPages`.

Tests were added for the reducer and for DataViews clamping (past-the-end, empty collection, loading, unknown total, valid page). CHANGELOG entries were added for both packages.

## Contribution

The author (@oandregal) drafted the change with Claude Code assistance, disclosed in the PR, and reviewed it. In the discussion, he notes that a reviewer's LGTM acknowledged the cold-start out-of-bounds case from the linked issue is not handled. He explains the cause (400 without total headers, swallowed by the core-data resolver) and is wary of a DataViews-side effect resetting to page 1, since DataViews can't distinguish a bad page from a transient network failure. He suggests the real fix belongs in the server/data layer, for example surfacing the error or retrying for totals, and leaves that unresolved.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
