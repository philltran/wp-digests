# #79771: RTC: Remove collaboration notification defaults filter

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @shekharnwagh
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Package] Edit Post`, `[Feature] Real-time Collaboration`, `[Package] Edit Site`
- **Merged:** [`56ee6e0`](https://github.com/WordPress/gutenberg/commit/56ee6e05344f24f1e450c5792d49aa2bf1a7758c)
- **Discussion:** [#79771](https://github.com/WordPress/gutenberg/pull/79771) · 2 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

This pull request removes the `editor.CollaborationNotificationPreferenceDefaults` JavaScript filter and its associated documentation and unit tests. The change hardcodes the default values for real-time collaboration notification preferences to `true` in both the post and site editors. It matters because it eliminates a newly introduced public API that was deemed premature pending further design work, while preserving the user-facing preferences system.

## Impact

- **Plugin & theme developers:** The `editor.CollaborationNotificationPreferenceDefaults` filter is permanently removed. Any code relying on it to override collaboration notification defaults will no longer execute. Defaults are now fixed at `true`.
- **Site owners & general developers:** No action required. The UI behavior remains unchanged; user preferences still override the hardcoded defaults via the Preferences API.
- **Hosting & platform teams:** No configuration changes needed. The editor initialization path is slightly simplified.

## Technical details

The diff removes the `applyFilters( 'editor.CollaborationNotificationPreferenceDefaults', ... )` call from `packages/edit-post/src/index.js` and `packages/edit-site/src/index.js`. Instead of passing filtered defaults to `preferencesStore.setDefaults()`, the code now directly passes hardcoded `true` values for `showCollaborationJoinNotifications`, `showCollaborationLeaveNotifications`, and `showCollaborationPostSaveNotifications`. The filter's documentation is deleted from `docs/reference-guides/filters/editor-filters.md`, and the corresponding unit tests in `packages/edit-post/src/test/index.js` and `packages/edit-site/src/test/index.js` are removed. Test coverage for `use-collaborator-notifications` is simplified to isolate the hook from the editor store and preferences store, removing the filter-dependent test cases.

## Contribution

Opened and merged by @shekharnwagh (co-authored with @mamaduka) as a follow-up to PR #79184. The PR was created after post-merge feedback indicated that exposing the filter as a long-term public API required additional design work. The team opted to keep the user-facing preferences intact while removing the extensibility hook to avoid shipping an incomplete API.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
