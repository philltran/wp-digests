# #80539: RTC: Remove excess autosave notices (when not useful)

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @alecgeatches
- **Labels:** `[Type] Task`, `[Package] Core data`, `[Package] Editor`, `[Feature] Real-time Collaboration`, `No Core Sync Required`, `[Package] Sync`
- **Merged:** [`cf6b796`](https://github.com/WordPress/gutenberg/commit/cf6b796c6b06a6eabf4f63d51df3acf08dcbd6b1)
- **Discussion:** [#80539](https://github.com/WordPress/gutenberg/pull/80539) · 8 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

Real-time collaboration (RTC) in the block editor now suppresses the “more recent autosave” notice when the shared CRDT document already contains the autosaved content. Previously, because RTC autosaves target revisions rather than the parent post, WordPress always flagged a newer autosave on load, causing a persistent and misleading notice for collaborative editors. This change introduces a lightweight Yjs snapshot mechanism to verify content overlap and eliminate redundant UI warnings.

## Impact

- **RTC users & editors:** Will see fewer redundant autosave recovery notices during collaborative sessions.
- **Plugin & theme developers:** No breaking changes or deprecated APIs. The `block_editor_settings_all` filter receives a new `autosave.crdtSnapshot` property when RTC is enabled.
- **Hosting & platform:** No configuration changes required. The snapshot is stored as post meta (`_crdt_autosave_snapshot`) and is automatically ignored when RTC is disabled.

## Technical details

- PHP: `Gutenberg_REST_Autosaves_Controller` now accepts a `crdt_snapshot` request parameter and stores it as post meta via `_crdt_autosave_snapshot` (capped at `MB_IN_BYTES`). The `gutenberg_add_autosave_details_to_editor_settings()` function (hooked to `block_editor_settings_all`) passes this snapshot to the editor settings when `wp_is_collaboration_enabled()`.
- JS/TS: `saveEntityRecord` in `@wordpress/core-data` now calls `getSyncManager()?.getEntitySnapshot()` to capture a base64-encoded Yjs snapshot and attaches it to the autosave payload under `CRDT_AUTOSAVE_SNAPSHOT_KEY`. It also ensures direct edits are pushed to the CRDT via `getSyncManager()?.update()` before snapshotting.
- The editor uses a new `useEntityContainsSnapshot()` hook (imported from `../provider/use-entity-contains-snapshot`) to compare the live CRDT document against the received snapshot. If `entityContainsSnapshot()` returns true, the autosave notice is suppressed. Local autosaves in `local-autosave-monitor/index.js` were refactored to use the same snapshot verification logic.

## Contribution

During review, @chriszarate suggested storing the full CRDT document for peer-to-peer transport robustness, but the author pivoted to a `Y.Snapshot` (state vector + delete set) to keep payloads small. The initial `initialSync` provider callback was also dropped per review feedback to avoid API bloat, accepting a 3-second fallback delay in edge cases. The implementation was refined across multiple commits to address local session storage edge cases and ensure snapshot capture ordering.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
