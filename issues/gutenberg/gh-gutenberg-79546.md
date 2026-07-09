# #79546: DataViews: Fix infinite-scroll jump on async page loads

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @arthur791004
- **Labels:** `[Type] Bug`, `[Package] DataViews`
- **Merged:** [`bcb77ad`](https://github.com/WordPress/gutenberg/commit/bcb77ad34b276ed6d006ab3167a36b41db187050)
- **Discussion:** [#79546](https://github.com/WordPress/gutenberg/pull/79546) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a scroll-jump bug in the DataViews `List` layout when using infinite scrolling over async network requests. The issue was caused by the scroll-anchor restoration logic discarding user scrolling during load gaps, and the footer resizing the container mid-load. The fix captures the container's `scrollTop` at anchor capture, compensates for user-induced scroll deltas on restore, and prevents the footer from mounting/unmounting during page loads while keeping a fixed-height loading spinner in place.

## Impact

- **DataViews consumers / Block developers**: No breaking changes or required code updates. The fix applies transparently to any `List` layout with `infiniteScrollEnabled: true`. Scroll position will now remain stable during async data fetches.
- **Platform / Editor engineers**: No action required. Internal rendering logic for `DataViewsFooter` and the infinite scroll hook was updated to handle loading states more gracefully. Footer visibility is now governed by a shared internal check that respects both pagination controls and bulk actions.
- **Site owners**: No direct impact; improves UX in any list-based data view using infinite scroll.

## Technical details

The core logic shifts in `packages/dataviews/src/hooks/use-infinite-scroll.ts`, where the anchor capture object now stores `scrollTop` and the restoration calculation compensates only for layout shifts:

```diff
- const scrollAdjustment = currentOffset - anchor.viewportOffset;
+ const scrollAdjustment = currentOffset - anchor.viewportOffset + (container.scrollTop - anchor.scrollTop);
```

In `packages/dataviews/src/components/dataviews-footer/index.tsx`, footer rendering was refactored to use a new `hasPaginationControls()` helper from `dataviews-pagination`, simplifying the conditional mount logic and preventing layout thrashing. Additionally, `packages/dataviews/src/components/dataviews-layouts/list/index.tsx` ensures the loading spinner remains reserved when `hasMoreItems` is true, with `.dataviews-loading-more[aria-hidden="true"]` set to `visibility: hidden` in `style.scss` to prevent height flapping. A new Storybook story (`AsyncInfiniteScroll`) was added for realistic network-backed testing. Note: IntersectionObserver-based row unloading was explicitly deferred to a follow-up PR due to complexity with the anchor system.

## Contribution

Opened and merged by @arthur791004 with co-authoring from @tellthemachines. The PR isolated two root causes for the scroll jump and implemented a two-part fix: anchor delta compensation and footer/container sizing stabilization. During review, @tellthemachines noted that IntersectionObserver row unloading was missing from the list layout and considered extending it alongside this fix, but agreed to merge the core bug fix first due to testing complexity and deferred the unloading logic to a follow-up PR.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
