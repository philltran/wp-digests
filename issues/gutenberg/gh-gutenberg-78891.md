# #78891: RTC: Add separate doc persistence endpoint

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @alecgeatches
- **Labels:** `[Type] Task`, `[Package] Core data`, `[Feature] Real-time Collaboration`, `No Core Sync Required`, `[Package] Sync`
- **Merged:** [`05bf6da`](https://github.com/WordPress/gutenberg/commit/05bf6da85b4d5ec7465f59c0c915614bddbae70d)
- **Discussion:** [#78891](https://github.com/WordPress/gutenberg/pull/78891) · 6 comments · 0 reactions

## Summary

The Real-Time Collaboration (RTC) module now uses a dedicated `wp-sync/v1/save` REST endpoint to persist CRDT document snapshots independently of standard post saves. This change eliminates unintended side effects when repairing or syncing CRDT state on entity load, such as triggering `wp_update_post` hooks, updating the `post_modified` date, and incorrectly disabling the editor's Publish button via `saveEntityRecord`. User-initiated saves continue to bundle the CRDT document normally with the post payload.

## Impact

- Plugin & theme developers: No action required. The endpoint is internal to `@wordpress/sync` and `@wordpress/core-data`.
- Platform/Hosting teams: The new `wp-sync/v1/save` route bypasses the full post save lifecycle, reducing server overhead during background CRDT repairs but still requiring `edit_post` capability validation per room.
- Headless & REST consumers: No change. Existing `wp/v2` routes and standard Gutenberg persistence flows remain unaffected.

## Technical details

- Introduces `WP_Sync_Config` (`lib/compat/wordpress-7.1/class-wp-sync-config.php`) and `WP_Sync_Save_Server` (`lib/compat/wordpress-7.1/class-wp-sync-save-server.php`).
- `WP_Sync_Config::ENTITY_CONFIG` defines `supports_crdt_doc_persistence: true` exclusively for `postType` entities. Unsupported types (e.g., taxonomy terms) are now explicitly skipped for CRDT persistence, preventing no-op save requests.
- The new `POST /wp-sync/v1/save` endpoint parses the `room` parameter, validates permissions via `WP_Sync_Config::can_user_sync_entity_type()`, and directly calls `update_post_meta()` with key `_crdt_document`. It bypasses `wp_update_post()`, post-save hooks, and UI save states.
- In `packages/core-data/src/resolvers.js`, the `persistCRDTDoc` callback inside `getEntityRecord` now short-circuits if `entityConfig.syncConfig?.supportsPersistence` is falsy. When supported, the client sync manager calls `saveCRDTDoc()`, serializes the Yjs document, and POSTs it to the new endpoint, queuing saves per-room to prevent stale overwrites.
- Before/after pattern change:
  ```js
  // Before: Triggered saveEntityRecord for bookkeeping, causing side effects
  persistCRDTDoc: () => resolveSelect.getEditedEntityRecord( ... ).then( (record) => { /* saveEntityRecord call */ })
  
  // After: Validates support flag first, then uses dedicated sync endpoint
  persistCRDTDoc: () => {
      if ( ! entityConfig.syncConfig?.supportsPersistence ) return;
      return resolveSelect.getEditedEntityRecord( ... ).then( async (editedRecord) => { /* serialize & POST to /wp-sync/v1/save */ })
  }
  ```
- E2E test utility `waitForConvergence()` removed the `includeCrdtDocument` flag. Peer refetches are no longer triggered for background repairs, as live Yjs state supersedes local meta and explicit post saves already bundle the doc.

## Contribution

Opened by @alecgeatches to address design concerns raised by @jsnajdr regarding CRDT persistence side effects during entity loading. The PR isolated CRDT bookkeeping into a dedicated REST endpoint, removing unnecessary `wp_update_post` execution and UI state mutations. After review cycles addressing permission handling and test alignment (including a behavior shift for the `waitForConvergence` E2E utility), it was merged as commit `05bf6da`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
