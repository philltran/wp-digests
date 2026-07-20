# #80436: [7.1 Backport] -- Fix upload snackbar double-counting a single HEIC upload in Safari

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Package] Editor`, `Backport to Gutenberg RC`, `[Feature] Client Side Media`
- **Merged:** [`5d0e977`](https://github.com/WordPress/gutenberg/commit/5d0e977ff19c81736b80c4ecadb61e6b89d793b0)
- **Discussion:** [#80436](https://github.com/WordPress/gutenberg/pull/80436) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a bug in the Gutenberg editor where uploading a single HEIC image in Safari incorrectly displayed "Uploading 2 media items" in the progress snackbar. The issue occurred because the `@wordpress/upload-media` queue delegated the server transport to the `mediaUpload` setting, causing the editor's progress tracker to count the same file twice. The fix introduces an `isTransportOnly` flag to `mediaUpload` that tells lifecycle-managing consumers to skip their own progress tracking and save locking when the queue already handles it.

## Impact

- **Plugin & theme developers**: No immediate action required for standard usage. If you call `mediaUpload` directly to manage custom upload flows, you can now pass `isTransportOnly: true` to opt out of the editor's built-in progress tracking and save locking.
- **Site owners & editors**: No action required. The snackbar will now correctly report single-file upload progress in Safari.
- **Platform & hosting teams**: No action required.

## Technical details

- Adds `isTransportOnly?: boolean` to the `UploadMediaArgs` interface in `packages/upload-media/src/store/types.ts`.
- Updates `packages/editor/src/utils/media-upload/index.js` to accept `isTransportOnly` (default `false`). When `true`, the function skips `lockPostSaving`/`lockPostAutosaving`, bypasses the upload progress tracker, and omits `trackAdvance` calls. The previous `isClientSideMediaActive` checks were consolidated into this flag to simplify the transport path.
- Modifies `packages/upload-media/src/store/private-actions.ts` to pass `isTransportOnly: true` when invoking the `mediaUpload` setting from the `uploadItem` action.
- Before/after pattern for the queue's transport call:
  ```js
  // Before (implicit double-count)
  settings.mediaUpload( { filesList: [ item.file ], ... } );
  
  // After
  settings.mediaUpload( { filesList: [ item.file ], isTransportOnly: true, ... } );
  ```
- Adds unit tests for `media-upload` and `private-actions`, plus an e2e test (`heic-upload-progress-snackbar.spec.js`) that verifies the snackbar shows "Upload complete" instead of a multi-file count for a single HEIC file.

## Contribution

This is a manual backport of trunk PR #80371 to the `wp/7.1` branch after an automated cherry-pick failed due to a `CHANGELOG.md` conflict. @adamsilverstein authored the backport, with @swissspidy co-authoring. @t-hamano reviewed the conflict resolution. The underlying fix was originally developed and merged to trunk, addressing the Safari HEIC double-count regression.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
