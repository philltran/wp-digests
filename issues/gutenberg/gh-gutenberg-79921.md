# #79921: Media Inserter: Allow core media categories to subscribe to changes

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Enhancement`, `[Feature] Media`, `[Package] Editor`, `[Package] Block editor`
- **Merged:** [`ad50f2a`](https://github.com/WordPress/gutenberg/commit/ad50f2a89940be47fbd39ed4d16f9907fe50f4a2)
- **Discussion:** [#79921](https://github.com/WordPress/gutenberg/pull/79921) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Media Inserter now automatically refreshes core media categories (Images, Videos, Audio, Attached images) when their underlying attachment queries are invalidated elsewhere in the editor. Previously, uploading or attaching media via the Media Library modal would leave the inserter category stale until manually closed and reopened. This change introduces a `subscribe` capability to core media categories, allowing them to listen to `@wordpress/data` resolution invalidation and trigger a refetch without requiring a full UI reload.

## Impact

- **Block & plugin developers:** No breaking changes. The `subscribe` method is an optional, core-internal capability on media category objects. External sources (e.g., `openverse`) remain unaffected.
- **Site owners & editors:** Improved UX in the block editor; media categories in the inserter now reflect newly uploaded or attached images/videos/audio in real time.
- **No action required** for existing themes, plugins, or custom media sources.

## Technical details

- **Files changed:** `packages/block-editor/src/components/inserter/media-tab/media-panel.js`, `packages/editor/src/components/media-categories/index.js`.
- **New/changed symbols:**
  - `subscribe` method added to core media category objects.
  - `createCoreMediaCategory()` factory in `packages/editor/src/components/media-categories/index.js` wraps `fetch` and attaches a `subscribe` callback.
  - `subscribeToMediaInvalidation()` utility uses `@wordpress/data`'s `subscribe` and `select( coreStore ).hasFinishedResolution( 'getEntityRecords', args )` to detect the resolved → unresolved edge.
- **Behavior change in `MediaCategoryPanel`:** Added a `useEffect` that calls `category.subscribe( () => setRefreshKey( ( key ) => key + 1 ), query )` when `subscribe` is present. The `refreshKey` increment forces the grid to refetch.
- **Gating:** `subscribe` is only attached to categories where `!category.isExternalResource`. External sources (like `openverse`) do not receive the method, preserving the existing behavior.
- **Before/After pattern:**
  *Before:* Categories only implemented `fetch`, `attach`, `detach`, and `invalidate`. Stale state required manual UI interaction.
  *After:* Core categories now implement `subscribe( onChange, query )` which registers a `@wordpress/data` listener. When `invalidateResolution` is called elsewhere, the listener fires, triggering `setRefreshKey` to refetch.

## Contribution

Opened by @andrewserong as part of a stack addressing media library synchronization issues (#76955, #77117, #73085). Merged after review by @talldan, @ntsekouras, and @ramonjd. The implementation relies on a `subscribe` callback pattern to bridge the block-editor package and core-data without exposing internal resolution logic to external sources. The PR landed shortly after the dependent #79844 (which added the underlying cache invalidation) was merged.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
