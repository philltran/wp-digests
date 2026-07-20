# #80198: Latest Posts: Fix slow category selection with large category lists

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Performance`, `[Package] Block library`, `[Block] Latest Posts`
- **Merged:** [`c7890e1`](https://github.com/WordPress/gutenberg/commit/c7890e1da67be2e5c8d06f284cec1a66f1f33784)
- **Discussion:** [#80198](https://github.com/WordPress/gutenberg/pull/80198) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Latest Posts block's category selector now handles large category lists efficiently by replacing an O(n²) object spread pattern with an O(n) in-place mutation during the `categorySuggestions` map generation. This eliminates noticeable UI lag when editing block controls on sites with thousands of categories. The change improves editor responsiveness without altering the block's public API or rendered output.

## Impact

- **Block & plugin developers:** No action required. The block's attributes, save logic, and client-side API remain unchanged.
- **Site owners with large taxonomies:** Will experience faster control interactions in the Latest Posts block when managing hundreds or thousands of categories.
- **Hosting & platform teams:** No configuration changes needed. The fix is contained within the block editor's client-side rendering logic.

## Technical details

The change modifies `packages/block-library/src/latest-posts/edit.js` inside the `Controls` component. Previously, `categoriesList?.reduce()` used an object spread on every iteration, creating a new object reference each time and resulting in O(n²) complexity. The diff replaces this with direct property assignment on the accumulator, reducing complexity to O(n). The reducer still seeds a fresh object per run, preventing shared state mutation.

**Before:**
```js
categoriesList?.reduce(
	( accumulator, category ) => ( {
		...accumulator,
		[ category.name ]: category,
	} ),
	{}
) ?? {};
```

**After:**
```js
categoriesList?.reduce( ( accumulator, category ) => {
	accumulator[ category.name ] = category;
	return accumulator;
}, {} ) ?? {};
```

No new hooks, filters, or block.json fields are introduced.

## Contribution

Opened by @Mamaduka and merged following a brief review cycle. @tyxla provided approval, and the author noted the performance regression was discovered while testing a related PR (#80191). The discussion remained focused on validating the O(n) optimization for large category lists, with no alternative approaches proposed or rejected.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
