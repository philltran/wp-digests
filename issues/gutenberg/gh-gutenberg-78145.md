# #78145: Fix: Disable collab sync when incompatible meta boxes are present.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @DarkMatter-999
- **Labels:** `[Package] Core data`, `[Feature] Real-time Collaboration`, `[Package] Sync`
- **Merged:** [`de0dde6`](https://github.com/WordPress/gutenberg/commit/de0dde644b0691f029509161afdeaafa868a53bf)
- **Discussion:** [#78145](https://github.com/WordPress/gutenberg/pull/78145) · 9 comments · 0 reactions

## Summary

When the Block Editor detects an incompatible legacy meta box, Real-Time Collaboration (RTC) sync is now fully torn down instead of running silently in the background. Previously, toggling `isCollaborationSupported` to false only updated the UI flag; the underlying SyncManager continued processing edits and peer connections. This caused silent disconnections when users hit the `MAX_PEERS` limit, while suppressed error modals masked the failure and risked data loss.

## Impact

- **Block Editor & Plugin/Theme Developers**: No migration or code changes required. Legacy meta boxes that previously triggered a partial RTC state now receive complete sync teardown and proper connection error feedback.
- **RTC & Sync Consumers**: Eliminates "half-enabled" states where background WebSocket/Peer connections outlasted UI expectations, ensuring `setCollaborationSupported( false )` reliably stops all entity syncing.
- **Note**: The post-lock holder UX gap for detecting cross-device editing remains unresolved and is deferred to a future iteration. No action required now.

## Technical details

- `packages/core-data/src/private-actions.js`: The `setCollaborationSupported` action creator now calls `getSyncManager()?.unloadAll()` when `supported` is `false`.
- `packages/sync/src/manager.ts`: Introduces `unloadAll()` to iterate over `entityStates` and `collectionStates`, invoking `unload()` on each and clearing the maps. Adds race-condition guards (`isEntityUnloaded`, `hasObserversAttached`) to `loadEntity` and `loadCollection`; if `unload()` runs during the async provider creation, newly created providers are destroyed and Yjs document initialization is aborted.
- `packages/core-data/src/sync.ts`: Exports a new `hasSyncManager()` guard to prevent accidental SyncManager bootstrapping when only teardown is intended.
- `packages/sync/src/types.ts`: Adds `unloadAll: () => void` to the `SyncManager` interface.

**Before/After Action Creator**
```js
// Before: UI flag set, sync kept running
export const setCollaborationSupported = ( supported ) =>
	( { dispatch } ) => {
		dispatch( { type: 'SET_COLLABORATION_SUPPORTED', supported } );
};

// After: Full teardown triggered when disabled
export const setCollaborationSupported = ( supported ) =>
	( { dispatch } ) => {
		dispatch( { type: 'SET_COLLABORATION_SUPPORTED', supported } );
		if ( ! supported && hasSyncManager() ) {
			getSyncManager().unloadAll();
		}
};
```

## Contribution

PR #78145 was authored by @DarkMatter-999 to resolve issue #78093. Review from @alecgeatches validated the core sync teardown logic but highlighted a lingering post-lock holder UX gap for cross-device editing; this was explicitly deferred to a separate follow-up PR after the merge candidate was deemed solid enough. Minor guards and a race-condition test were merged inline, and the PR was approved and merged with co-authors @alecgeatches and @mmtr.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
