# #79955: Worker threads: reject pending RPC calls on worker failure or termination

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Feature] Client Side Media`, `Backport to WP 7.1 Beta/RC`
- **Merged:** [`70f9a95`](https://github.com/WordPress/gutenberg/commit/70f9a9594d6532301fa31c3313f38275c60d26b9)
- **Discussion:** [#79955](https://github.com/WordPress/gutenberg/pull/79955) · 9 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Fixes a bug in @wordpress/worker-threads where RPC calls to Web Workers (used by @wordpress/vips and video conversion) would hang indefinitely if the worker crashed, failed to initialize, or was terminated. The change ensures pending promises reject immediately, allowing the upload queue to surface errors, release the editor's save lock, and recover gracefully.

## Impact

- **Plugin & theme developers / block authors:** No direct API changes, but client-side media processing will now fail gracefully instead of leaving blocks stuck on spinners or posts permanently unsaveable.
- **Hosting & platform teams:** Reduces memory leaks and stuck editor states caused by orphaned worker promises during high-concurrency media processing.
- **No action required** for existing code, but developers relying on synchronous worker behavior or custom RPC wrappers should be aware that `terminate()` and worker crashes now explicitly reject pending calls.

## Technical details

In `packages/worker-threads/src/main-thread.ts`, `wrap()` now maintains a `WorkerState` object containing a `pendingRejects` set. When `terminate()` is called, or when the worker emits `error` or `messageerror`, `failWorker()` iterates through `pendingRejects` and rejects each promise. Subsequent method calls on a failed remote immediately return `Promise.reject( state.failure )`.

In `packages/upload-media/src/store/actions.ts`, `cancelItem()` wraps worker cancel calls to prevent the cancellation flow from aborting when the worker is dead:

```diff
-		await vipsCancelOperations( id );
+		await vipsCancelOperations( id ).catch( () => {} );
```

This bridges a gap in the `comctx` RPC layer, which previously only settled promises on response messages, by explicitly tracking and rejecting in-flight calls on worker lifecycle events.

## Contribution

Opened and merged by @adamsilverstein, with co-authors @ramonjd and @andrewserong. The PR addresses #79953, where vips worker crashes left the upload queue and editor save lock permanently stuck. Review focused on ensuring the cancellation flow in @wordpress/upload-media remained resilient when worker calls rejected. A follow-up timeout mechanism for WASM deadlocks was explicitly deferred to keep the PR focused. The change was initially missed for the WP 7.1 backport and later stacked on #80420.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
