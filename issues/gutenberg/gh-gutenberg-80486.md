# #80486: [7.1/Backport] Media: Remove the redundant __heicUploadSupport flag (#80452)

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Package] Block editor`, `[Package] Media Utils`
- **Merged:** [`b709589`](https://github.com/WordPress/gutenberg/commit/b709589acf2eb8d8c7c9a0e826e0c65816822076)
- **Discussion:** [#80486](https://github.com/WordPress/gutenberg/pull/80486) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This change removes the redundant `window.__heicUploadSupport` global flag and consolidates the gate for client-side media processing under `window.__clientSideMediaProcessing`. The PHP function that injects the flag was renamed and now re-evaluates the `gutenberg_is_client_side_media_processing_enabled()` filter at runtime. This ensures the HEIC canvas conversion fallback correctly respects the media processing filter, fixing broken HEIC uploads in browsers like Safari that lack `SharedArrayBuffer` support.

## Impact

- **Plugin & theme developers:** No action required. If you were reading `window.__heicUploadSupport`, switch to `window.__clientSideMediaProcessing`.
- **Block & media developers:** The `gutenberg_set_heic_upload_support_flag()` PHP function is removed. The `Window` interface in `media-utils` no longer declares `__heicUploadSupport`.
- **Site owners:** No action required. HEIC uploads in Safari and other non-Chromium browsers will now correctly fall back to canvas conversion when cross-origin isolation is absent.
- **Breaking/Deprecation:** `window.__heicUploadSupport` and `gutenberg_set_heic_upload_support_flag()` are removed. No migration path is needed unless you were directly referencing these internal symbols.

## Technical details

In `lib/media/load.php`, the duplicate `gutenberg_set_client_side_media_processing_flag()` definition is removed and the original `gutenberg_set_heic_upload_support_flag()` is renamed. It now runs on `admin_init`, explicitly checks `gutenberg_is_client_side_media_processing_enabled()`, and injects `window.__clientSideMediaProcessing = true` via `wp_add_inline_script()`. On the frontend, `packages/block-editor/src/components/provider/index.js` updates `shouldEnableHeicCanvasProcessing()` to check the new flag, and `packages/block-library/src/image/edit.js` simplifies the `isSideloading` selector:

```diff
- if (
-   ( ! window.__clientSideMediaProcessing &&
-     ! window.__heicUploadSupport ) ||
-   ! id
- ) {
+ if ( ! window.__clientSideMediaProcessing || ! id ) {
```

The TypeScript declaration in `packages/media-utils/src/utils/upload-media.ts` drops `__heicUploadSupport` from the `Window` interface. E2E tests replace the old `gutenberg-test-plugin-disable-client-side-media-processing` with a new `disable-cross-origin-isolation.php` plugin that sets `gutenberg_use_document_isolation_policy` to `false`, adding a `waitForRequest` assertion for the `sideload` endpoint to prove the canvas fallback path executed.

## Contribution

The change was opened and merged as a backport of #80452 to the `wp/7.1` branch. The original trunk PR failed an automated cherry-pick due to a changelog conflict with an unrelated entry (#77894). The conflict was resolved by isolating the `#80452` changelog entry under `## Unreleased`, and the code changes were re-cherry-picked after being verified as byte-identical to trunk outside of changelog files. The PR includes updated e2e tests and a new test plugin to validate the Safari canvas fallback path.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
