# #78097: Make RTC-related APIs private

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Code Quality`, `[Package] Core data`, `[Package] Editor`, `[Package] Edit Post`, `[Feature] Real-time Collaboration`, `Backported to WP Core`
- **Merged:** [`a8cc6ce`](https://github.com/WordPress/gutenberg/commit/a8cc6ce39cc1060c713cdc2f1901417c1fe27519)
- **Discussion:** [#78097](https://github.com/WordPress/gutenberg/pull/78097) · 24 comments · 0 reactions

## Summary

Five Real-Time Collaboration (RTC) APIs that had been publicly exposed in the Gutenberg plugin are now routed through the `@wordpress/private-apis` lock/unlock mechanism, making them inaccessible from the global `wp` namespace. The affected symbols are the `core` store selector `getSyncConnectionStatus`, the `core` store action `setSyncConnectionStatus`, the `core/editor` selector `isCollaborationEnabledForCurrentPost`, and the `@wordpress/core-data` TypeScript enums `SelectionType` and `SelectionDirection`. This is part of the Gutenberg 7.0 effort to disable RTC before it ships to WordPress core, and the PR is backported to core.

## Impact

**Plugin & theme developers**
- Calls to `wp.data.select('core').getSyncConnectionStatus()`, `wp.data.dispatch('core').setSyncConnectionStatus(…)`, and `wp.data.select('core/editor').isCollaborationEnabledForCurrentPost()` will now return `undefined` — these selectors/actions are no longer registered on the public store interface.
- `wp.coreData.SelectionType` and `wp.coreData.SelectionDirection` are no longer exported from the public package surface.
- Access requires going through the `@wordpress/private-apis` `lock`/`unlock` pattern, which is intentionally not a stable public API.
- If you are not consuming any RTC APIs, no action is required.

**Headless & REST consumers**
- No REST API or block.json changes; this is a JavaScript data-store API surface change only.

**Hosting & platform**
- The backport to WP core means these APIs were never part of a stable core release; no migration is needed for core-only deployments.

## Technical details

The diff removes the public documentation entries for `isCollaborationEnabledForCurrentPost` from `docs/reference-guides/data/data-core-editor.md` and removes `getSyncConnectionStatus`/`setSyncConnectionStatus` from `docs/reference-guides/data/data-core.md`, reflecting that these are no longer part of the public API contract.

On the implementation side, the exports in `packages/core-data`, `packages/editor`, and `packages/edit-post` are moved behind the `@wordpress/private-apis` boundary. The +46 B bundle delta across `build/scripts/core-data/index.min.js` (+7 B), `build/scripts/edit-post/index.min.js` (+5 B), and `build/scripts/editor/index.min.js` (+34 B) reflects the wrapping overhead of the private-APIs lock calls.

The original approach was to gate exposure on the `IS_GUTENBERG_PLUGIN` compile-time constant — so these selectors/actions would silently disappear in the core build while remaining public in the plugin build. This was abandoned in favour of unconditional privatization: the APIs are now private in both the plugin and core builds. E2E tests that previously called these APIs directly were updated to use the `unlock()` accessor internally.

**Before** (publicly accessible):
```js
wp.data.select('core').getSyncConnectionStatus();
wp.data.dispatch('core').setSyncConnectionStatus('connected');
wp.data.select('core/editor').isCollaborationEnabledForCurrentPost();
```

**After** (returns `undefined`; must use private-apis unlock to access internally):
```js
wp.data.select('core').getSyncConnectionStatus(); // undefined
```

## Contribution

Opened by @t-hamano as one of a coordinated set of PRs (#78080, #78081, #78085) and Trac ticket #65205 targeting the Gutenberg 7.0 / core backport window. @Mamaduka pushed back on the initial `IS_GUTENBERG_PLUGIN`-gate approach, arguing the convention is that APIs only become truly public once they ship in core, so there was nothing to protect. @ellatrix noted the `@wordpress/sync` package was already being hidden separately in #78085. @t-hamano revised the PR to unconditional privatization, reasoning that keeping API specs flexible until RTC is ready for a stable release is preferable to an unnecessary `IS_GUTENBERG_PLUGIN` toggle that would need to be removed later. Reviewed and merged with props to @talldan, @alecgeatches, @Mamaduka, @ellatrix, @youknowriad, and @mmtr.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
