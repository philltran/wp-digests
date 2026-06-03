# #78683: Use search_columns=post_title for parent page selector REST API searches

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @sarthaknagoshe2002
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Package] Fields`
- **Merged:** [`5eb51cc`](https://github.com/WordPress/gutenberg/commit/5eb51ccf20ac6b2b7952d715eef20e6ce301a652)
- **Discussion:** [#78683](https://github.com/WordPress/gutenberg/pull/78683) · 1 comments · 0 reactions

## Summary

The parent page selector in both the classic editor sidebar (`PageAttributesParent`) and the DataViews/Site Editor interface now passes `search_columns: ['post_title']` to the REST API when querying for candidate parent pages. Previously, the search hit `post_title`, `post_excerpt`, and `post_content` simultaneously; pages whose body or excerpt contained the search term could exhaust the `per_page: 100` budget and displace actual title matches, causing the dropdown to show zero visible options. The fix aligns the server-side query with the client-side filter, which already restricts display to title matches.

## Impact

**Plugin & theme developers**
- No action required. The change is internal to Gutenberg editor components; no public API, filter, or hook signature changed.

**Site owners / content editors**
- Parent page search in the Page Attributes sidebar and DataViews is now reliable on sites with many pages whose body content shares keywords with page titles. Searches that previously returned blank dropdowns will now return correct results.

**Hosting & platform**
- REST requests to `/wp/v2/pages` that originate from the parent page selector will now include the `search_columns=post_title` query parameter, which maps to `WP_Query`'s `search_columns` arg (introduced in WordPress 6.6). No server-side configuration needed; the parameter is supported in core.

## Technical details

Two files were modified with identical one-line additions:

- `packages/editor/src/components/page-attributes/parent.js` — classic Block Editor sidebar
- `packages/fields/src/fields/parent/parent-edit.tsx` — DataViews / Site Editor

In both cases, the `search_columns` key is appended to the `query` object only when `fieldValue` is non-empty (i.e., the user has typed a search term).

**Before:**
```js
if ( !! fieldValue ) {
    query.search = fieldValue;
    query.orderby = 'relevance';
}
```

**After:**
```js
if ( !! fieldValue ) {
    query.search = fieldValue;
    query.orderby = 'relevance';
    query.search_columns = [ 'post_title' ];
}
```

The `search_columns` REST parameter maps to `WP_Query`'s `search_columns` argument and restricts the `LIKE` clause in the generated SQL to `post_title` only, preventing `post_excerpt` and `post_content` from matching. No new hooks, filters, or REST schema fields are introduced.

## Contribution

The PR was opened by @sarthaknagoshe2002 and merged (commit `5eb51cc`) with review credit also going to @Mamaduka. It closes Gutenberg issue #74631. The fix is minimal — two single-line additions — and no alternative approaches were documented in the discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
