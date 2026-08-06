# #80832: Views: honor all developer-defined view config overrides

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @oandregal
- **Labels:** `[Type] Bug`, `Backported to WP Core`
- **Merged:** [`e7ea49d`](https://github.com/WordPress/gutenberg/commit/e7ea49d55c0dbdbcd3b4e5ef7d6029c3d51131c4)
- **Discussion:** [#80832](https://github.com/WordPress/gutenberg/pull/80832) · 16 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `@wordpress/views` package now resolves view configuration as a proper layer stack, ensuring that developer-defined `view_list` overrides for `type`, `perPage`, and `fields` are actually applied. Previously, these overrides were silently ignored, and the persistence model stored the entire view state, causing plugin defaults to be overwritten by stale user preferences. The fix also restricts `page` and `search` to URL-only management and removes them from the REST schema.

## Impact

- **Plugin & theme developers**: `view_list` overrides for `type`, `perPage`, and `fields` now correctly apply. User preferences no longer permanently lock out plugin-defined defaults or layouts.
- **REST API consumers**: `search` and `page` are removed from the base view config schema returned by the REST endpoint.
- **No immediate code changes required** for existing implementations, but developers relying on the old (broken) override behavior or expecting `search`/`page` in the REST schema will see changes.

## Technical details

The diff replaces the monolithic `packages/views/src/filter-utils.ts` with a new `packages/views/src/resolve-view.ts` that implements a layered resolution algorithm. `load-view.ts` and the `useView` hook now call `resolveView()`, which merges `defaultView`, `defaultLayouts`, `activeViewOverrides`, and `persistedView` using a recursive `mergeLayer()` function that merges plain objects leaf-by-leaf. Only differences between the resolved view and the base are persisted via `diffLayer()`, meaning user modifications are stored as partial overrides rather than full view objects. `page` and `search` are explicitly excluded from persistence and schema. On the PHP side, `lib/compat/wordpress-7.1/class-gutenberg-rest-view-config-controller-7-1.php` strips `search` and `page` from `get_view_base_schema()`, and `lib/compat/wordpress-7.1/class-gutenberg-view-config-data.php` updates its docblock to reflect that `default_view` no longer manages `search`.

## Contribution

Opened and merged by @oandregal with co-authors @ntsekouras and @jorgefilipecosta. The PR addresses a long-standing bug where `view_list` overrides were ignored. During review, the author noted a fundamental tension in the preferences model: storing full views vs. storing only user changes. A cleaner, `theme.json`-style hierarchy (core < plugin < user) was discussed but deferred to a future release to avoid breaking changes during the 7.1 RC cycle. Support for `search` and `page` overrides was also explicitly deferred after testing confirmed URL-only management was safer. The change was backported to WordPress Core via PR #12766.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
