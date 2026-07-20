# #80371: Fix upload snackbar double-counting a single HEIC upload in Safari

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Package] Editor`, `Backported to WP Core`, `[Feature] Client Side Media`
- **Merged:** [`88074b1`](https://github.com/WordPress/gutenberg/commit/88074b1575999a372f28f6c83c9d9d0144720751)
- **Discussion:** [#80371](https://github.com/WordPress/gutenberg/pull/80371) · 10 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a bug in the block editor where uploading a single HEIC image in Safari incorrectly displayed "Uploading 1 of 2" in the progress snackbar. The issue occurred because the `@wordpress/upload-media` queue and the editor's `mediaUpload` wrapper both tracked the same converted JPEG file. The fix introduces an `isTransportOnly` flag that tells the wrapper to skip its own progress tracking and save locking when it is being used solely as a server transport by the upload queue.

## Impact

- **Plugin & theme developers:** No immediate action required. The `mediaUpload` wrapper now accepts an optional `isTransportOnly` parameter (defaults to `false`), preserving backward compatibility.
- **Site owners & editors:** No action required. Fixes a confusing UI state during HEIC uploads in Safari.
- **Platform & hosting teams:** No action required. This is a client-side editor fix with no server or database implications.

## Technical details

- **Files changed:** `packages/editor/src/utils/media-upload/index.js`, `packages/upload-media/src/store/private-actions.ts`, `packages/editor/README.md`, `packages/upload-media/CHANGELOG.md`.
- **Core change:** The `mediaUpload` function in `@wordpress/editor` previously used `window.__clientSideMediaProcessing && isClientSideMediaSupported()` to decide whether to register files with the progress snackbar and lock post saving. This check failed in Safari (where full CSM is unsupported but HEIC canvas conversion is active), causing double registration.
- The diff replaces the environment check with a new `isTransportOnly` boolean parameter. When `true`, the wrapper skips `lockPostSaving`, `lockPostAutosaving`, `trackAdvance`, and snackbar registration.
- `@wordpress/upload-media`'s `uploadItem` action now passes `isTransportOnly: true` when invoking the `mediaUpload` setting, ensuring the queue owns the lifecycle UX.
- **Before/After:**
  ```javascript
  // Before (editor/src/utils/media-upload/index.js)
  const isClientSideMediaActive = window.__clientSideMediaProcessing && isClientSideMediaSupported();
  if ( ! isClientSideMediaActive ) {
      lockPostSaving( lockKey );
      // ... snackbar tracking ...
  }

  // After
  export default function mediaUpload( { isTransportOnly = false, ... } ) {
      if ( ! isTransportOnly ) {
          lockPostSaving( lockKey );
          // ... snackbar tracking ...
      }
  }
  ```
- The change also fixes a latent gap: direct calls to `mediaUpload` while full CSM is active now correctly track and lock saving, regardless of the previous environment check.

## Contribution

Opened and merged by @adamsilverstein, with co-authors @andrewserong and @swissspidy. The PR was merged to trunk and subsequently backported to the `wp/7.1` branch via #80436 to resolve a cherry-pick conflict. The design shifted from relying on a global environment check (`isClientSideMediaActive`) to passing explicit caller intent (`isTransportOnly`), which cleanly separates the queue's transport usage from direct block-editor calls.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
