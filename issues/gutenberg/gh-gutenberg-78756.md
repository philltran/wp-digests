# #78756: RTC: Fix CRDT deferred updates resulting in jumbled typing

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @alecgeatches
- **Labels:** `[Type] Bug`, `[Package] Core data`, `[Feature] Real-time Collaboration`, `[Package] Sync`
- **Merged:** [`6a5fd9b`](https://github.com/WordPress/gutenberg/commit/6a5fd9b1a7473ff383fc4e4651c90295b018575a)
- **Discussion:** [#78756](https://github.com/WordPress/gutenberg/pull/78756) · 6 comments · 0 reactions

## Summary

Fixes a race condition in Real-Time Collaboration (RTC) where rapid concurrent typing over WebSockets with slight network delays caused characters to be dropped or cursors to misalign. The fix synchronously commits local editor changes to the underlying `Y.Doc` before remote CRDT updates merge, preventing stale block snapshots from overwriting uncommitted keystrokes. Additionally, it narrows a `clientId` preservation rule to code editors only, ensuring visual editor selection history stays aligned with the shared document state.

## Impact

- **RTC & WebSocket editors:** Resolves dropped characters and cursor drift during concurrent editing sessions.
- **Plugin/Theme developers & Hosts:** No breaking changes or external API modifications. The internal sync timing shift (event-loop deferred → synchronous) is scoped to `@wordpress/sync` internals.
- **Action required:** None for standard usage. Developers actively building or customizing RTC/WebSocket sync should verify cursor placement and selection behavior under concurrent editing; no migration steps needed.

## Technical details

- **`SyncManager.update()` timing:** Removes the `yieldToEventLoop()` wrapper around local CRDT updates, enforcing synchronous application to the local `Y.Doc`. This closes the window where a remote update could merge into an uncommitted document and overwrite recent keystrokes.
- **`clientId` preservation narrowing:** Restores standard `clientId` updates for normal visual block merges to prevent selection history drift in `applyPostChangesToCRDTDoc()`. The previous behavior (skipping `clientId` for `content`-only updates) is preserved only for the Code Editor path, where reparsing generates fresh random IDs.
- **Serialization timing:** `createPersistedCRDTDoc()` now serializes the `Y.Doc` immediately since content updates are no longer deferred.
- **Deferral preservation:** Selection/undo ordering remains explicitly deferred via `setTimeout(..., 0)` in `applyPostChangesToCRDTDoc()`, keeping visual editor history behavior intact.
- **Testing infrastructure:** Adds a `npm run rtc:ws:slow` helper that injects a 100ms delay into the local WebSocket server for reproducing the race condition.

## Contribution

Opened and merged by `@alecgeatches` (commit `6a5fd9b`). The PR resolves a sync race condition exposed during WebSocket-based RTC development, building on earlier `clientId` handling from PR #78483. Review notes emphasized that synchronous CRDT updates must be paired with targeted `clientId` restoration to keep cursor context accurate across merges. A local `npm run rtc:ws:slow` test runner was added to simulate network delays, with a note that enabling the updated e2e tests in CI will be handled in a subsequent PR.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
