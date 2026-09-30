# #81737: Media: Detect HEIC uploads from the file header rather than the file name

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `[Feature] Client Side Media`
- **Merged:** [`df4594d`](https://github.com/WordPress/gutenberg/commit/df4594d39a41b4557304c26df85b57607d39687b)
- **Discussion:** [#81737](https://github.com/WordPress/gutenberg/pull/81737) · 7 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The block editor's client-side media pipeline now identifies HEIC uploads by reading the ISOBMFF File Type Box from the file's first 64 bytes instead of trusting `File.type`, which browsers derive solely from the file extension. A HEIC photo saved as `.jpg`/`.png` (or one that Windows reports with an empty MIME type) is now correctly routed through the HEIC conversion path rather than being sent to a vips or raw-upload path that cannot decode it, eliminating a permanent "Uploading" hang. A 15-second timeout is also added around `ImageDecoder.decode()` so a decoder that never settles (missing platform codec) cannot stall the upload indefinitely.

## Impact

- **Site owners / editors uploading HEIC photos:** No action required. Files with a wrong extension or an empty browser-reported type now convert correctly instead of hanging. On Windows without the HEVC Video Extension, the upload now reports a clean "couldn't decode HEIC" error rather than spinning forever.
- **Plugin & theme developers:** New public export `isHeicFile()` from `@wordpress/upload-media` is available for use in custom upload pipelines. No existing API is removed or deprecated; the internal `HEIC_MIME_TYPES` constant in `@wordpress/block-editor`'s provider is gone but was never a public export.
- **Hosting & platform:** No server-side changes. The fix is entirely client-side in the block editor's upload pipeline. Server-side `wp_get_image_mime()` already performs the same header check.
- **Headless & REST consumers:** No effect; this change is in the editor's client-side media processing path only.

## Technical details

**New `isHeicFile()` / `isHeicBuffer()` in `packages/upload-media/src/utils`:**

`isHeicFile(file)` trusts a HEIC MIME type (`image/heic`, `image/heif`) when present; otherwise it reads the first 64 bytes and inspects the ISOBMFF File Type Box (`ftyp`). AVIF brands are ruled out first because AVIF shares the HEIF container and also declares `mif1`. Only files that could plausibly be images are read, so an ISOBMFF video is never mistaken for a still. Both functions are exported from `@wordpress/upload-media` via `packages/upload-media/src/index.ts`.

**`prepareItem()` in `packages/upload-media/src/store/private-actions.ts`:**

```ts
// Before
const isImage = file.type.startsWith( 'image/' );
const isVipsSupported = CLIENT_SIDE_SUPPORTED_MIME_TYPES.includes( file.type );
const isHeic = HEIC_MIME_TYPES.includes( file.type );

// After
const isHeic = await isHeicFile( file );
const isMisnamedHeic = isHeic && ! HEIC_MIME_TYPES.includes( file.type );
const isImage = file.type.startsWith( 'image/' ) || isHeic;
const isVipsSupported = ! isHeic && CLIENT_SIDE_SUPPORTED_MIME_TYPES.includes( file.type );
```

When `isMisnamedHeic` is true, a new `File` is constructed with the basename plus `.heic` and type `HEIC_MIME_TYPES[0]` so the WebCodecs decoder is configured correctly. The re-labelled file is passed to `canvasConvertToJpeg()` and stored as `originalHeicFile` for the `source_original` sideload.

**`heicMediaUpload()` in `packages/block-editor/src/components/provider/index.jsx`:**

The function is now `async`. The batch partition that previously used `HEIC_MIME_TYPES.includes( file.type )` now awaits `isHeicFile( file )` per file. The local `HEIC_MIME_TYPES` constant is removed.

**`canvasConvertToJpeg()` in `packages/upload-media/src/canvas-utils.ts`:**

`ImageDecoder.decode()` is raced against a `setTimeout` of `IMAGE_DECODER_TIMEOUT` (15 000 ms, exported constant). On timeout the decoder is closed (which rejects the pending decode) and a `timedOut` flag is set. In the `catch` block, a non-timed-out rejection is re-thrown (damaged file → processing error per #81123); a timed-out rejection falls through to the next decoding strategy. The `finally` block clears the timer and closes the decoder only if it was not already closed by the timeout path.

**Tests added:**
- `packages/upload-media/src/test/is-heic-buffer.ts` — unit tests for `isHeicBuffer` and `isHeicFile` covering HEIC brands, AVIF exclusion, JPEG SOI, and short buffers.
- `packages/upload-media/src/store/test/actions.ts` — two `prepareItem` tests: a HEIC File Type Box with empty `File.type` and one with `image/jpeg` type, both asserting `ErrorCode.IMAGE_TRANSCODING_ERROR`.
- `packages/upload-media/src/test/canvas-utils.ts` — a fake-timer test for a decode that never settles (asserts `HeicUnsupportedError` after `IMAGE_DECODER_TIMEOUT`), and a test that a canvas failure after a successful decode throws a processing error rather than falling through.
- E2E spec `test/e2e/specs/editor/various/heic-wrong-extension.spec.js` covering both the full client-side mode and the HEIC-only canvas mode (Safari path).

## Contribution

Opened by @adamsilverstein as a supersedure of #81211 by @singhakanshu00. The header-check idea originated in @t-hamano's review of #81211 and was generalized here to cover both the wrong-extension and empty-type cases (the earlier PR only sniffed when `File.type` was empty). The `ImageDecoder` timeout commit (66c4e0b) is carried from #81211 under @singhakanshu00's authorship, with a follow-up (a10c5de) narrowing the fall-through to timeout-only and raising the ceiling from 3 to 15 seconds. @369work provided a Windows 11 test report confirming the empty-type case resolves with the header check alone. @aduth flagged a changelog validation issue from #83043 that required a rebase. The PR was merged as `df4594d`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
