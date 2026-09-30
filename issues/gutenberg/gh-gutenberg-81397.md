# #81397: Media: Stop claiming "Upload complete" when every upload failed

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Package] Editor`, `Backported to WP Core`, `[Feature] Client Side Media`
- **Merged:** [`91b5f89`](https://github.com/WordPress/gutenberg/commit/91b5f890c1f8e7c9dcb57bb43b20b1f6830c48cb)
- **Discussion:** [#81397](https://github.com/WordPress/gutenberg/pull/81397) · 14 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The editor's upload progress snackbar no longer says "Upload complete" when every file in a batch failed. It used to treat the upload queue draining as success, but cancelled items leave the queue the same way as uploaded ones, so a failed upload showed a completion checkmark right next to its own error. The snackbar now distinguishes three outcomes: everything uploaded ("Upload complete" with checkmark), partial ("Uploaded 3 of 5", no checkmark), and nothing uploaded (progress notice removed, no completion notice).

## Impact

**Site owners / editors**
- A failed upload (e.g. an unsupported HEIC decode or a server-rejected type such as `.exr`) now shows only the error notice, with no contradictory "Upload complete" checkmark.
- Partly failed batches report "Uploaded N of M" without a checkmark.
- Screen reader announcements follow the same three cases: "Media upload complete", the "Uploaded N of M" text, or "Media upload failed".

**Plugin & theme developers**
- No public API changes. `getFailureCount` on `@wordpress/upload-media` is a private selector (accessed via `unlock`), so it is not a supported extension point.
- Code or tests that assert on the "Upload complete" notice after a fully failed upload will see different behavior.
- The PR carries the `Backported to WP Core` label, and discussion mentions considering it for a 7.1.1 fix.

**Hosting & headless**
- No action required.

## Technical details

Both upload paths now keep a monotonically increasing failure tally, and the snackbar diffs it across a batch.

- **`@wordpress/upload-media`**: a new private selector `getFailureCount` in `src/store/private-selectors.ts` returns a running count of cancelled top-level items. Per the PR description, sub-size cancellations are excluded, since a failing sub-size cancels the parent, and the parent is what gets counted. (The reducer/action side of this is in the truncated part of the diff.)
- **Non-CSM tracker** (`packages/editor/src/components/upload-progress-snackbar/tracker.js`):
  - New module-level `cumulativeFailures`, deliberately outside `state` because `state` is cleared when a session ends.
  - `advance( count )` now delegates to an internal `finish( count, 0 )`.
  - New `advanceFailed( count = 1 )` calls `finish( count, count )`.
  - New `getFailureCount()` exports the tally. `reset()` zeroes it.
- **`packages/editor/src/utils/media-upload/index.js`**: the wrapper's `onError` now calls `trackFailure( 1 )` (`advanceFailed`) instead of `trackAdvance( 1 )`.
- **`UploadProgressSnackbar` (`index.jsx`)**: `useSelect` now also reads `unlock( select( uploadStore ) ).getFailureCount()`. On batch start, `failuresAtStartRef` records `csmFailureCount + getTrackedFailureCount()`. When the queue drains:
  - `failed = Math.min( total, failures - failuresAtStartRef.current )` and `uploaded = total - failed`, where `total` is `peakRef.current`.
  - `uploaded === 0`: `speak( 'Media upload failed' )`, `removeNotice( NOTICE_ID )`, no completion notice.
  - Otherwise: content is `Upload complete` if `failed === 0`, else `sprintf( 'Uploaded %1$d of %2$d' )`. `icon` is `UPLOAD_DONE` only on full success and `undefined` on partial.
  - `csmFailureCount` was added to the effect's dependency list.

Jest tests cover full success, all tracked failed, partial ("Uploaded 2 of 3"), all CSM failed, and failures carried over from earlier batches. Per the PR description, e2e tests in `upload-progress-snackbar.spec.js` cover the `.exr` server-rejection case on both upload paths. The changelogs for `editor` and `upload-media` are updated, and the `editor` changelog's duplicate `### Bug Fixes` sections are merged.

## Contribution

The PR closes #81132 and #81708 and was authored with Claude Code, per the author's disclosure, with review and testing by @andrewserong. During testing, Cover block drop behavior surfaced two bugs that turned out to be pre-existing on trunk: the tracker being left with stuck in-flight files when `uploadMedia()` refuses a multi-file batch, and the client-side `mediaUpload` replacement ignoring `multiple`. These were split into issue #82041 and the stacked PR #82046. Andrew suggested merging #82046 first, but the author kept the order as-is since #82046 is stacked on this branch. The author put the PR on the 7.1 board to consider it for 7.1.1.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
