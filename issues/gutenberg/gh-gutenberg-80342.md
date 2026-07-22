# #80342: Backport to 7.1: Fix upload hang when converting long animated GIFs (#80260)

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Feature] Client Side Media`
- **Merged:** [`413bc0e`](https://github.com/WordPress/gutenberg/commit/413bc0eab6de27341fa1b668f8320d592c328e2a)
- **Discussion:** [#80342](https://github.com/WordPress/gutenberg/pull/80342) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Fixes a client-side media processing bug where uploading long animated GIFs would hang indefinitely. The hang occurred because generating a static JPEG poster for the converted video decoded every animation frame into a single vertical strip, exceeding libjpeg’s 65,500 px dimension limit and crashing the image worker. The fix ensures `convertImageFormat` only loads all frames when the output format supports animation, and hardens the worker RPC layer to properly propagate non-`Error` exceptions as promise rejections.

## Impact

- **Plugin & theme developers / site owners using client-side media processing:** No code changes required. The fix resolves a silent upload hang for long animated GIFs.
- **Headless & REST consumers:** No direct impact; the change is isolated to the Gutenberg editor’s client-side worker pipeline.
- **No action required.** The backport applies cleanly to the `wp/7.1` branch and ships as a transparent bug fix.

## Technical details

- `packages/vips/src/index.ts`: `convertImageFormat` previously checked `supportsAnimation(inputType)` to decide whether to pass `[n=-1]` to libvips. It now also checks `supportsAnimation(outputType)`. Converting a GIF to JPEG now passes an empty load string and `{}` options, decoding only the first frame instead of stacking all frames vertically.
- `packages/worker-threads/src/worker-thread.ts`: `expose()` now wraps the target object with `withNormalizedErrors()`. This interceptor catches thrown values (including `WebAssembly.Exception` or `Error` instances with empty messages) and converts them into standard `Error` objects with non-empty messages before they reach the RPC reply layer. This prevents worker crashes from being misinterpreted as successful `undefined` results, which previously left the upload queue in a permanent loading state.
- New unit tests in `packages/vips/src/test/convert-image-format.ts` and `packages/worker-threads/src/test/error-propagation.test.ts` verify the frame-loading logic and error normalization. An e2e test in `test/e2e/specs/editor/various/gif-to-video.spec.js` validates the full upload pipeline with a 700-frame GIF fixture.

## Contribution

Originally authored by @adamsilverstein and @andrewserong, this PR backports the fix from #80260 to the wp/7.1 branch. The cherry-pick encountered a minor CHANGELOG.md conflict that was resolved by dropping an unrelated changelog entry. Review focused on verifying the worker error propagation edge case, with @noruzzamans and @t-hamano confirming the fix with synthetic long-GIF fixtures. The approach was straightforward: constrain the libvips load options to the output format’s capabilities and add a defensive wrapper around the worker RPC layer.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
