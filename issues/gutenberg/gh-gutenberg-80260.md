# #80260: Fix upload hang when converting long animated GIFs: decode only the first frame for still outputs

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Feature] Client Side Media`
- **Merged:** [`ee1bdb8`](https://github.com/WordPress/gutenberg/commit/ee1bdb84ebdf1e2a59bf9d63ebe07cf32e96e4ca)
- **Discussion:** [#80260](https://github.com/WordPress/gutenberg/pull/80260) · 20 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Fixes a permanent upload hang in the block editor when processing long animated GIFs with client-side media processing enabled. The hang occurred because generating a static poster frame decoded every animation frame into a single vertical strip, exceeding libjpeg’s 65,500 px dimension limit and causing a silent worker thread failure. The fix restricts multi-frame decoding to output formats that support animation and normalizes non-standard error objects in the worker RPC layer so failures properly reject promises instead of resolving as `undefined`.

## Impact

- **Plugin & theme developers / block authors**: No direct API changes. Client-side media processing workflows involving GIF-to-video conversion will now complete reliably, and the generated `animated_video_poster` will correctly contain only the first frame instead of a stacked film strip.
- **Hosting & platform teams**: No configuration changes required. The fix ships in Gutenberg and is backported to WordPress Core.
- **No action required** for existing codebases; the change is transparent to callers of `convertImageFormat` and `expose`.

## Technical details

- In `@wordpress/vips`, `convertImageFormat` now gates the `[n=-1]` load option on both input and output animation support. The condition changed from `supportsAnimation( inputType )` to `supportsAnimation( inputType ) && supportsAnimation( outputType )`, so still outputs (e.g., GIF → JPEG) decode only the first frame.
- In `@wordpress/worker-threads`, `expose()` now wraps the target object with an internal `withNormalizedErrors()` helper. This wrapper catches thrown values (including `WebAssembly.Exception` or empty `Error` instances) and converts them to standard `Error` objects with non-empty messages, ensuring the main thread receives a proper promise rejection instead of a successful `undefined` result.
- The diff modifies `packages/vips/src/index.ts` and `packages/worker-threads/src/worker-thread.ts`, adding unit tests for the frame-loading matrix and error propagation, plus an e2e test that verifies a 700-frame GIF upload completes and produces a correctly dimensioned poster.

## Contribution

Opened and merged by @adamsilverstein with co-authorship from @andrewserong. The pair identified the root cause through manual testing with large GIFs and traced the hang to a libjpeg dimension limit combined with swallowed worker thread exceptions. The discussion explored additional guardrails (size caps, timeouts, progress tracking) but the merged diff implements only the frame-loading gate and error normalization. The change was backported to WordPress Core.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
