# #80203: Make the Playlist blocks stable

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @scruffian
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Playlist`
- **Merged:** [`b453eb9`](https://github.com/WordPress/gutenberg/commit/b453eb98b8e8edec42174c005eee633fbe60fa7d)
- **Discussion:** [#80203](https://github.com/WordPress/gutenberg/pull/80203) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `core/playlist` and `core/playlist-track` blocks have been stabilized and are now registered in the default block library, removing their experimental status. This eliminates the need for the block experiments flag to access these blocks and updates their documentation and test fixtures to reflect production readiness. Site owners and developers can now rely on these blocks without version or feature-flag constraints.

## Impact

- **Plugin & theme developers:** No code changes required. The blocks are now available by default in the block editor without enabling experimental features.
- **Site owners:** The Playlist and Playlist Track blocks will appear in the inserter and function normally in existing posts without requiring Gutenberg updates or feature flags.
- **Headless & REST consumers:** Block schema and serialization remain unchanged; only the experimental flag is removed from `block.json`.
- **No action required.** Existing content using these blocks will continue to render identically.

## Technical details

The diff removes `"__experimental": true` from `packages/block-library/src/playlist/block.json` and `packages/block-library/src/playlist-track/block.json`. In `packages/block-library/src/index.js`, the `playlist` and `playlistTrack` modules are moved from the conditional experimental registration block (guarded by `__experimentalBlocks`) to the default `getAllBlocks()` return object. This ensures they are exported and registered alongside stable core blocks. The change also strips experimental warnings from the auto-generated README files and updates integration test fixtures to use the stable `core/playlist` and `core/playlist-track` names in serialized HTML. No new hooks, filters, or REST schema changes are introduced.

## Contribution

Opened and merged by @scruffian with co-authorship from @jeryj, who added the missing required block output tests. The PR was reviewed and approved for inclusion ahead of the Gutenberg 23.6 RC release. The discussion was straightforward, focusing on test coverage and final stabilization steps rather than architectural debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
