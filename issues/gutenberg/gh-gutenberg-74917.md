# #74917: Upload Media: Add error taxonomy, localized messages, and dev diagnostics

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Enhancement`, `[Status] In Progress`, `Needs Testing`, `[Feature] Client Side Media`
- **Merged:** [`a96b389`](https://github.com/WordPress/gutenberg/commit/a96b3899b3e858a18b65245e5a1373a1760b632e)
- **Discussion:** [#74917](https://github.com/WordPress/gutenberg/pull/74917) · 14 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `@wordpress/upload-media` package now exposes a structured `ErrorCode` enum and a `getErrorMessage()` helper for handling and displaying upload failures. This replaces bare string error codes with typed constants and provides i18n-ready message maps for all client-side upload errors. The change also adds `SCRIPT_DEBUG`-gated diagnostic logging and User Timings API instrumentation for media processing, plus a fix for a double-dispatch bug in `uploadItem`.

## Impact

- **Plugin & theme developers**: Can now import `ErrorCode` and `getErrorMessage` from `@wordpress/upload-media` to programmatically handle `UploadError` instances and render localized UI feedback. No breaking changes to existing public APIs, but internal error handling is now strictly typed.
- **Developers debugging uploads**: Setting `SCRIPT_DEBUG=true` now surfaces cancellation and batch-completion logs via `console.debug` and performance traces under a custom "Upload Media" track in DevTools.
- **Site owners & standard integrations**: No action required; diagnostics are production-gated and the error taxonomy is backward-compatible with existing `UploadError` consumers.

## Technical details

- `packages/upload-media/src/error-messages.ts` introduces `getErrorMessage(code: ErrorCode | string, fileName: string): ErrorMessageConfig`, mapping all nine `ErrorCode` enum values (`EMPTY_FILE`, `SIZE_ABOVE_LIMIT`, `MIME_TYPE_NOT_SUPPORTED`, `MIME_TYPE_NOT_ALLOWED_FOR_USER`, `HEIC_DECODE_ERROR`, `IMAGE_TRANSCODING_ERROR`, `IMAGE_ROTATION_ERROR`, `MEDIA_TRANSCODING_ERROR`, `GENERAL`) to localized `title`, `description`, and optional `action` strings.
- `packages/upload-media/src/index.ts` now exports `ErrorCode`, `UploadError`, `getErrorMessage`, and `ErrorMessageConfig`.
- `packages/upload-media/src/store/private-actions.ts` replaces hardcoded string codes (e.g., `'HEIC_DECODE_ERROR'`, `'IMAGE_TRANSCODING_ERROR'`) with `ErrorCode` members. It also wraps `dispatch.finishOperation` in an idempotent `finishUpload` closure guarded by a `finished` flag to prevent double-dispatch when both `onFileChange` and `onSuccess` fire.
- `packages/upload-media/src/store/utils/debug-logger.ts` adds `debug()` and `measure()` helpers. Both are gated on `globalThis.SCRIPT_DEBUG === true`. `measure()` writes to the User Timings API under a custom "Upload Media" track for `uploadItem`, `sideloadItem`, `resizeCropItem`, `rotateItem`, and `transcodeImageItem`.

## Contribution

Opened by @adamsilverstein and merged with co-authors @swissspidy and @andrewserong. The PR was originally scoped to include auto-retry logic, but that functionality landed separately in #76765. The retry scaffolding (`UploadError#isRetryable`, `RETRYABLE_CODES`, and aspirational network/server error codes) was removed to prevent overlap, leaving this PR focused on error taxonomy, localized messaging, and dev diagnostics. Review testing highlighted a site editor hang that was traced to trunk, not this PR's changes.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
