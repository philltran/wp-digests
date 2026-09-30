# #68302: Aggregate the customer and review report totals in one query

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @nerrad
- **Labels:** `plugin: woocommerce`, `Performance`
- **Merged:** [`cde33c6`](https://github.com/woocommerce/woocommerce/commit/cde33c6b4dbeb0fc14d7872df7ca7e531d80a3ba)
- **Discussion:** [#68302](https://github.com/woocommerce/woocommerce/pull/68302) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The WooCommerce REST endpoints `GET /wc/v3/reports/reviews/totals` and `GET /wc/v3/reports/customers/totals` now compute their totals with aggregate SQL instead of repeated or materialized queries. The reviews endpoint replaces five per-rating `get_comments()` counts with one grouped query, cutting a cold request from six queries to two. The customers endpoint no longer selects and sorts every paying-customer ID just to discard them, and counts directly instead. Response payloads and cache invalidation signals are meant to stay the same.

## Impact

**Site owners / store admins**
- Cold-cache report requests are cheaper. The author's measurements: reviews at 100k rows went from ~327 ms to ~192 ms, and customers at 100k rows dropped retained memory from ~3.5 MB to ~103 KB (peak delta ~10 MB to 0). Warm requests are unchanged.
- No action required.

**Plugin & extension developers**
- The reviews aggregate is a direct query, no longer executed through `WP_Comment_Query`. It still passes its clauses through `comments_clauses`, so callbacks that modify `where`/`join` keep applying. `comments_pre_query` and `pre_get_comments` no longer fire for this report.
- The customers count no longer goes through `users_pre_query` or `found_users_query`. The PR states a search of 975 WooCommerce extensions found no `found_users_query` callbacks and one `users_pre_query` action that mutates the query rather than short-circuiting it. `pre_user_query` still applies via `prepare_query()`.
- A `comments_clauses` callback that empties `fields`, `join`, `where` or `groupby` results in zeroed totals rather than invalid SQL.
- The automated review flagged moderate merge risk for extensions that customize user or comment queries and could get incorrect totals.

**Headless & REST consumers**
- No schema change. The reviews endpoint still returns five buckets (`rated_1_out_of_5` to `rated_5_out_of_5`), zero-filled when a rating has no reviews. The customers endpoint still returns `paying` and `non_paying`.

## Technical details

**Reviews totals** (`class-wc-rest-report-reviews-totals-controller.php`, `get_reports()`):
- Before: a loop of five `get_comments()` calls with `count => true`, `post_type => 'product'`, `meta_key => 'rating'` and `meta_value => $i`.
- After: a `WP_Comment_Query` is instantiated and `parse_query()`ed only to serve as the `comments_clauses` context. Clauses are then built by hand:
  - `fields`: `commentmeta.meta_value AS rating, COUNT(*) AS total`
  - `join`: inner joins on `$wpdb->posts` and `$wpdb->commentmeta`
  - `where`: `comment_approved IN ('0','1')`, `comment_type NOT IN ('note')`, `post_type = 'product'`, `meta_key = 'rating'`, and `meta_value IN ('1'..'5')`
  - `groupby`: `commentmeta.meta_value`
- The clauses go through `apply_filters_ref_array( 'comments_clauses', ... )`, then run via `$wpdb->get_results()`. Rows are mapped into a zero-filled `array_fill_keys( range( 1, 5 ), 0 )`.
- Caching: key `wc_report_reviews_totals_{wp_cache_get_last_changed('comment')}` in the `comment-queries` group, so the existing `last_changed` invalidation still works. Results are cached only when `$wpdb->last_error` is empty.
- Pending (`0`) reviews count. Spam, trash, `note` type, unrated, non-product posts, and ratings like `0` or a padded `05` are excluded, since ratings are compared as strings.

**Customers totals** (`class-wc-rest-report-customers-totals-controller.php`):
- Before: `new WP_User_Query( [ 'fields' => 'ID', 'number' => 0, 'count_total' => true, ... ] )` followed by `get_total()`, which materialized all matching IDs.
- After: `$q = new WP_User_Query(); $q->prepare_query( $args );` builds role, capability and site scoping plus the `paying_customer = 1` meta join. The controller then runs:

```php
$wpdb->get_var( "SELECT COUNT(*) {$customers_query->query_from} {$customers_query->query_where}" );
```

- Cache key: `wc_report_customers_totals_paying_{blog_id}_{wp_cache_get_last_changed('users')}` in the `user-queries` group. The blog ID is included because the query is site-scoped while `user-queries` is global on multisite. A `null` result (failed query) is not cached.
- `count_users()` for the total is unchanged.

**Other changes**
- The now-obsolete `get_comments` PHPStan baseline entry for the reviews controller is removed.
- A changelog file (`Type: performance`, `Significance: patch`) is added.
- Tests: customer tests now assert exact `paying_customer` matching (`'01'` and `0` don't count) and role exclusion. Review tests create fixtures via `wp_new_comment()` with review-form `$_POST` fields and via `POST /wc/v3/products/reviews`, and assert a third-party `comments_clauses` callback narrows the report.

## Contribution

Authored by @nerrad with AI assistance (Claude Code), closing issue #67850. CodeRabbit's review rated merge risk as moderate, citing compatibility for extensions that customize user or comment queries. The PR description documents the resulting design decision: the `comments_clauses` pass-through was kept, while `comments_pre_query`, `pre_get_comments`, `users_pre_query` and `found_users_query` were explicitly dropped, justified by the extension search and by those hooks being unable to represent a grouped per-rating result. A bot also reminded the author about REST API documentation.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
