# #80379: Media: Add timeout and size guardrails to client-side GIF to video conversion

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Enhancement`, `[Feature] Client Side Media`, `Backport to WP 7.1 Beta/RC`
- **Merged:** [`8852382`](https://github.com/WordPress/gutenberg/commit/8852382c3609374c0c7542593a8796e3f54452e3)
- **Discussion:** [#80379](https://github.com/WordPress/gutenberg/pull/80379) · 11 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Client-side animated GIF to video conversion now includes a total-pixel budget check and a 30-second timeout to prevent CPU exhaustion and upload hangs on pathological GIFs. When a GIF exceeds the default 300 megapixel budget or the conversion exceeds the timeout, the process is gracefully skipped or abandoned, keeping only the original GIF upload without surfacing an error to the user. Both thresholds are configurable via the `TranscodeGif` operation arguments.

## Impact

- **Plugin & theme developers / block authors:** No immediate action required for standard usage. If you customize client-side media uploads via the `TranscodeGif` operation, you can now pass `timeout` and `maxTotalPixels` to adjust guardrails.
- **Site owners / editors:** Large animated GIFs that previously caused the editor to hang or crash during upload will now skip video conversion gracefully, keeping the upload process responsive.
- **Hosting & platform:** Reduces client-side CPU/memory spikes during media uploads, improving stability on lower-end devices.
- **No breaking changes.** The `cancelItem` action now fires worker cancellation calls without awaiting them, which prevents queue deadlocks but does not change the public API contract.

## Technical details

- `packages/upload-media/src/store/actions.ts`: `cancelItem` now calls `vipsCancelOperations( id )` and `cancelGifToVideoOperations( id )` without `await`. This prevents the upload queue from blocking when a WASM worker is synchronously stuck.
- `packages/upload-media/src/store/private-actions.ts`: `transcodeGifItem` passes `timeout` and `maxTotalPixels` to `convertGifToVideo`. It now checks for `isSizeLimitConversionError` and `isConversionTimeoutError`, logging a `SCRIPT_DEBUG` diagnostic via `debug()` and silently cancelling the sideload item.
- `packages/upload-media/src/store/types.ts`: Extends `OperationArgs[ OperationType.TranscodeGif ]` with optional `timeout` (ms, default 30000) and `maxTotalPixels` (default 300000000) properties.
- `packages/upload-media/src/store/utils/video-conversion.ts`: Introduces `isSizeLimitConversionError` and `isConversionTimeoutError` helpers that match error message prefixes. `convertGifToVideo` now accepts an options object for these guardrails and uses `Promise.race` to enforce the timeout, cancelling the worker-side operation on expiry.
- Before/after usage pattern for `transcodeGifItem`:
  ```javascript
  // Before
  const file = await convertGifToVideo( item.id, gifFile, outputMimeType );

  // After
  const file = await convertGifToVideo( item.id, gifFile, outputMimeType, {
    timeout: args?.timeout,
    maxTotalPixels: args?.maxTotalPixels,
  } );
  ```

## Contribution

Opened by `@adamsilverstein` and merged in commit `8852382`. The PR addresses hanging uploads reported during testing of client-side GIF conversion, where large screen-recording GIFs would block the editor for minutes. Review discussion confirmed that the pixel-budget check and timeout protect the conversion step specifically, while a separate PR (#80268) handles sub-size generation. The team also identified and fixed a queue deadlock where `cancelItem` awaited unresponsive WASM worker cancellations, changing it to a fire-and-forget pattern. The change was backported to the `wp/7.1` branch.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
