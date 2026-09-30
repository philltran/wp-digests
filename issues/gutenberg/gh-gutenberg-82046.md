# #82046: Media: Refuse a multi-file drop on a placeholder that takes one file

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Feature] Media`, `[Package] Editor`, `[Package] Block editor`, `Backported to WP Core`, `[Feature] Client Side Media`
- **Merged:** [`cfbf61c`](https://github.com/WordPress/gutenberg/commit/cfbf61c0a12c6a065be4c998d8c58899ec398f6e)
- **Discussion:** [#82046](https://github.com/WordPress/gutenberg/pull/82046) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Dropping more than one file on a media placeholder that accepts a single file (e.g. the Cover block's) is now refused up front with "Only one file can be used here." on both the server-side and client-side upload paths. Previously the server-side path left the upload progress snackbar stuck and swallowed all later uploads into it, while the client-side path ignored `multiple` entirely, uploaded every file, and left the block holding whichever finished last.

## Impact

- **Site owners / editors:** Multi-file drops onto single-file placeholders now show an error instead of a stuck "Uploading…" notice (server-side) or orphaned media-library uploads (client-side). No more reloading the editor to clear the stranded snackbar.
- **Plugin & theme developers:** Custom blocks using `MediaPlaceholder` with `multiple` unset (default `false`) now see `onError` called with the refusal message on a multi-file drop under client-side media processing, matching the server-side path. Blocks that pass `multiple: true` are unaffected. Code that depended on multi-file drops silently uploading everything under client-side processing will change behavior.
- **Hosting & platform:** No action required.
- **Backporting:** Labeled `Backported to WP Core`. The discussion flags it as a 7.1.1 backport candidate.

## Technical details

Two separate `mediaUpload` implementations get the same `multiple` guard, placed before any side effects.

**Server-side path, `packages/editor/src/utils/media-upload/index.js`:** at the top of the wrapper, `if ( ! multiple && filesList.length > 1 )` calls `onError( __( 'Only one file can be used here.' ) )` and returns. This runs before the batch is registered with the upload-progress tracker (`addFiles`/`trackStart`). Previously `uploadMedia()` from `@wordpress/media-utils` refused the batch with a single `onError`, but the tracker had already counted every file and only advanced by one per error. The tracker appends to the in-progress session, so later uploads joined the stuck one.

**Client-side path, `packages/block-editor/src/components/provider/index.js`:** both `mediaUpload` and `heicMediaUpload` (the HEIC-only interceptor) now destructure `multiple = true` and call a new local helper `refuseExtraFiles( filesList, multiple, onError )`, which returns `true` after calling `onError` with the same message. `heicMediaUpload` also forwards `multiple` when delegating to the fallback `mediaUpload`. Previously `multiple` was never destructured, so every file went to `addItems` on the `uploadStore` and each arrived via `onChange` separately. The helper is local because `@wordpress/media-utils` is not a `block-editor` dependency.

```js
// before (client-side path): multiple ignored
void registry.dispatch( uploadStore ).addItems( { files: Array.from( filesList ), /* ... */ } );

// after
if ( refuseExtraFiles( filesList, multiple, onError ) ) {
	return;
}
```

Note the default is `multiple = true` in the block-editor interceptors, so callers that don't pass `multiple` are unaffected there.

**Tests:** new Jest spec `packages/editor/src/utils/media-upload/test/index.js` checks that a refused batch leaves the tracker state `null` and a later `addFiles` starts its own session. New e2e spec `single-file-placeholder-drop.spec.js` covers both paths, using the `gutenberg-test-plugin-disable-client-side-media-processing` plugin for the server-side case. The CSM skip gate was extracted into a shared `client-side-media-utils.js`. CHANGELOG entries were added to both packages. No new hooks, REST, or schema changes.

## Contribution

The two bugs were spotted by @andrewserong while testing #81397, which neither caused nor fixed them. The fix was initially stacked on that PR and later rebased onto `trunk` to land independently. The author, @adamsilverstein (with Claude Code disclosed as writer of code and description), weighed an alternative of taking the first file and silently ignoring the rest, but rejected it: it would change server-side behavior and drop deliberately selected files without telling the user. The author offered to switch if reviewers preferred. The new e2e test was flagged flaky once (the client-side case saw two leftover media items before passing on retry). In discussion, @t-hamano clarified that the "Backport to WP Minor Release" label alone suffices for backport tracking.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
