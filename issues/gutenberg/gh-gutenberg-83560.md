# #83560: Media editor: show an error when saving attachment details fails

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Feature] Media`
- **Merged:** [`fdadc72`](https://github.com/WordPress/gutenberg/commit/fdadc72c787af1e95a1d05eca9f5fa0cad6abe24)
- **Discussion:** [#83560](https://github.com/WordPress/gutenberg/pull/83560) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The media editor's save hook silently swallowed REST errors when saving attachment details, causing the editor to close as if the save had succeeded. The fix passes `{ throwOnError: true }` to `saveEditedEntityRecord` so that a failed save rejects the promise, triggering the hook's existing error-handling path: an error snackbar is shown and the editor stays open with the user's unsaved changes intact.

## Impact

- **Site owners / editors:** No code change required. If a save fails (network drop, transient server error, permission issue), the media editor now displays an error snackbar ("Could not save image.") and remains open instead of closing silently. Previously the modal would close and the user would believe their changes were persisted.
- **Plugin & theme developers:** No API change. The fix is internal to `@wordpress/media-editor`. No hooks, filters, or public functions are added or modified.
- **No action required** for any audience; this is a behavioral correction in the bundled media editor.

## Technical details

The change is a single argument addition in `packages/media-editor/src/components/media-editor/use-save-media-editor.ts` (line ~145):

```ts
// Before
saved = ( await saveEditedEntityRecord(
  'postType',
  'attachment',
  id
) ) as Media | undefined;

// After
saved = ( await saveEditedEntityRecord(
  'postType',
  'attachment',
  id,
  { throwOnError: true }
) ) as Media | undefined;
```

`saveEditedEntityRecord` (from `@wordpress/core-data`) resolves to `undefined` on a REST error by default. Without `throwOnError`, the `await` succeeded, `saved` was `undefined`, and the hook fell through to its fallback branch that called `onSaved` with the stale record—closing the modal. With `throwOnError: true`, the promise rejects, the hook's existing `catch` block fires, dispatches the error snackbar, and the editor remains open. No new hooks, filters, or REST routes are introduced.

## Contribution

Opened by @ramonjd and co-authored with @andrewserong. The PR was split out of the larger #81805 effort. The record shows only the two co-author credits from the bot and no substantive review discussion or alternative approaches considered.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
