# #79478: DataFormPostSummary: fix different `useSelect` returned values 

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Bug`, `[Package] Editor`
- **Merged:** [`472b417`](https://github.com/WordPress/gutenberg/commit/472b417db435a1f6303900c2ce48245d35222312)
- **Discussion:** [#79478](https://github.com/WordPress/gutenberg/pull/79478) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request resolves a `useSelect` stability warning in the `DataFormPostSummary` component by consolidating data fetching and stabilizing returned object keys. Previously, `availableTemplates` was fetched in a separate `useSelect` call alongside entity records, causing key mismatches (`entityData` vs `entityIds`) across renders. The fix merges `availableTemplates` into the initial `useSelect`, flattens the entity record fetch into a single `records` object, and dynamically resolves entity IDs during save operations. This eliminates the console warning without altering the component's external behavior.

## Impact

- **Plugin & theme developers:** No action required. This is an internal Editor component fix.
- **Site owners:** No action required.
- **Headless & REST consumers:** No impact.
- **Note:** The change stabilizes the Editor Inspector's DataForm experiment but does not expose new public APIs or change existing block/editor interfaces.

## Technical details

The diff modifies `packages/editor/src/components/sidebar/dataform-post-summary.js`.
- Consolidates `availableTemplates` fetching into the first `useSelect` call alongside `postType`, `postId`, and `isPostStatusRemoved`, ensuring consistent return keys.
- Flattens the second `useSelect` (previously returning `{ entityData, entityIds, availableTemplates }`) to return a single `records` object keyed by namespace.
- Replaces the static `entityIds[ key ]` lookup during `editEntityRecord` with a dynamic resolution using `useRegistry()` and `entity.getId( registry.select )`, ensuring the correct ID is passed regardless of whether the entity is a singleton (e.g., `root/site`).
- Removes the early return in the `data` `useMemo` when supplementary data is empty, simplifying the merge logic.

Before/after pattern for ID resolution:
```js
// Before
editEntityRecord( entity.kind, entity.name, entityIds[ key ], value );
// After
const id = entity.getId ? entity.getId( registry.select ) : undefined;
editEntityRecord( entity.kind, entity.name, id, value );
```

## Contribution

The discussion centered on the architectural complexity of the multi-entity data fetching pattern. The author clarified that the current approach is a temporary workaround to satisfy `shallowEqual` checks in `useSelect` while waiting for a proper core API for multi-entity forms. The reviewer acknowledged the complexity but approved the stabilization fix, noting it would be addressed in future extensibility work.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
