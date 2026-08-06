# #81130: Media: Reword the HEIC upload error and keep it up until dismissed

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Package] Block library`, `Backported to WP Core`, `[Feature] Client Side Media`
- **Merged:** [`415b6f4`](https://github.com/WordPress/gutenberg/commit/415b6f45080cb941a179dc701d581bff192ed846)
- **Discussion:** [#81130](https://github.com/WordPress/gutenberg/pull/81130) · 16 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The HEIC upload error message in the block editor is now dynamically generated based on the user’s browser and operating system, replacing a static, generic warning. The snackbar also persists until manually dismissed instead of auto-expiring after six seconds. This ensures users receive actionable, platform-specific guidance when client-side HEIC conversion fails.

## Impact

- **Plugin & theme developers:** No breaking changes. The `@wordpress/upload-media` package now exports `getHeicUnsupportedMessage` and `getHeicConversionAdvice` for external consumption.
- **Site owners & editors:** HEIC upload failures now display tailored advice (e.g., suggesting Safari on macOS or acknowledging Linux codec limitations) and remain visible until dismissed, reducing confusion about why conversion failed.
- **No action required** for existing code; the change is internal to the block editor’s media handling.

## Technical details

The diff introduces `packages/upload-media/src/heic-support.ts`, which exports `getHeicUnsupportedMessage()` and `getHeicConversionAdvice()`. Detection relies on `navigator.userAgentData.brands` with a fallback to `navigator.userAgent`, bucketing environments into `BrowserFamily` (`'firefox'`, `'safari'`, `'chromium'`) and `Platform` (`'macos'`, `'windows'`, `'linux'`, `'other'`). These functions replace hardcoded strings in `canvasConvertToJpeg()` (throw), `getErrorMessage()` (HEIC entry), and the `HEIC_DECODE_ERROR` action in `private-actions.ts`. In `packages/block-library/src/image/edit.js`, `createErrorNotice()` now receives `{ type: 'snackbar', explicitDismiss: true }`, overriding the default six-second timeout for all image upload errors.

## Contribution

Opened by @adamsilverstein and merged with co-authors, the PR initially attempted to render the error inside the Image block placeholder to ensure persistence. @andrewserong flagged layout constraints in narrow columns and constrained Groups, prompting a revert of the placeholder logic. The team agreed to ship the snackbar wording update with `explicitDismiss: true` for the 7.1 cycle, deferring a top-of-editor notice implementation to a follow-up.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
