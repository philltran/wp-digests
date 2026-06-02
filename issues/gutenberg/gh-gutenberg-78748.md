# #78748: RTC: Return forbidden rooms together

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @shekharnwagh
- **Labels:** `[Type] Enhancement`, `[Feature] Real-time Collaboration`, `No Core Sync Required`, `[Package] Sync`
- **Merged:** [`62dd153`](https://github.com/WordPress/gutenberg/commit/62dd153a0441339994f7443e129513e9b3103272)
- **Discussion:** [#78748](https://github.com/WordPress/gutenberg/pull/78748) · 6 comments · 0 reactions

## Summary

This Gutenberg PR changes the HTTP polling real-time collaboration (RTC) sync endpoint so a multi-room poll that hits multiple permission-denied rooms reports *all* of them in a single `403`, instead of failing one room at a time. Previously `check_permissions()` returned a `WP_Error` on the first forbidden room, so the client had to retry once per denied room — generating repeated `403`s and extra round trips — before settling on a working set. Now the server collects every failing room and returns them in the error's `data.rooms` array, and the polling client unregisters them all at once while preserving pending updates for the still-valid rooms.

## Impact

- **Plugin & feature developers (RTC/Sync):** Anyone consuming the `wp-sync/v1/updates` REST endpoint or the `@wordpress/sync` HTTP polling provider gets a faster, single-pass resolution of forbidden rooms. The `403` error now carries structured `data.rooms` (array of denied room names) in addition to the existing message text.
- **Backward compatibility:** No breaking change. Denied room names remain embedded in the human-readable `message`, so older clients relying on message substring matching still work. The client also filters `data.rooms` against the rooms it actually requested, ignoring unrelated names in a response.
- **Core/platform note:** RTC has been removed from WordPress trunk (it is no longer in core; potentially moving to a feature plugin). This change lives only in Gutenberg and is tagged `No Core Sync Required` — it currently sits in `lib/compat/wordpress-7.1/` but is not subject to core-sync CI.
- **Site owners / end users:** No action required. The practical effect is fewer spurious `403`s and no false "Connection lost" modal when an editor opens a page that references template/template-part rooms they cannot edit.

## Technical details

**Server** — `lib/compat/wordpress-7.1/class-wp-http-polling-sync-server.php`, `check_permissions()`:

The per-room loop no longer returns early. It accumulates failures into `$forbidden_rooms` and, after the loop, returns one `WP_Error`:

```php
// Before: return on first failure
if ( ! $this->can_user_sync_entity_type( ... ) ) {
    return new WP_Error(
        'rest_cannot_edit',
        sprintf( __( 'You do not have permission to sync this entity: %s.', 'gutenberg' ), $room ),
        array( 'status' => rest_authorization_required_code() )
    );
}

// After: collect, then return all
$forbidden_rooms[] = $room;
// ...after loop...
if ( ! empty( $forbidden_rooms ) ) {
    return new WP_Error(
        'rest_cannot_edit',
        sprintf( __( 'You do not have permission to sync one or more entities: %s.', 'gutenberg' ), implode( ', ', $forbidden_rooms ) ),
        array(
            'status' => rest_authorization_required_code(),
            'rooms'  => $forbidden_rooms,
        )
    );
}
```

The error code stays `rest_cannot_edit` with `data.status` 403; the new `data.rooms` key is the structured payload.

**Client** — `packages/sync/src/providers/http-polling/polling-manager.ts`:

The `WPRestError` type gains `data: { status: number; rooms?: string[] }`. The previous `identifyForbiddenRoom()` helper — which substring-matched the error message against requested room names, with a length-descending sort to avoid `postType/post:1` matching `postType/post:10` — is **removed entirely**. `handleForbiddenError()` now reads `error.data.rooms`, filters it to the names present in the failing request payload (`requestedRoomNames` Set), and unregisters each via `unregisterRoom( room, { sendDisconnectSignal: false } )`. Pending updates for the remaining valid rooms are restored for retry on the next poll cycle. If `data.rooms` is absent (a generic auth failure), it falls back to treating the error as affecting all rooms.

**Tests:** The prefix-collision test (`unregisters the correct room when room names share a prefix`) is dropped since message matching is gone. New JS tests cover multi-room forbidden responses and defensive filtering of rooms not in the request. PHP `test_sync_permission_checked_per_room` (ticket 77243) now asserts `data['data']['rooms']` equals the full forbidden set and that `message` contains each denied room name.

## Contribution

Authored by **@shekharnwagh** (with AI assistance noted: OpenClaw for code, Codex for the PR description), closing issue #77243. Props recorded for **@shekharnwagh**, **@alecgeatches**, and **@tyxla**. The main discussion point was labeling: the author applied `No Core Sync Required` because RTC was removed from core trunk. **@alecgeatches** asked **@tyxla** to confirm; **@tyxla** verified RTC is no longer in trunk and that RTC work in Gutenberg need not live in the 7.1 compat directory (and could move to its own directory to avoid core-sync CI), with RTC possibly becoming a feature plugin. The author merged after confirmation, deferring the directory-relocation decision to a follow-up.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
