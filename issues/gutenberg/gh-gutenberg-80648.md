# #80648: wp-build: sync the page template preload field list with core-data

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Status] In Progress`, `Backported to WP Core`, `[Feature] Client Side Media`
- **Merged:** [`c3580f3`](https://github.com/WordPress/gutenberg/commit/c3580f3f2e11fd67121e10a1217232fd24d48d2b)
- **Discussion:** [#80648](https://github.com/WordPress/gutenberg/pull/80648) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Updates the `/?_fields=` preload query string in `@wordpress/build` page templates to exactly match the `_fields` list in `@wordpress/core-data`. This alignment prevents unconsumed preloads, duplicate network requests to `/`, and `[api-fetch] [preload]` console warnings on generated admin pages.

## Impact

- **Plugin & theme developers using `@wordpress/build`**: Preloads will now be correctly consumed, eliminating duplicate requests and console warnings without requiring code changes.
- **Core developers & site operators**: The font library and options connectors pages will no longer double-fetch the settings endpoint on load.
- **No action required**: The fix ships automatically with the next `@wordpress/build` package sync to WordPress core.

## Technical details

Modifies `packages/wp-build/templates/page.php.template` and `packages/wp-build/templates/page-wp-admin.php.template`. The `$preload_paths` array's first element now includes `image_max_bit_depth` and `image_strip_meta` in the `/?_fields=` query string, aligning exactly with `packages/core-data/src/entities.js`. The comment above the array was updated to explicitly state that field order must match exactly, as `api-fetch` preloading relies on exact URL matching to consume cached responses.

## Contribution

Opened and merged by @adamsilverstein with co-authors @t-hamano and @Mamaduka. Review discussion clarified that although the font library and options connectors pages do not directly use client-side media, the `__unstableBase` entity request in `@wordpress/boot` pulls from the shared `entities.js` field list, making the extra fields unavoidable but harmless. The PR was backported to the `wp/7.1` branch to ensure it ships with the next release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
