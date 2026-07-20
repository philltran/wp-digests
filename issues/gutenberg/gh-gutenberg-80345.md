# #80345: Fix upload snackbar stuck in uploading state on server-side uploads

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Package] Editor`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Feature] Client Side Media`
- **Merged:** [`84c71f8`](https://github.com/WordPress/gutenberg/commit/84c71f80f34d4835423f4b4e78b6b2de374a16ea)
- **Discussion:** [#80345](https://github.com/WordPress/gutenberg/pull/80345) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a bug where the editor’s upload progress snackbar remains stuck in the "Uploading" state indefinitely when media is uploaded via the traditional server-side path. This occurs when client-side media processing is disabled via the `wp_client_side_media_processing_enabled` filter or unsupported in the browser. The fix resolves a render-phase state update that corrupted `useSyncExternalStore` bookkeeping, ensuring the snackbar correctly transitions to "Upload complete" and dismisses.

## Impact

- **Plugin & theme developers / Editor users:** No code changes required. The fix applies automatically when client-side media processing is disabled or unavailable.
- **Hosting & platform teams:** No configuration changes needed. Sites relying on the traditional upload path will see corrected snackbar behavior.
- **Breaking changes:** None. This is a pure bug fix with no API changes or deprecations.

## Technical details

The bug originated in `packages/editor/src/components/upload-progress-snackbar/index.js`, where `UploadProgressSnackbar` used `useState` to track the peak upload count (`peak`). The component called `setPeak()` synchronously during render to update the high-water mark, triggering a render-phase state update. This discarded the current render pass and corrupted the internal snapshot state of the `useSyncExternalStore` subscription to the `upload-media` store, causing the tracker’s final completion notification to be silently dropped.

The fix replaces the `peak` state with a `useRef` (`peakRef`) and moves the high-water mark update into the `useEffect` that manages the notice lifecycle. This eliminates the render-phase state mutation.

Before:
```js
const [ peak, setPeak ] = useState( 0 );
if ( sessionTotal > peak ) {
    setPeak( sessionTotal );
}
```

After:
```js
const peakRef = useRef( 0 );
// ... inside effect ...
if ( sessionTotal > peakRef.current ) {
    peakRef.current = sessionTotal;
}
```

The effect dependency array was also updated to include `sessionTotal` instead of `peak`. An e2e test was added to `test/e2e/specs/editor/various/upload-progress-snackbar.spec.js` to verify the server-side upload path.

## Contribution

Opened and merged by @adamsilverstein, with co-authors @andrewserong and @noruzzamans. The PR was reviewed, tested in Playground, and merged directly into the Gutenberg RC branch before being cherry-picked to `wp/7.1` for inclusion in the upcoming WordPress release. The discussion focused on verifying the fix against the server-side upload path and confirming the e2e test reliably reproduces the regression.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
