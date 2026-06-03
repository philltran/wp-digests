# #78508: Edit post: consume preload cache before React mount

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Performance`, `[Package] API fetch`, `[Package] Editor`, `[Package] Edit Post`, `[Package] Private APIs`
- **Merged:** [`ed16bf4`](https://github.com/WordPress/gutenberg/commit/ed16bf45789d999b434f389e8288833613f1e393)
- **Discussion:** [#78508](https://github.com/WordPress/gutenberg/pull/78508) · 6 comments · 0 reactions

## Summary

The Edit Post editor now consumes the API fetch preload cache synchronously before the React application mounts, eliminating intermediate loading states and preventing stale data leaks across navigations. Preloaded resolutions are applied to the store before any `useSelect` subscribers attach, and the cache is immediately purged after first load to ensure subsequent route changes do not read cross-contaminated data.

## Impact

- **Editor & Plugin/Theme Developers:** No breaking API changes or direct migration required. The preload lifecycle is now stricter, but external code interacting with `wp.data` stores will continue to function.
- **Performance:** Reduces render-blocking waits in the post editor by ensuring preloaded data is available at mount time, avoiding an additional resolution cycle.
- **Debugging & Testing:** Cache is cleared immediately after first consumption; unconsumed entries now emit a console warning listing the stale routes. E2E tests will surface failures if `preloadResolutions` trigger network requests on mount or leave unused cache behind.
- **No action required** for standard plugin/theme authors not manipulating the editor's internal data flow.

## Technical details

- Shifts preload cache hydration from a reactive, post-mount subscriber pattern to a synchronous pre-render step. Store subscribers are attached after `wp.apiFetch` has already hydrated the state tree with preloaded responses.
- Unifies resolver caching: multiple resolvers requesting the same path (e.g., `/wp/settings`) now share a single cached resolution instead of maintaining isolated copies.
- Enforces strict cache lifecycle on mount: `clearPreloadedData` executes immediately after initial render. Any paths remaining in the cache trigger a console warning detailing the unconsumed routes, flagging redundant server-side preloads.
- Affects core build output in `packages/api-fetch/index.min.js`, `packages/editor/index.min.js`, and `packages/private-apis/index.min.js`. The diff reflects ~480 B growth in `api-fetch` and ~410 B in `edit-post` due to tighter lifecycle guards and warning logging.
- Pseudocode behavior shift:
  ```js
  // Before: Post-mount resolution could trigger secondary reads or stale merges.
  render( <Editor /> );
  store.select( 'core/editor' ).getEditedPost(); // subscribers attach, may miss preload

  // After: Preload applied before render; cache purged immediately after.
  hydratePreloadedToStore( preloadCache ); // pre-mount sync hydration
  render( <Editor /> );
  clearPreloadedCache(); // immediate cleanup + warning for leftovers
  ```

## Contribution

Authored by `@ellatrix` and merged in commit `ed16bf4`. The design iteration focused on preventing intermediate loading states and enforcing cache hygiene across editor routes. `@jsnajdr` contributed architectural direction around dev-mode reporting to detect redundant `preloadResolutions` and unused cache, which directly shaped the decision to enforce immediate cache clearing and add warning logs. Test coverage adjustments were proposed to verify zero network requests on mount and fail CI if preload entries remain unconsumed.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
