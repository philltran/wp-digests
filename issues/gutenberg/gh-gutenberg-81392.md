# #81392: Core Data: Fix post staying dirty after saving edited post meta with RTC enabled

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @chriszarate
- **Labels:** `[Type] Bug`, `[Package] Core data`, `[Feature] Real-time Collaboration`
- **Merged:** [`79e0ed0`](https://github.com/WordPress/gutenberg/commit/79e0ed06933d8f6d78c988ef08a639bf1c4933ca)
- **Discussion:** [#81392](https://github.com/WordPress/gutenberg/pull/81392) · 13 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

With real-time collaboration (RTC) enabled, a post stayed dirty after a successful save whenever `meta` had been edited: the Save button stayed active and repeated saves never cleared it. `saveEntityRecord` now passes the original, pre-`__unstablePrePersist` edits to `receiveEntityRecords` as `persistedEdits`, instead of the augmented request payload. The post-save prune of state edits can now match and clear the meta edit.

## Impact

- **Site owners / editors:** With the collaborative editing experiment on, posts with meta edits (footnotes, or plugin-sidebar meta such as ActivityPub) now return to a clean "Saved" state after saving. No more phantom unsaved-changes state or leave-page prompts.
- **Plugin & theme developers:** Plugins that write post meta from the editor session, or mutate meta server-side during save, are no longer stuck dirty under RTC. No code changes required.
- **Code touching `RECEIVE_ITEMS`:** The `persistedEdits` value dispatched with `receiveEntityRecords` from `saveEntityRecord` is now the original edits rather than the `__unstablePrePersist`-transformed payload. Third-party reducers or middleware that inspect that action payload may see different values. A reviewer raised this as a possible compatibility concern; the thread records no resolution.
- **Hosting / headless:** No REST, schema, or DB changes.

## Technical details

The bug was a mismatch between edits held in state and the edits used to prune them after a save.

1. A meta edit snapshots the full edited meta object into state edits (`mergedEdits: { meta: true }`), including the load-time `meta._crdt_document`.
2. On save, `prePersistPostType` injects a freshly serialized CRDT snapshot into `meta._crdt_document` in the request payload, so it always differs from the load-time value.
3. `saveEntityRecord` passed that augmented payload as `persistedEdits`. The `edits` reducer compares `meta` as one atomic object and clears a state edit only if it deep-equals the response value or the persisted edits. The state's `edits.meta` matched neither, so the meta edit survived every save.

The same mismatch also affected the auto-draft flow, where `prePersistPostType` rewrites an `Auto Draft` title to an empty string.

The fix is a one-argument change in `packages/core-data/src/actions.js`. It replaces `edits` with `record`, the edits captured before `__unstablePrePersist` ran:

```js
dispatch.receiveEntityRecords(
	kind,
	name,
	updatedRecord,
	undefined,
	true,
	record // was: edits
);
```

The request body sent to the server (`data: edits`) is unchanged, so the augmented CRDT snapshot is still persisted. Values injected by `__unstablePrePersist` never exist in state edits, so they need not take part in the comparison. The response record, also part of the prune comparison, still reflects them.

Tests added:
- A unit test in `packages/core-data/src/test/actions.js` asserts `receiveEntityRecords` receives the original edits while `apiFetch` receives the augmented payload.
- An e2e spec, `collaboration-save-clean-state.spec.ts`, edits meta (footnotes), saves with the keyboard, and asserts `isEditedPostDirty()` is false and the top bar shows "Saved".

A CHANGELOG entry was added to `packages/core-data`.

## Contribution

@chriszarate authored the fix with Claude Code assistance, as disclosed in the PR. @youknowriad asked for an e2e test in addition to the unit test, and one was added. @mcsf asked whether changing the `RECEIVE_ITEMS` payload could break third-party state handling, while saying he liked the client never seeing server-bound augmentations. @Mamaduka found that the same issue was being addressed in #77611 and #81372 and ensured props reached the earlier contributors. @youknowriad also reported that a draft post with a single footnote still appeared dirty after save in his testing and asked whether others could reproduce it. The thread shows no answer to that.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
