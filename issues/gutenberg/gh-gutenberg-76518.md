# #76518: Real Time Collaboration: Introduce filters for the polling intervals.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @peterwilsoncc
- **Labels:** `[Type] Enhancement`, `Needs Dev Note`, `[Feature] Real-time Collaboration`, `Backported to WP Core`, `[Package] Sync`
- **Merged:** [`273bc0b`](https://github.com/WordPress/gutenberg/commit/273bc0b6d9b853dbba5c019d0d26e3c18b431422)
- **Discussion:** [#76518](https://github.com/WordPress/gutenberg/pull/76518) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

This change introduces four JavaScript filters to the Real Time Collaboration (RTC) client, allowing developers and hosting providers to tune the frequency of polling requests. The default polling rate can reach up to four times per second, which risks overloading low-resource or shared hosting environments when multiple editors are active. By exposing these intervals as filterable values, server load can be reduced without modifying core synchronization logic.

## Impact

['Plugin & theme developers / Hosting providers: Can significantly reduce block editor network overhead by applying custom filters to slow polling on constrained infrastructure.', 'Site owners & editors: No direct action required; tuning is transparent and applies only when explicitly hooked.', 'Migration/Configuration: Optional. Implement via `wp.hooks.addFilter` if you need to cap server requests during high-concurrency editing sessions.']

## Technical details

The modification adds four filter hooks to the sync client's polling manager, executed before applying the calculated interval. The hooks are:
- `sync.pollingManager.pollingIntervalNoCollaborators`
- `sync.pollingManager.pollingIntervalWithCollaborators`
- `sync.pollingManager.pollingIntervalMaxErrorBackoff`
- `sync.pollingManager.pollingIntervalBackgroundTab`

Each expects and returns an integer representing milliseconds. They are applied to the `wp-sync` script payload using WordPress's public hook system (`wp.hooks.addFilter`). The default fallbacks ensure existing behavior is preserved unless a filter explicitly returns a different value.

Example override pattern:
```javascript
wp_add_inline_script( 'wp-sync', "
  wp.hooks.addFilter( 'sync.pollingManager.pollingIntervalWithCollaborators', 'my-org/tune-rpc', function( interval ) {
    return 5000;
  } );
" );
```

## Contribution

Opened by @peterwilsoncc to resolve issue #76517, with co-authorship from @czarate. The PR was reviewed as a lightweight enhancement to prevent accidental resource exhaustion on shared hosting. It was marked for backporting to WordPress Core and flagged for an official Dev Note due to its utility for platform teams managing real-time editing workloads.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
