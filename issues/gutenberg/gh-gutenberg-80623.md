# #80623: Editor: Avoid unnecessary term re-fetches in FlatTermSelector

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Performance`, `[Package] Editor`
- **Merged:** [`8c607b4`](https://github.com/WordPress/gutenberg/commit/8c607b4aa218d593fec04594fa153682331e70ea)
- **Discussion:** [#80623](https://github.com/WordPress/gutenberg/pull/80623) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The block editor's `FlatTermSelector` component now prevents redundant `getEntityRecords` API calls when managing post taxonomies. By sorting the `include` array before building the query key and clearing the search state after term selection or creation, the editor avoids unnecessary network requests triggered by term reordering or stale search filters. This is a performance optimization with no behavioral changes.

## Impact

- **Plugin & theme developers / block authors:** No action required. This is an internal editor optimization with no public API changes.
- **Site owners / editors:** Smoother taxonomy selection experience with fewer network requests when assigning, creating, or reordering terms.
- **Hosting & platform teams:** Negligible impact; reduced API load is a minor positive.
- No breaking changes, deprecations, or migration steps.

## Technical details

The change modifies `packages/editor/src/components/post-taxonomies/flat-term-selector.js`. Two specific adjustments were made to the `getEntityRecords` query construction and state management:
- The `include` parameter in the query object now sorts `_termIds` numerically before joining them into a comma-separated string. This ensures that reordering the same set of term IDs produces an identical query key, preventing cache misses and redundant fetches.
- `setSearch( '' )` is called immediately after `setValues( uniqueTerms )` during term updates. This clears any lingering search input that would otherwise trigger an extra `getEntityRecords` call with a stale query parameter.

```javascript
// Before
include: _termIds?.join( ',' ),

// After
include: _termIds?.length
    ? [ ..._termIds ].sort( ( a, b ) => a - b ).join( ',' )
    : undefined,
```
The PR notes that a third potential optimization (clearing stale `termIds` queries) was deferred due to implementation complexity.

## Contribution

The author identified a third potential optimization to clear stale `termIds` queries but explicitly deferred it, noting it would require excessive state bookkeeping. The reviewer accepted this trade-off, and the PR merged with only the two query-key and search-state fixes.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
