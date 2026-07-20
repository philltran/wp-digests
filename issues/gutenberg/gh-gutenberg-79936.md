# #79936: RTC: Preserve collaborators' unsaved edits when another user saves

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @shekharnwagh
- **Labels:** `[Type] Bug`, `[Package] Core data`, `[Feature] Real-time Collaboration`
- **Merged:** [`3e47c3e`](https://github.com/WordPress/gutenberg/commit/3e47c3e74f9affe576ef0cbadab85ae5d2ae2643)
- **Discussion:** [#79936](https://github.com/WordPress/gutenberg/pull/79936) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Real-time collaboration (RTC) now preserves collaborators' unsaved edits when another user triggers a save. Previously, every save pushed the entire server-returned record into the Yjs CRDT document, causing stale scalar fields (like `template`, `status`, `slug`, `title`, `excerpt`) and post meta to overwrite peers' local edits. The fix filters the save response to sync only fields the server actually mutated, preventing race conditions between saves and collaborative edits.

## Impact

- **Plugin & theme developers / RTC users:** No immediate action required. The change is internal to the `@wordpress/core-data` package and transparently improves collaboration reliability.
- **Direct `saveEntityRecord()` callers:** Behavior is now consistent with `editEntityRecord()`; direct saves are pushed into the CRDT before persistence, ensuring programmatic updates don't get reverted by subsequent collaborative edits.
- **No breaking changes or deprecations.** Existing code continues to work without modification.

## Technical details

The diff modifies `packages/core-data/src/actions.js` to introduce `getServerMutatedFields()` and `getServerMutatedMetaFields()`, which diff the REST save response against the persisted record and in-flight CRDT edits using `fastDeepEqual`. When `saveEntityRecord()` is called on an existing record with `syncConfig`, the payload is now pushed into the CRDT via `getSyncManager()?.update()` *before* `__unstablePrePersist` hooks and the API fetch. After the save completes, only server-mutated fields are passed to `SyncManager#update` with `LOCAL_UNDO_IGNORED_ORIGIN`. Post meta is compared per subkey to avoid reintroducing stale siblings, and raw/rendered attribute shapes are normalized via the newly exported `getRawValue()` from `packages/core-data/src/utils/crdt.ts`. New records or saves without a persisted baseline fall back to pushing the full response.

## Contribution

Opened and merged by @shekharnwagh, with co-authors @alecgeatches and @t-hamano. Review by @alecgeatches identified two gaps: direct `saveEntityRecord()` calls bypassing the CRDT, and post meta being overwritten due to object-level comparison. The author addressed both in subsequent commits, adopting a persisted-meta baseline and subkey-level diffing. The team confirmed RTC is not targeting WordPress 7.1 core, so no backport labels were applied.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
