# #82553: API Fetch: Expose defaultFetchHandler so overrides can restore it

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] API fetch`
- **Merged:** [`c24bc33`](https://github.com/WordPress/gutenberg/commit/c24bc33faa78a80f67ced592ebc1ead850c6cdfd)
- **Discussion:** [#82553](https://github.com/WordPress/gutenberg/pull/82553) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`@wordpress/api-fetch` now exposes its built-in `window.fetch`-based handler as `apiFetch.defaultFetchHandler`. Previously `setFetchHandler()` overwrote the module-local `fetchHandler` and the default was never exported, so an override could not be undone. Consumers can now restore the original behavior with `apiFetch.setFetchHandler( apiFetch.defaultFetchHandler )`.

## Impact

- **Plugin & theme developers:** Purely additive. Code that temporarily replaces the fetch handler (tests, mocks, offline/alternate transports, debugging) can now restore the default without reimplementing it.
- **Site owners / hosting / headless consumers:** No action required; no behavior change unless code opts in to the new property.
- No deprecations, removals, or migrations. The bundle size of `build/scripts/api-fetch/index.min.js` grows by about 7 B.

## Technical details

In `packages/api-fetch/src/index.ts`:

- The existing `defaultFetchHandler` constant (typed `FetchHandler`) gains a JSDoc comment; its implementation is unchanged.
- The `ApiFetch` interface gains `defaultFetchHandler: FetchHandler`.
- The property is attached to the exported function alongside the other statics: `apiFetch.defaultFetchHandler = defaultFetchHandler;`.

```js
// Before: override was one-way
apiFetch.setFetchHandler( myHandler );

// After: restore the default
apiFetch.setFetchHandler( apiFetch.defaultFetchHandler );
```

The README gains a short example of restoring the default, and `packages/api-fetch/CHANGELOG.md` gets a New Features entry. The same changelog edit also adds a trailing period to the preceding #82408 entry. The diff contains no test changes; the PR describes a manual console check using `wp.apiFetch`.

## Contribution

Follow-up to an earlier api-fetch PR (#82408, which added `apiFetch.unregister` and `apiFetch.httpV1Middleware`), where the need to restore the default handler came up in review. Authored by @Mamaduka, with @manzoorwanijk credited in the props bot's co-author list; the PR notes it was assisted by Claude. The discussion contains only bot comments.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
