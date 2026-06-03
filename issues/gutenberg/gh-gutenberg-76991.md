# #76991: RTC: Provide `PROTOCOL_MISMATCH` error handling

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @chriszarate
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Feature] Real-time Collaboration`, `[Package] Sync`
- **Merged:** [`79a697a`](https://github.com/WordPress/gutenberg/commit/79a697a84fe4f59e944151c8093b7d94cd7d4f4f)
- **Discussion:** [#76991](https://github.com/WordPress/gutenberg/pull/76991) · 5 comments · 0 reactions

## Summary

This PR adds proactive handling for a `PROTOCOL_MISMATCH` error code in the Real-time Collaboration (RTC) sync subsystem. When the `/wp-sync/v1/updates` REST endpoint responds with the `rest_sync_protocol_mismatch` error code (HTTP 426), the editor now immediately surfaces the `SyncConnectionErrorModal` rather than waiting through the normal 20-second countdown. The `PROTOCOL_MISMATCH` constant is exported from the `sync-error-messages` utility and consumed by the modal component. Nothing in core currently emits this error — the change is scaffolding designed to support future RTC protocol upgrade paths without requiring a follow-up UI change at that time.

## Impact

**Plugin & theme developers / platform teams running RTC infrastructure**
- No immediate behavioral change: `rest_sync_protocol_mismatch` is not emitted by any current server code. This handling only activates when a future server-side update introduces it.
- If you are running a custom sync server or proxying `/wp-sync/v1/updates`, returning HTTP 426 with error code `rest_sync_protocol_mismatch` will now trigger the protocol-mismatch branch of the connection error modal immediately, bypassing the retry countdown.
- The PR explicitly notes that neither the "allow save" nor "disallow save" code path can guarantee no data loss during a protocol mismatch — plan accordingly for upgrade sequencing.

**Site owners / editors**
- No visible change today. If RTC protocol upgrades arrive in a future release (targeted for WP 7.1 per the discussion), stale tabs that connect to an upgraded sync server will see an immediate error dialog rather than a prolonged retry loop.

**No action required** for any audience until a server-side protocol change is shipped.

## Technical details

The diff touches two packages — `@wordpress/editor` and `@wordpress/sync` (reflected in the build size increases of +91 B and +122 B respectively).

**`packages/editor/src/utils/sync-error-messages`**
A new named export `PROTOCOL_MISMATCH` (a string constant) is added. This constant is the canonical identifier used to distinguish this error from generic connection failures.

**`packages/editor/src/components/sync-connection-error-modal/index.tsx`**
`PROTOCOL_MISMATCH` is imported alongside the existing `getSyncErrorMessages` import:

```ts
// Before
import { getSyncErrorMessages } from '../../utils/sync-error-messages';

// After
import {
    getSyncErrorMessages,
    PROTOCOL_MISMATCH,
} from '../../utils/sync-error-messages';
```

The modal component is then updated (diff truncated past line 128) to detect the `PROTOCOL_MISMATCH` code and skip the normal retry countdown, displaying the error dialog immediately. @alecgeatches confirmed this behavior: the 20-second wait is bypassed specifically for this error type.

**REST error surface**: The error the server must return to trigger this path is `rest_sync_protocol_mismatch` with HTTP status `426 Upgrade Required`, as confirmed by the test filter shared in the discussion against the `/wp-sync/v1/updates` route.

No new hooks, filters, block.json fields, or DB changes are introduced. The feature is gated behind the RTC subsystem, which is not yet shipped to stable WordPress.

## Contribution

Opened by @chriszarate. @alecgeatches contributed follow-up commits: merging trunk, adding room unregister improvements, and implementing the immediate-dialog behavior for protocol mismatch (skipping the 20-second retry). @t-hamano flagged that RTC was removed from the WordPress 7.0 milestone (see the [Make/Core post from 2026-05-08](https://make.wordpress.org/core/2026/05/08/rtc-removed-from-7-0/)), resulting in this being retargeted at 7.1. The PR was merged at `79a697a`. Discussion was limited to implementation coordination; no alternative designs were debated on the thread.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
