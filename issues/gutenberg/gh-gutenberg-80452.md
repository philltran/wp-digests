# #80452: Media: Remove the redundant __heicUploadSupport flag

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Package] Block editor`, `[Package] Media Utils`, `Backported to WP Core`, `No Core Sync Required`, `[Feature] Client Side Media`
- **Merged:** [`ae9a734`](https://github.com/WordPress/gutenberg/commit/ae9a73420ee8a476b32885d6938082a24e754e65)
- **Discussion:** [#80452](https://github.com/WordPress/gutenberg/pull/80452) · 10 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Removes the redundant `window.__heicUploadSupport` flag and consolidates client-side media processing gating under `window.__clientSideMediaProcessing`. This fixes a regression where Safari on core WordPress 7.1 installs failed to trigger client-side HEIC-to-JPEG conversion, and ensures the `wp_client_side_media_processing_enabled` filter consistently disables both the full VIPS pipeline and the canvas fallback.

## Impact

- **Plugin & theme developers:** The `window.__heicUploadSupport` flag is removed; use `window.__clientSideMediaProcessing` instead. If you hook into `wp_client_side_media_processing_enabled`, disabling it now consistently blocks both the WASM/VIPS pipeline and the Safari HEIC canvas fallback. Previously, late-loading plugins could disable the pipeline but leave the canvas flag active.
- **Safari users on core WP 7.1+:** Client-side HEIC conversion will now work as intended without requiring a separate core PHP patch.
- **No action required** for most sites; this is a transparent internal consolidation and bug fix.

## Technical details

- `lib/media/load.php`: `gutenberg_set_heic_upload_support_flag()` is removed and replaced by `gutenberg_set_client_side_media_processing_flag()`, which now runs in the tier 1 section and includes a runtime re-check of `gutenberg_is_client_side_media_processing_enabled()` before echoing `window.__clientSideMediaProcessing = true`.
- `packages/block-editor/src/components/provider/index.js`: `shouldEnableHeicCanvasProcessing()` now reads `window.__clientSideMediaProcessing` instead of `window.__heicUploadSupport`.
- `packages/block-library/src/image/edit.js`: The `isSideloading` check in `ImageEdit` is simplified from a dual-flag check to a single flag check:
  ```js
  // Before
  if (
    ( ! window.__clientSideMediaProcessing &&
      ! window.__heicUploadSupport ) ||
    ! id
  ) {
    return false;
  }

  // After
  if ( ! window.__clientSideMediaProcessing || ! id ) {
    return false;
  }
  ```
- `packages/media-utils/src/utils/upload-media.ts`: The `__heicUploadSupport` property is removed from the `Window` interface type declaration.
- E2E test `heic-upload-progress-snackbar.spec.js` now activates `disable-cross-origin-isolation` to simulate Safari’s lack of `SharedArrayBuffer`, ensuring the canvas fallback path is genuinely tested via a sideload request assertion.

## Contribution

During review, a near-identical PR (#80453) was identified and closed in favor of this PR, which retained the necessary runtime filter re-check and E2E sideload assertion. The commit was backported to the `wp/7.1` branch after a manual conflict resolution, with the author coordinating the cherry-pick directly.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
