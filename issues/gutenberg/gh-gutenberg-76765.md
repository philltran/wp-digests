# #76765: Upload Media: Add retry with exponential backoff and network resilience

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Enhancement`, `[Status] In Progress`, `Needs Testing`, `[Package] Editor`, `[Feature] Client Side Media`
- **Merged:** [`9c0f9b0`](https://github.com/WordPress/gutenberg/commit/9c0f9b08a0d46947c21722f77baffdf743011ba3)
- **Discussion:** [#76765](https://github.com/WordPress/gutenberg/pull/76765) · 29 comments · 0 reactions

## Summary

This enhancement adds automatic retry logic and network resilience to the client-side media upload pipeline. Failed uploads are now retried up to three times using exponential backoff with jitter, and the upload queue automatically pauses during offline periods and resumes when connectivity returns. These changes improve upload reliability for large files or unstable networks without altering existing editor UI flows.

## Impact

- **Plugin & theme developers**: Upload behavior is now resilient by default; no code changes are required unless you need to reproduce legacy single-attempt failures.
- **Embedders & custom clients**: Can opt out of automatic retries by passing `settings={{ retry: undefined }}` to `MediaUploadProvider` or dispatching `updateSettings({ retry: undefined })`.
- **No action required** for standard WordPress sites and users; the feature is wired into the default store state and activates automatically where the client-side media pipeline is used.

## Technical details

- Initializes retry configuration in the `core/upload-media` package via `DEFAULT_STATE.settings.retry`: `{ maxRetryAttempts: 3, initialRetryDelayMs: 1000, maxRetryDelayMs: 30000, backoffMultiplier: 2, retryJitter: 0.1 }`.
- Classifies transient errors by matching against known client-side failure strings: `Failed to fetch` (Chrome), `Load failed` (Safari), Node DNS/TCP codes (`ECONNRESET`, `ETIMEDOUT`, `ENOTFOUND`), and `@wordpress/api-fetch`'s `fetch_error`.
- Implements a `useNetworkReconnect` hook that registers `offline`/`online` event listeners, dispatching `pauseQueue` and `resumeQueue` actions to manage the upload queue lifecycle.
- Opt-out relies on shallow merging within `updateSettings()`, allowing embedders to pass `retry: undefined` without wiping other store settings. State transitions explicitly track `PENDING_RETRY` timestamps before falling back to permanent failure notices.

## Contribution

Opened by @adamsilverstein to improve client-side media upload reliability (addressing #76790) and merged at commit `9c0f9b0`. The PR was finalized with co-authored reviews/edits from @andrewserong, @jasmussen, @swissspidy, @karmatosed, and @ramonjd. The delivery focuses on the `@wordpress/upload-media` package and editor hooks, introducing ~746 B of additional build weight alongside comprehensive unit tests for backoff calculations, error classification patterns, and queue lifecycle management.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
