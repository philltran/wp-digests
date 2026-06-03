# #78569: Post Taxonomies: Drop redundant `per_page: -1` from taxonomy queries

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Performance`, `[Package] Editor`
- **Merged:** [`4c5f96b`](https://github.com/WordPress/gutenberg/commit/4c5f96b83921e25fed706a5a0d66dd8a0706efb8)
- **Discussion:** [#78569](https://github.com/WordPress/gutenberg/pull/78569) · 3 comments · 0 reactions

## Summary

The block editor data layer no longer passes a redundant `per_page: -1` argument to taxonomy queries. This change strips the unused parameter from the `getEntityRecords` call for the `taxonomy` entity, aligning the client request with the fact that the `/wp/v2/taxonomies` REST endpoint ignores pagination parameters entirely. The cleanup eliminates unnecessary middleware processing and simplifies the outgoing HTTP request without altering core data-fetching behavior.

## Impact

- **Editor Users & Frontend Developers**: No action required. The underlying taxonomy retrieval logic remains unchanged.
- **Plugin & Theme Developers**: No action required. The `taxonomy` entity resolver and REST endpoint behavior are identical; passing or omitting `per_page` on the client has no functional difference.
- **Platform/Infrastructure**: Negligible performance impact limited to slightly cleaner query strings in editor network requests. No breaking changes, deprecations, or migration steps.

## Technical details

In `packages/editor/src/components/post-taxonomies/check.js`, the `getEntityRecords( 'root', 'taxonomy', ... )` call is updated to drop its third argument (the query args object). 

Previously:
```javascript
getEntityRecords( 'root', 'taxonomy', { per_page: -1 } )
```

Now:
```javascript
getEntityRecords( 'root', 'taxonomy' )
```

Because the `taxonomy` entity is already flagged with `supportsPagination: false` (introduced in PR #76406), the resolver's paginated branch is skipped. The REST `/wp/v2/taxonomies` route returns a keyed object rather than an array, causing `fetchAllMiddleware` to short-circuit after rewriting the vestigial parameter to `per_page=100`. Removing the arg collapses the wire URL from `GET /wp-json/wp/v2/taxonomies?context=edit&per_page=100` to a clean `GET /wp-json/wp/v2/taxonomies?context=edit`.

## Contribution

Opened and merged by @ellatrix. The PR addresses a minor data-layer inefficiency identified after the `supportsPagination: false` flag was applied to the `taxonomy` entity. Discussion was minimal, focused on confirming that the `per_page` parameter was purely cosmetic noise due to the existing pagination bypass and middleware rewrite logic. No alternative approaches or design debates were noted beyond verifying local test results.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
