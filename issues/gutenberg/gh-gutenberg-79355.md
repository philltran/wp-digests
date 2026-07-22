# #79355: View Config: request a subset of properties with the `_fields` parameter

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @oandregal
- **Labels:** `[Type] Enhancement`, `[Package] Core data`, `[Package] Editor`, `[Package] Edit Site`, `[Feature] DataViews`
- **Merged:** [`c951883`](https://github.com/WordPress/gutenberg/commit/c95188333c3bb94dcacb21d456d9824766a064d0)
- **Discussion:** [#79355](https://github.com/WordPress/gutenberg/pull/79355) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `useViewConfig` hook and its underlying `core-data` resolver now accept an optional `fields` parameter, allowing consumers to request a subset of top-level view configuration properties via the REST API `_fields` query parameter. This optimization reduces payload size by ensuring each editor component fetches only the configuration keys it actually consumes. The reducer was also updated to merge partial responses, preventing subsequent subset requests from overwriting previously cached properties.

## Impact

- **Plugin & theme developers:** No breaking changes to public APIs, but the hook's return type now marks `default_view`, `default_layouts`, `view_list`, and `form` as `| undefined`. TypeScript consumers should handle optional properties. Direct calls to `core-data`'s `getViewConfig` resolver can now pass an `options.fields` array.
- **Editor & block developers:** No action required. Internal components have already been updated to request only their required keys.
- **Platform & hosting:** Reduced bandwidth and memory usage for `/wp/v2/view-config` requests when `_fields` is utilized.

## Technical details

- `packages/views/src/use-view-config.ts` now accepts `{ kind, name, fields? }` and passes a sorted, comma-separated `fields` string to `core-data`.
- `packages/core-data/src/resolvers.js` updates `getViewConfig` to map the `fields` option to the `_fields` REST parameter via `addQueryArgs( '/wp/v2/view-config', query )`.
- `packages/core-data/src/reducer.js` changes the `RECEIVE_VIEW_CONFIG` case from a direct spread overwrite (`{ ...state, [key]: action.config }`) to a shallow merge (`{ ...state[ key ], ...action.config }`), ensuring partial responses accumulate safely.
- Internal consumers (e.g., `page-patterns`, `page-templates`, `sidebar-dataviews`, `dataform-post-summary`) now pass explicit `fields` arrays.

**Usage pattern change:**
```ts
// Before
const { view_list } = useViewConfig( { kind: 'postType', name: 'page' } );

// After
const { view_list } = useViewConfig( { kind: 'postType', name: 'page', fields: ['view_list'] } );
```

## Contribution

Authored by @oandregal and co-authored by @ntsekouras as a follow-up to #79347. Review centered on verifying that the reducer correctly merges partial responses without clobbering cached state, and that internal editor screens consistently scope their requests. The PR description notes that a server-side optimization to skip computing unrequested database-backed properties is intentionally deferred for a future backward-compatible addition.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
