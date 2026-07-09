# #76565: RTC: Implement front-end peer limits

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @pkevan
- **Labels:** `[Type] Enhancement`, `Needs Dev Note`, `[Feature] Real-time Collaboration`, `[Type] Iteration`, `Backported to WP Core`, `[Package] Sync`
- **Merged:** [`7efbb86`](https://github.com/WordPress/gutenberg/commit/7efbb86580279901edc2fa6ac9ecc3b52f88b8a0)
- **Discussion:** [#76565](https://github.com/WordPress/gutenberg/pull/76565) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Real-time collaboration (RTC) in the block editor now enforces a client-side limit on concurrent editors per document, defaulting to three peers. This prevents server resource exhaustion without requiring PHP backport changes by inspecting awareness state returned from the sync provider and disconnecting excess clients. A `connection-limit-exceeded` error is emitted when the threshold is breached, triggering a modal that blocks further edits until a peer disconnects or the user retries.

## Impact

- **Multi-editor & agency teams:** Automatically caps active editing sessions to three per document by default. Excess browser tabs receive a "Too many editors connected" modal and are disconnected.
- **Plugin & theme developers:** Can adjust the concurrent peer threshold using the `sync.pollingProvider.maxPeersPerRoom` filter via `wp.hooks.applyFilters()`.
- **Hosting & platform teams:** No server-side configuration, PHP patches, or architecture changes required; limits are enforced entirely on the client.
- **Action required:** Developers supporting collaborative workspaces with three or more active editors may need to coordinate session management or apply the new filter to raise the cap.

## Technical details

The `@wordpress/sync` package's front-end polling provider now counts unique WordPress user IDs from the server-returned awareness payload after each poll cycle. If the count exceeds the configured maximum, the client emits a `connection-limit-exceeded` event and gracefully disconnects the excess peer. The default cap is hardcoded to 3 but can be overridden externally via `wp.hooks.applyFilters('sync.pollingProvider.maxPeersPerRoom', ...)`. The change targets the client-side sync flow entirely, avoiding PHP modifications while maintaining server protection. Compiled output lands in `build/scripts/sync/index.min.js` (+178 B).

## Contribution

Opened by @pkevan to replace a previous PHP-dependent implementation (PR #75381) with a purely front-end enforcement mechanism. Merged after review cycles addressed awareness payload parsing and retry modal behavior, with known edge cases (refresh race conditions, persistent error overlays) deferred to follow-up PRs like #76554. Co-authored by @chriszarate and @ingeniumed.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
