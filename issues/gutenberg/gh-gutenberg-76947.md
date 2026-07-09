# #76947: RTC: Add optional `shouldSync` function to entity sync config

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @chriszarate
- **Labels:** `[Type] Task`, `[Type] Enhancement`, `[Package] Core data`, `Needs Dev Note`, `[Feature] Real-time Collaboration`, `Backported to WP Core`, `[Package] Sync`
- **Merged:** [`9e4aef2`](https://github.com/WordPress/gutenberg/commit/9e4aef27a6078ce95af9305623030595d3b4c591)
- **Discussion:** [#76947](https://github.com/WordPress/gutenberg/pull/76947) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Introduces an optional `shouldSync` callback to the Core Data entity sync configuration, allowing conditional gating of Real-Time Collaboration (RTC) for specific entities or collections. This prevents unnecessary synchronization overhead when loading static or non-collaborative content, such as pre-existing comments on a post.

## Impact

- **Block Authors & Plugin Developers**: Can now conditionally exclude entities from the RTC sync pipeline by supplying a `shouldSync` function in the entity config. Use your own logic to filter by entity type or context before WebSocket/polling connections are established.
- **Hosts & Platform Teams**: Reduces unnecessary database storage and WebSocket connections when real-time collaboration is enabled globally but not required for specific content types (e.g., loaded comment blocks).
- **No immediate action required** for default configurations, as the callback remains optional and preserves existing sync behavior when omitted.

## Technical details

The entity sync configuration schema in `packages/core-data/src/entities.js` now accepts an optional `shouldSync` property. When provided, it is invoked to determine participation; returning `false` halts the registration of the sync provider for that entity or collection. This replaces the previous all-or-nothing behavior where enabling sync on a base entity (like `comment`) automatically applied to every loaded instance, eliminating resource bloat without requiring a separate entity type or permissions overhaul.

## Contribution

Merged by @chriszarate (co-authored by @alecgeatches and @mmtr86) and backported to the `wp/7.0` branch. Opened in response to #76932, which identified that loading a comments block on a post with existing comments forced every comment into the sync loop. The team settled on an optional callback over splitting the entity type to keep the change localized to Core Data and avoid broader permissions/system changes.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
