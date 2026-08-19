# #81697: Add `__unstable_wp_sync_storage` filter for pluggable storage backends

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @josephfusco
- **Labels:** `[Type] Enhancement`, `First-time Contributor`, `[Feature] Real-time Collaboration`
- **Merged:** [`87826e5`](https://github.com/WordPress/gutenberg/commit/87826e5c9c4e3c9f36b0fd16d775db3eabcceb08)
- **Discussion:** [#81697](https://github.com/WordPress/gutenberg/pull/81697) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds the `__unstable_wp_sync_storage` filter to Gutenberg's experimental real-time collaboration code, letting plugins replace the default `WP_Sync_Post_Meta_Storage` implementation with a custom `WP_Sync_Storage` backend. The default behavior remains post-meta storage, but the filter gives an integration point for plugins that want to avoid the cache invalidation associated with storing sync data in post meta. The filter is explicitly unstable because the collaboration sync interface is still experimental and may change.

## Impact

- **Plugin developers building real-time collaboration:** can hook `__unstable_wp_sync_storage` and return an object implementing `WP_Sync_Storage`; if the returned value does not implement that interface, the code falls back to `WP_Sync_Post_Meta_Storage`.
- **Site owners and hosting teams:** no action required; the default storage remains `WP_Sync_Post_Meta_Storage`.
- **Theme developers:** no action required.
- **Headless and REST consumers:** the diff does not change REST routes or schema, but a custom storage backend could change how collaborative sync data is persisted and served.
- **Stability:** the filter is in `lib/experiments` and is not subject to the backward-compatibility policy.

## Technical details

In `lib/experimental/collaboration/collaboration.php`, inside `gutenberg_register_collaboration_rest_routes()`, the sync storage object is now obtained through a filter before constructing `WP_HTTP_Polling_Sync_Server`.

```php
// Before
$sync_storage = new WP_Sync_Post_Meta_Storage();
$sync_server  = new WP_HTTP_Polling_Sync_Server( $sync_storage );
```

```php
// After
$sync_storage = apply_filters( '__unstable_wp_sync_storage', new WP_Sync_Post_Meta_Storage() );

if ( ! $sync_storage instanceof WP_Sync_Storage ) {
    $sync_storage = new WP_Sync_Post_Meta_Storage();
}

$sync_server = new WP_HTTP_Polling_Sync_Server( $sync_storage );
```

New filter: `__unstable_wp_sync_storage`. Expected value: `WP_Sync_Storage`. Fallback: `WP_Sync_Post_Meta_Storage`. The diff does not add database changes or REST route changes.

## Contribution

Reviewers discussed the stability of the API: one reviewer noted that real-time collaboration was returning to experiment status and that the current sync interface assumes a pure naïve relay that could change, so the filter should be named and documented as unstable. The filter was then renamed with the `__unstable_` prefix and docblock language was added. Another reviewer approved, noting that APIs under `lib/experiments` do not need to follow the backward-compatibility policy.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
