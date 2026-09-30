# #82408: API Fetch: Add unregister to remove a registered middleware

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] API fetch`
- **Merged:** [`44cfc07`](https://github.com/WordPress/gutenberg/commit/44cfc075453376a10bccc6b8a569cbdc850c362b)
- **Discussion:** [#82408](https://github.com/WordPress/gutenberg/pull/82408) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

`@wordpress/api-fetch` gains `apiFetch.unregister( middleware )`, which removes a previously registered middleware by reference and returns a boolean indicating whether it was registered. The built-in `httpV1Middleware` is now exposed as `apiFetch.httpV1Middleware`, so consumers can opt out of the `X-HTTP-Method-Override` behavior that rewrites `PATCH`, `PUT` and `DELETE` requests as `POST`. Previously, middleware registered via `apiFetch.use()` could never be removed.

## Impact

**Plugin & theme developers**
- New public API: `apiFetch.unregister( middleware )` (returns `boolean`) and `apiFetch.httpV1Middleware`. Purely additive; no deprecations or removals.
- Calling `apiFetch.unregister( apiFetch.httpV1Middleware )` makes `PATCH`, `PUT` and `DELETE` requests go out with their real method instead of `POST` plus an `X-HTTP-Method-Override` header.
- Removal is **global**: every `apiFetch` caller on the page loses the middleware. The README warns that `httpV1Middleware` exists for servers and firewalls that reject those methods, so removing it can break saving on such hosts. Only remove middleware you registered yourself, or a built-in the whole page can do without.
- `userLocaleMiddleware` and `namespaceEndpointMiddleware` are deliberately not exposed, because removing them can break the app.

**Hosting & platform**
- The change makes it possible to test whether the method-override shim is still needed in a given environment; it does not change default behavior.

**Site owners / headless consumers**
- No action required. Default behavior is unchanged.

## Technical details

All changes are in `packages/api-fetch/`.

- `src/index.ts`: adds `unregisterMiddleware( middleware )`, which does `middlewares.indexOf( middleware )`, returns `false` if not found, otherwise `splice`s it out and returns `true`. Matching is by identity.
- The `ApiFetch` interface gains `unregister: ( middleware: APIFetchMiddleware ) => boolean` and `httpV1Middleware: typeof httpV1Middleware`. These are assigned as `apiFetch.unregister = unregisterMiddleware;` and `apiFetch.httpV1Middleware = httpV1Middleware;`.
- Note that `use()` does `middlewares.unshift()`, so the list is shared module state and removal affects all callers.
- Tests in `src/test/index.jsdom.test.js` cover: a custom middleware no longer being called after unregistering; the boolean return value across register/unregister/unregister; and a `DELETE` request first being sent as `POST` with `X-HTTP-Method-Override: DELETE`, then as a real `DELETE` without the header after `httpV1Middleware` is unregistered.
- README gains a "Removing middlewares" section, and CHANGELOG gets a New Features entry.

```js
import apiFetch from '@wordpress/api-fetch';

// Send DELETE as DELETE, not POST + X-HTTP-Method-Override
apiFetch.unregister( apiFetch.httpV1Middleware );
```

Bundle size impact is +48 B on `build/scripts/api-fetch/index.min.js`.

## Contribution

This PR was presented as an alternative to earlier attempts (#70506 and #67919) and closes long-standing issues #16805 and #67655. The stated rationale is that `httpV1Middleware` has been unconditional since it moved into the package (#7018), descending from a 2018 shim for hosts rejecting `PUT` (#5741), and has already caused a bug (#76284). During review, @jsnajdr pointed out that `setFetchHandler` is similarly irreversible because `defaultFetchHandler` isn't exported; @Mamaduka split that into follow-up #82553. The PR notes it was assisted by Claude.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
