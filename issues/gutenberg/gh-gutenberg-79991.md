# #79991: RTC: Only apply CRDT updates synchronously when collaborating

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Performance`, `[Package] Core data`, `[Feature] Real-time Collaboration`, `[Package] Sync`
- **Merged:** [`202cea7`](https://github.com/WordPress/gutenberg/commit/202cea7397483bb35c537906d392d19a1fd59f17)
- **Discussion:** [#79991](https://github.com/WordPress/gutenberg/pull/79991) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Real-time collaboration (RTC) in the block editor now defers CRDT document updates off the typing hot path when no remote peers are present, restoring pre-regression typing latency. Previously, every keystroke triggered a synchronous CRDT commit regardless of collaboration state, causing a ~75% performance drop. The change ensures synchronous commits only occur when remote peers are active to prevent race conditions, while solo editing remains fast.

## Impact

- **Block editor & plugin developers:** No immediate action required for standard usage. The `SyncManager.update` behavior is now context-aware and automatically optimizes based on peer presence.
- **Core-data & sync consumers:** `SyncManager.createPersistedCRDTDoc` is now asynchronous and returns a `Promise<string | null>`. Any code that calls or chains this method must handle the promise.
- **Hosting & platform teams:** No configuration or migration steps required. The change is internal to the editor's sync layer.

## Technical details

The diff modifies `packages/sync/src/manager.ts` to introduce `updateOrDefer`, which checks `getAwareness( objectType, objectId )?.getStates().size > 1` to determine if remote peers are present. When collaborating, it calls `updateCRDTDoc` synchronously; otherwise, it uses `deferUpdateCRDTDoc` (wrapped via `yieldToEventLoop`) to defer work to the next event loop tick. `createPersistedCRDTDoc` is converted to an `async` function that awaits a zero-delay `setTimeout` to flush deferred updates before serializing the `Y.Doc`. Callers in `packages/core-data/src/entities.js` now `await` this method:

```js
// Before
const serializedDoc = getSyncManager()?.createPersistedCRDTDoc( objectType, objectId );
// After
const serializedDoc = await getSyncManager()?.createPersistedCRDTDoc( objectType, objectId );
```

Type definitions in `packages/sync/src/types.ts` reflect the new `Promise<string | null>` return type and allow `ObjectID | null` for `getAwareness`.

## Contribution

Opened and merged by @Mamaduka (co-authored with @chriszarate) to address a typing latency regression introduced in #78756. @alecgeatches verified the fix restored ~15ms latency for solo editing while preserving the synchronous merge behavior required for multi-user collaboration. The approach was validated against the original race-condition reproduction before merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
