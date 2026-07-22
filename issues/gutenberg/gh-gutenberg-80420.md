# #80420: [7.1 Backport] -- Media: Add timeout and size guardrails to client-side GIF to video conversion

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Feature] Client Side Media`
- **Merged:** [`39bb970`](https://github.com/WordPress/gutenberg/commit/39bb97004c6c4374f2ac31a307146fbed641abc1)
- **Discussion:** [#80420](https://github.com/WordPress/gutenberg/pull/80420) · 8 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Client-side animated GIF-to-video conversion now enforces a 30-second timeout and a total-pixel budget (width × height × frame count) to prevent oversized GIFs from blocking the upload queue. When either guardrail is triggered, conversion is gracefully skipped, the original GIF is retained, and a `SCRIPT_DEBUG` diagnostic is logged instead of surfacing a user-facing error. The change also makes worker cancellation calls fire-and-forget in `cancelItem` to prevent stuck workers from gating attachment finalization.

## Impact

- **Plugin & theme developers:** No immediate action required for standard uploads. If you programmatically trigger GIF-to-video conversion via the `TranscodeGif` operation args, you can now pass `timeout` and `maxTotalPixels` to override defaults.
- **Site owners & editors:** Large or complex GIFs that previously stalled the media library or upload queue will now upload cleanly as static GIFs without conversion errors.
- **Hosting & platform teams:** No configuration changes needed; guardrails are client-side and default to safe limits.
- **Breaking changes:** None. The `cancelItem` action now returns immediately instead of awaiting worker cancellation, but this is an internal queue behavior fix with no public API impact.

## Technical details

The diff modifies `packages/upload-media/src/store/actions.ts` to remove `await` from `vipsCancelOperations( id )` and `cancelGifToVideoOperations( id )`, changing them to fire-and-forget `.catch( () => {} )` calls:

```diff
// Before
await vipsCancelOperations( id ).catch( () => {} );
await cancelGifToVideoOperations( id ).catch( () => {} );

// After
vipsCancelOperations( id ).catch( () => {} );
cancelGifToVideoOperations( id ).catch( () => {} );
```

This prevents `cancelItem` from blocking when the VIPS worker is synchronously stuck inside a WASM call. In `packages/upload-media/src/store/private-actions.ts`, `transcodeGifItem` now passes `timeout` and `maxTotalPixels` from `args` to `convertGifToVideo`. New error guards `isConversionTimeoutError` and `isSizeLimitConversionError` intercept timeout/size-limit rejections, log a `debug()` message, and dispatch `cancelItem` with `silent = true` to keep the original attachment.

`packages/upload-media/src/store/types.ts` extends `OperationArgs[ OperationType.TranscodeGif ]` with:

```ts
timeout?: number; // Defaults to 30000ms
maxTotalPixels?: number; // Budget for width × height × frames
```

The `convertGifToVideo` utility signature now accepts an options object for these parameters, routing them to the underlying `@wordpress/video-conversion` package.

## Contribution

Opened by @adamsilverstein as a backport of trunk PR #80379 to the `wp/7.1` branch. Automated cherry-picking initially failed due to conflicts with #79955, which was resolved by merging #80421 first to align the `cancelItem` changes. @andrewserong and @t-hamano reviewed and co-authored the backport. The diff was verified to be byte-identical to trunk across the affected packages, with all 410 unit tests passing.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
