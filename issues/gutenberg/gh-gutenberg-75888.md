# #75888: Upload Media: Enable concurrent sideload uploads

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Status] In Progress`, `[Package] Editor`, `[Type] Feature`, `Needs PHP backport`, `[Package] Media Utils`, `[Feature] Client Side Media`
- **Merged:** [`c994c18`](https://github.com/WordPress/gutenberg/commit/c994c186704ce38fd6ed8ce5b02e42219183bf2f)
- **Discussion:** [#75888](https://github.com/WordPress/gutenberg/pull/75888) · 10 comments · 0 reactions

## Summary

The media upload store no longer serializes sideload requests for a single attachment, allowing multiple thumbnail and scaled version uploads to execute in parallel within the existing `maxConcurrentUploads` concurrency limit. This removes an unnecessary per-attachment serialization layer that previously forced sequential HTTP PUT requests, significantly reducing client-side latency during bulk imports and processing of large images with multiple missing image sizes.

## Impact

- **Plugin & theme developers / headless consumers**: No breaking changes to the block editor or REST API contracts. The `POST`/`PUT` workflow for `/wp/v2/media` remains functionally identical, but requests for a single media ID will now run concurrently rather than sequentially.
- **Hosting & platform teams**: Reduced server request queuing time during bulk sideloads. Ensure PHP-side handling of `_wp_attachment_metadata` updates is robust, as overlapping PUT requests to the same attachment ID may trigger concurrent post-meta writes.
- **Action required**: No migration or code changes needed for standard implementations. If your client-side upload logic previously depended on strict sequential ordering per attachment, adjust your concurrency assumptions accordingly.

## Technical details

- Removed per-attachment serialization logic from the `upload-media` store that enforced single-flight uploads per media ID.
- Deleted four internal helpers exclusively used by the removed serialization path: `shouldPauseForSideload`, `resumeItemByPostId`, `isUploadingToPost`, and `getPausedUploadForPost`.
- The existing `maxConcurrentUploads` configuration (defaulting to 5) now acts as the sole concurrency gate, replacing the per-ID lock.
- **Behavior shift**: Previously, uploading an image requiring multiple generated sizes triggered sequential `_wp/v2/media/<id>` PUT requests (observed as a staircase pattern in DevTools). The updated behavior sends up to 5 sideloaded thumbnail/scaled variants simultaneously, bounded only by the global concurrency cap. The test plan explicitly calls for verifying `$_wp_attachment_metadata` integrity under concurrent loads to ensure overlapping post-meta updates do not corrupt attachment state.

## Contribution

Merged by @adamsilverstein with co-authors @westonruter and @swissspidy. The PR was opened to eliminate sequential bottlenecks during gallery/bulk sideloads and reduce overall upload latency. During review, @swissspidy flagged potential race conditions when updating `_wp_attachment_metadata` simultaneously; this concern was acknowledged as a valid follow-up item but not implemented as a concurrency guard in the final merge, allowing the serialization removal to proceed.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
