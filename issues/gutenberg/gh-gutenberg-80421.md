# #80421: [7.1 Backport] -- Worker threads: reject pending RPC calls on worker failure or termination 

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Feature] Client Side Media`
- **Merged:** [`0214a36`](https://github.com/WordPress/gutenberg/commit/0214a36027145f560e467217f4d6b0bcef0dc3b2)
- **Discussion:** [#80421](https://github.com/WordPress/gutenberg/pull/80421) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

When the vips or video conversion worker crashes or is terminated, pending RPC promises previously stayed pending forever, leaving uploads stuck in the queue with no error surfaced. This change tracks pending call rejecters and rejects them immediately when the worker emits an `error`, `messageerror`, or is explicitly terminated. It also makes worker cancellation calls best-effort so a crashed worker cannot block the rest of the cancellation flow.

## Impact

- **Plugin & theme developers / Core editors:** Uploads using client-side media processing will no longer hang indefinitely if the background worker fails. Errors will properly surface to the user instead of leaving items stuck in the queue.
- **Hosting & platform teams:** No configuration changes required. The fix is contained within the `@wordpress/worker-threads` and `@wordpress/upload-media` packages.
- **Action required:** None for most users. Developers who previously implemented polling or forced queue resets to work around stuck uploads can remove those workarounds.

## Technical details

In `packages/worker-threads/src/main-thread.ts`, the `wrap()` function now maintains a `WorkerState` object via a `remoteStates` WeakMap instead of a simple `remoteWorkers` map. It attaches `error` and `messageerror` listeners to the underlying `Worker`. When either fires, `failWorker()` iterates through a `pendingRejects` set, rejects each promise with the failure reason, and clears the set. Subsequent method calls check `state.failure` and reject immediately. The `terminate()` function now calls `failWorker()` before calling `worker.terminate()`.

In `packages/upload-media/src/store/actions.ts`, the `cancelItem()` action now wraps worker cancellation calls with `.catch( () => {} )` to prevent a crashed worker from halting the cancellation flow:

```ts
// Before
await vipsCancelOperations( id );
await cancelGifToVideoOperations( id );

// After
await vipsCancelOperations( id ).catch( () => {} );
await cancelGifToVideoOperations( id ).catch( () => {} );
```

Unit tests verify that pending calls reject on `error`/`messageerror`/`terminate()`, that already-settled calls are unaffected by later failures, and that `cancelItem` still triggers `onError` callbacks and removes items when worker cancellation rejects.

## Contribution

Opened as a backport of trunk PR #79955 to the `wp/7.1` branch. The automated cherry-pick initially failed due to a `CHANGELOG.md` conflict, which was resolved by manually merging the changelog entries. @t-hamano and @adamsilverstein coordinated the stacking order with PR #80420 to ensure clean application on the release branch, ultimately merging the backport first so the dependent cherry-pick applied cleanly. The diff was verified to be byte-identical to trunk for all source files, with 392 unit tests passing.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
