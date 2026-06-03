# #78565: Edit Post Preload: Cover remaining bound GET/OPTIONS requests on load

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Performance`, `[Package] Core data`, `[Package] Edit Post`
- **Merged:** [`ff388d8`](https://github.com/WordPress/gutenberg/commit/ff388d8617780498faaa231189a55e01bbfd012a)
- **Discussion:** [#78565](https://github.com/WordPress/gutenberg/pull/78565) · 3 comments · 0 reactions

## Summary

The Post Editor preload specification was updated to cache four previously unbound REST API requests during initialization, eliminating post-mount network roundtrips that caused brief render delays. The change proactively resolves the front-page template lookup (`GET /wp/v2/templates/lookup?slug=front-page`), current post author details (`GET /wp/v2/users/{id}`), collection-level permissions for `posts` (`OPTIONS /wp/v2/posts`), and taxonomies (`GET /wp/v2/taxonomies?context=edit`).

## Impact

- **Editor users & platform teams**: Improved perceived load performance in the Post Editor by shifting critical UI data fetches to the initial preload cache.
- **Plugin/Theme developers**: No breaking changes or migration required. The optimization is internal to core editor packages.
- **REST API consumers**: No external API behavior changes; only client-side spec assertions and resolver kickoffs were modified.

## Technical details

- **PHP preload registration**: Added four endpoints to the `block_editor_rest_api_preload_paths` filter in `lib/compat/wordpress-7.0/preload.php` for the `core/edit-post` context, dynamically resolving the user ID via `$post->post_author`.
- **JS resolver kickoffs**: Updated `packages/edit-post/src/index.js` to trigger these endpoints during editor boot:
  - Phase 1: `core.getDefaultTemplateId({ slug: 'front-page' })` and `core.canUser('create', { kind: 'postType', name: postType })` (consumes the OPTIONS cache).
  - Phase 2: `core.getUser(post.author, { context: 'view', _fields: 'id,name' })`.
- **Validation guard fix**: Tightened `getDefaultTemplateId.shouldInvalidate` in `packages/core-data/src/resolvers.js` to ignore `RECEIVE_ITEMS` actions lacking `persistedEdits`. This prevents the initial site root fetch from incorrectly invalidating the just-resolved front-page template ID, which previously forced a redundant post-mount refetch.
- **Test updates**: `test/e2e/specs/preload/post-editor.spec.js` was updated to assert these URLs no longer appear in post-mount requests. Pagination-unbounded queries (`per_page=-1`) were explicitly excluded from preloading to prevent payload bloat.

## Contribution

Opened and merged by @ellatrix as a direct follow-up to preload consolidation work in #78508. The change focuses strictly on covering the remaining bound GET/OPTIONS requests identified in the post-editor preload spec. Design trade-offs were limited to explicitly deferring pagination-unbounded queries (e.g., comments, wp_pattern_category) to keep the initial cache payload bounded.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
