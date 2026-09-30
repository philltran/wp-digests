# #82638: Data: Add `getResolutionArgs` to share a resolution across selector calls

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Data`, `[Package] Core data`, `[Package] Block library`
- **Merged:** [`e5636a9`](https://github.com/WordPress/gutenberg/commit/e5636a9825a80814984a8b6869d1d4ea75eabd2c)
- **Discussion:** [#82638](https://github.com/WordPress/gutenberg/pull/82638) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

`@wordpress/data` adds `getResolutionArgs`, an optional property you can set on a resolver. It takes the selector's arguments and returns the cache key to use for resolution. `core-data`'s `canUser` now uses it to drop the action from its key, so `create`, `read`, `update` and `delete` on one resource share a single resolver run and a single OPTIONS request. This fixes a race: `resolveSelect( coreStore ).canUser( 'create', resource )` could resolve to `undefined` instead of a boolean while another action on the same resource was still fetching. The old resolver avoided duplicate requests by returning early, and returning early marked it as finished before any data had arrived.

## Impact

**Plugin & block developers**
- `await resolveSelect( coreStore ).canUser( action, resource, id )` now reliably resolves to a boolean, even when a sibling action for the same resource is already in flight. Suspense-based consumers get the same fix. `useSelect` consumers were never affected, because the selector re-runs once permissions land.
- The public `canUser( action, resource, id )` selector signature is **unchanged**.
- Store authors can use the new `resolver.getResolutionArgs` to make several selector calls share one resolution. Before, the only prior art was `__unstableNormalizeArgs`, which rewrites the selector's arguments and the key together.

**Possible breakage: code that manipulates `canUser` resolution metadata directly**
- The resolution key for `canUser` is now `[ resource, id ]`, not `[ action, resource, id ]`. Metadata **actions** (`startResolution`, `finishResolution`, `finishResolutions`) are not normalized through `getResolutionArgs`. Code that dispatches them for `'canUser'` with an action as the first argument now writes to the wrong key. This is common in tests and prefetch code. Drop the action:
  ```js
  // Before
  dispatch( coreStore ).finishResolution( 'canUser', [ 'read', { kind: 'postType', name: 'wp_navigation' } ] );
  // After
  dispatch( coreStore ).finishResolution( 'canUser', [ { kind: 'postType', name: 'wp_navigation' } ] );
  ```
- Per the new tests, metadata selectors such as `hasFinishedResolution( 'canUser', [ action, resource ] )` still accept the action-bearing form.
- The internal `canUser` resolver thunk now takes `( resource, id )`. Code that dispatched the resolver directly with an action argument (as `canUserEditEntityRecord` used to) must switch to `resolveSelect.canUser( action, … )`.

**Site owners:** no action required.

## Technical details

The `@wordpress/data` part of the diff is truncated, so this paragraph relies on the PR description. `getResolutionArgs` runs after `__unstableNormalizeArgs`. Its return value is the key for resolution metadata state and the resolvers cache, and the arguments passed to `fulfill`, `isFulfilled` and `shouldInvalidate`. The selector itself still gets its original arguments.

**`packages/core-data/src/resolvers.js`**
- `canUser` changes from `( requestedAction, resource, id ) => async ( { dispatch, registry, resolveSelect } )` to `( resource, id ) => async ( { dispatch, resolveSelect } )`. Three things are removed:
  - the `ALLOWED_RESOURCE_ACTIONS.includes( requestedAction )` validation, including its `'… is not a valid action.'` throw;
  - the `hasStartedResolution` loop that returned early when a sibling action was already resolving;
  - the batched `finishResolutions( 'canUser', … )` for sibling actions.

  The resolver now just calls `dispatch.receiveUserPermissions()` with all four permission keys.
- New property:
  ```js
  canUser.getResolutionArgs = ( action, resource, id ) => [ resource, id ];
  ```
- `canUserEditEntityRecord` now calls `await resolveSelect.canUser( 'update', { kind, name, id: recordId } )` instead of `dispatch( canUser( 'update', … ) )`.
- In the `getEntityRecord` and `getEntityRecords` permission prefetch, `finishResolutions( 'canUser', … )` gets one entry per record (`[ { kind, name, id } ]`) instead of four action-prefixed entries. `receiveUserPermissions` still receives all four action keys.

**Tests**
- New `packages/core-data/src/test/can-user.jsdom.test.js` covers:
  - one OPTIONS request for all four actions;
  - each action resolving to its own permission;
  - an in-flight resolution being awaited instead of returning `undefined`;
  - shared `hasFinishedResolution` state across actions;
  - separate resolution per resource ID.
- The Navigation block's `use-navigation-menu` tests now call `startResolution`/`finishResolution` without the action argument.

A `core-data` CHANGELOG bug-fix entry was added.

## Contribution

This closes #72798 and implements an approach @jsnajdr suggested in that issue: let the resolver choose its own cache key, so it no longer has to return early to deduplicate. `__unstableNormalizeArgs` was considered and rejected because it rewrites the selector's arguments and the key together, so it can't drop an argument the selector still needs. After merge, @Mamaduka opened a follow-up PR (#83067) to stabilize the API.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
