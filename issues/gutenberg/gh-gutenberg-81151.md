# #81151: Core Data: saveDirtyEntities: improve messaging of server errors

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mcsf
- **Labels:** `[Type] Bug`, `[Package] Core data`, `Backported to WP Core`
- **Merged:** [`000b25a`](https://github.com/WordPress/gutenberg/commit/000b25a404c4c01053d1d0d33c25a426af2f62c6)
- **Discussion:** [#81151](https://github.com/WordPress/gutenberg/pull/81151) · 7 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `saveDirtyEntities` action in the Core Data package now surfaces server-provided validation error messages to users instead of falling back to a generic "Saving failed." notice. This change improves editor UX by ensuring meaningful error feedback is displayed as a snackbar notice when entity saves fail, reducing user confusion during validation errors.

## Impact

- **Plugin & theme developers:** No direct code changes required. If you interact with the `saveDirtyEntities` private action, note that it now returns the aggregate promise and surfaces specific error messages via the notices store instead of swallowing them.
- **Editor users:** Will see precise server validation errors (e.g., "The CSS must not contain </style>.") instead of a generic failure notice.
- **No action required** for standard block/theme development or existing integrations.

## Technical details

Modified `saveDirtyEntities` in `packages/core-data/src/private-actions.js` to improve error extraction and notice rendering:
- Added `throwOnError: true` to `saveEditedEntityRecord` and `saveEditedSiteEntities` calls, followed by `.catch( ensureError )`.
- Replaced the generic fallback with a filtered extraction of `error.message`, passed through `decodeEntities` from `@wordpress/html-entities`.
- Error notices are now rendered as `type: 'snackbar'` with a consistent `id: saveNoticeId`.
- Introduced `ensureError`, a helper that normalizes non-Error throws (plain objects, strings, or objects with `toString`) into standard `Error` instances, preserving the original via `cause`.
- Changed the action to explicitly `return Promise.all( pendingSavedRecords )`, enabling test suites and callers to await the full save cycle.

**Before:**
```js
if ( values.some( ( value ) => typeof value === 'undefined' ) ) {
    registry.dispatch( noticesStore ).createErrorNotice( __( 'Saving failed.' ) );
}
```

**After:**
```js
const errors = values.filter( ( v ) => v instanceof Error );
if ( errors.length ) {
    const firstMessage = errors.find( ( e ) => e.message )?.message;
    registry.dispatch( noticesStore ).createErrorNotice(
        decodeEntities( firstMessage || __( 'Saving failed.' ) ),
        { type: 'snackbar', id: saveNoticeId }
    );
}
```

## Contribution

Opened as a rebase of #77447, with initial implementation by @prachigarg19 and subsequent polishing and test coverage by @mcsf. The commit was merged to `trunk` and backported to the `wp/7.1` branch after a cherry-pick conflict was resolved via PR #81216.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
