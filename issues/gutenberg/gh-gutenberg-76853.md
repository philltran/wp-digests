# #76853: RTC: Change SyncConnectionModal to isSyncConnectionErrorHandled filter and drop IS_GUTENBERG_PLUGIN check

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ingeniumed
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `Needs Dev Note`, `[Feature] Real-time Collaboration`, `Backported to WP Core`
- **Merged:** [`ec73dd8`](https://github.com/WordPress/gutenberg/commit/ec73dd8957703b34358e8bfc26a8e9e73b81be5e)
- **Discussion:** [#76853](https://github.com/WordPress/gutenberg/pull/76853) · 9 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Decouples the Real-Time Collaboration (RTC) sync connection error modal from the Gutenberg plugin dependency by removing the `IS_GUTENBERG_PLUGIN` conditional check. The change renames the existing filter to `isSyncConnectionErrorHandled` and restructures it as a signal pattern, allowing any plugin or theme to intercept connection failures and render custom UI without requiring the Gutenberg plugin to be active.

## Impact

['**Plugin & theme developers:** Can now override or completely replace the default RTC error modal. The new `isSyncConnectionErrorHandled` filter acts as a signal; returning a non-null value suppresses the built-in modal and hands control to the custom implementation.', '**Site owners & editors:** No immediate change to default behavior unless they adopt a plugin that hooks into this signal.', '**Migration required:** Any existing customization using the previous `SyncConnectionModal` filter name must update to the new pattern. Core default rendering remains unchanged.']

## Technical details

The unified diff removes the `IS_GUTENBERG_PLUGIN` check that previously gated the editor's RTC error modal logic. The hook is renamed to `isSyncConnectionErrorHandled` (referenced as `editor.SyncConnectionError` in implementation examples) and converted from a data-passing filter into a boolean-like signal: `wp.hooks.applyFilters( 'editor.SyncConnectionError', null, errorCode )`. If a plugin's callback returns a truthy value, the editor skips rendering the default modal and assumes responsibility for UI placement. This shifts the RTC error contract from a fixed props payload to a flexible take-over signal.

## Contribution

Opened by @ingeniumed with technical review from @alecgeatches and @maxschmeling. Initial iterations proposed a fixed filter contract passing modal props, but reviewers noted that hardcoding a stable prop schema would restrict future UI needs (e.g., adding marketing assets or custom layouts). The team pivoted to a broadcast/signal pattern where plugins simply return non-null values to claim rendering control. Merged as commit ec73dd8 and flagged for backport to WordPress Core.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
