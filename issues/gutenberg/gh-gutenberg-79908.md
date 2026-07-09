# #79908: Collaboration: only report changed properties from the default sync config

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Package] Core data`, `[Feature] Real-time Collaboration`
- **Merged:** [`8d7d31b`](https://github.com/WordPress/gutenberg/commit/8d7d31b382ba333dc0c2e2cdadc9dc4b141d12ed)
- **Discussion:** [#79908](https://github.com/WordPress/gutenberg/pull/79908) · 5 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

Fixes a Real-time Collaboration regression where syncing any taxonomy record (e.g., the default category) incorrectly marked it as edited, permanently flagging posts as dirty and silently changing the pre-publish panel button from "Publish" to "Save". The issue occurred because `defaultGetChangesFromCRDTDoc` returned the entire CRDT document map on every sync update without comparing properties against the current edited record. With this fix, only genuinely changed properties are reported as edits, preventing stale objects from incorrectly flagging records as dirty and eliminating browser unload warnings on reload.

## Impact

- **Editor & Plugin/Theme Developers**: No action required. This is a core data sync behavior fix that operates internally within the `@wordpress/core-data` store.
- **Site Owners & Multi-author Contributors**: Resolves an edge case where collaboration traffic could inadvertently mark draft posts as permanently dirty, alter the pre-publish UI label, and trigger browser save prompts on page unload. No configuration or migration steps are needed.

## Technical details

The change modifies `packages/core-data/src/utils/crdt.ts` by updating the signature of `defaultGetChangesFromCRDTDoc(crdtDoc, editedRecord)` to accept an `editedRecord` parameter and filter returned changes against it. Previously, the function immediately returned `getRootMap( crdtDoc, CRDT_RECORD_MAP_KEY ).toJSON()`, which produces fresh object instances for properties like `meta` and `_links`. These unchained objects always triggered edits when dispatched via the raw `EDIT_ENTITY_RECORD` handler, marking resolved taxonomy records as dirty.

**Before:**
```ts
function defaultGetChangesFromCRDTDoc( crdtDoc: CRDTDoc ): ObjectData {
  return getRootMap( crdtDoc, CRDT_RECORD_MAP_KEY ).toJSON();
}
```

**After:**
```ts
function defaultGetChangesFromCRDTDoc(
  crdtDoc: CRDTDoc,
  editedRecord: ObjectData
): ObjectData {
  const docRecord = getRootMap( crdtDoc, CRDT_RECORD_MAP_KEY ).toJSON();
  return Object.fromEntries(
    Object.entries( docRecord ).filter( ( [ key, newValue ] ) =>
      haveValuesChanged( editedRecord?.[ key ], newValue )
    )
  );
}
```

The fix reuses the `haveValuesChanged` deep-equality pattern already present in `getPostChangesFromCRDTDoc`. A new e2e spec (`collaboration-taxonomy-record-dirty.spec.ts`) validates that relaying unchanged sync messages across multiple tabs no longer triggers `hasEditsForEntityRecord('taxonomy/category', 1)` and preserves the "Publish" button label.

## Contribution

Opened by @adamsilverstein to resolve a regression where CRDT sync updates incorrectly flagged taxonomy records as dirty. Co-authored with @adamziel, @maxschmeling, and @alecgeatches during refinement. Merged into `trunk` and cherrypicked to `release/23.5`. The approach focused on applying the existing deep-comparison logic from the post-specific sync config to the default taxonomy config rather than introducing a broader architecture change.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
