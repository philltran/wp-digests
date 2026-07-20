# #77100: Core Data: Cleanup edits matching persisted record on undo/redo

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Package] Core data`, `[Feature] History`
- **Merged:** [`d11fa53`](https://github.com/WordPress/gutenberg/commit/d11fa53c908e5c8c47192efe61e976cc7ecbb9ce)
- **Discussion:** [#77100](https://github.com/WordPress/gutenberg/pull/77100) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a bug in the Core Data history system where fully undoing changes left an entity record marked as dirty, keeping the editor’s Save button enabled. The `withMultiEntityRecordEdits` reducer now compares each undo/redo value against the persisted record and clears matching edits, ensuring the dirty state accurately reflects unsaved changes. This prevents false-positive save prompts and aligns the undo/redo behavior with the expected editor state.

## Impact

- **Plugin & theme developers / block authors:** No action required. The Core Data history system now correctly clears the dirty state when all edits are undone.
- **Editor users:** The Save button will correctly disable after fully undoing changes to a post or page.
- **Real-time collaboration (RTC) users:** No change in this PR; RTC bypasses `withMultiEntityRecordEdits` and retains the dirty-state bug until a separate fix lands.
- **No breaking changes, deprecations, or migration steps.**

## Technical details

The diff introduces a shared utility `clearUnchangedEdits` (`packages/core-data/src/utils/clear-unchanged-edits.ts`) that uses `fastDeepEqual` to compare edit values against their persisted counterparts, returning `undefined` for matches so the edits reducer drops them. This utility replaces inline comparison logic in two places:

- `packages/core-data/src/actions.js`: The `editEntityRecord` thunk now calls `clearUnchangedEdits( editsWithMerges, record )` instead of manually reducing over `Object.keys( edits )`.
- `packages/core-data/src/reducer.js`: The `withMultiEntityRecordEdits` higher-order reducer now extracts the persisted record from `state?.queriedData?.items?.default?.[ recordId ]`, maps undo/redo `changes` to an `edits` object, and passes it to `clearUnchangedEdits( edits, persistedRecord )` before dispatching `EDIT_ENTITY_RECORD`.

Before/after pattern in the reducer:
```javascript
// Before
edits: Object.entries( changes ).reduce(
  ( acc, [ key, value ] ) => {
    acc[ key ] = action.type === 'UNDO' ? value.from : value.to;
    return acc;
  },
  {}
),

// After
const persistedRecord = state?.queriedData?.items?.default?.[ recordId ];
const edits = Object.fromEntries(
  Object.entries( changes ).map( ( [ key, value ] ) => [
    key,
    action.type === 'UNDO' ? value.from : value.to,
  ] )
);
edits: clearUnchangedEdits( edits, persistedRecord ),
```

The utility also unwraps `raw` subfields from persisted values (`persistedRecord[ key ]?.raw ?? persistedRecord[ key ]`) to ensure accurate comparison. Unit tests verify deep equality and `undefined` assignment, and an e2e test confirms the Save button disables after full undo.

## Contribution

Opened and merged by @Mamaduka, with co-authors including @youknowriad, @tyxla, @jsnajdr, and others. The author initially discussed the issue with @tyxla at WCEU and decided to ship the dirty-state fix separately from real-time collaboration (RTC) changes to avoid blocking users not using RTC. During review, @jsnajdr noted duplication between the thunk and reducer, prompting the extraction of `clearUnchangedEdits` as a shared utility. The RTC path was explicitly deferred to a follow-up PR (#79213) pending internal review.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
