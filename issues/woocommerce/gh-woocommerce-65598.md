# #65598: Deprecate QueryFilters class

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @dinhtungdu
- **Labels:** `plugin: woocommerce`
- **Merged:** [`a9a659c`](https://github.com/woocommerce/woocommerce/commit/a9a659c9c0cecc36f7b50f3f39287f673f4f905b)
- **Discussion:** [#65598](https://github.com/woocommerce/woocommerce/pull/65598) · 3 comments · 0 reactions

## Summary

WooCommerce 11.0 deprecates the `Automattic\WooCommerce\Blocks\QueryFilters` class, which historically handled product query filtering for shop archives and widgets. The class is converted into a compatibility wrapper that emits `wc_deprecated_function()` notices and delegates all filtering logic to the newer `QueryClauses` and `FilterDataProvider` service classes. Full removal is scheduled for WooCommerce 12.0.

## Impact

- **Plugin & Theme Developers**: Instantiating or extending `Automattic\WooCommerce\Blocks\QueryFilters` will trigger deprecation warnings. Existing query/filter functionality continues to work via delegation, but developers should migrate direct uses to `QueryClauses` and `FilterDataProvider`.
- **Hosting & Platform Teams**: No action required; core WooCommerce store functionality remains intact.
- **Action Required**: Update any custom code directly invoking `QueryFilters::main_query_filter()`, `add_query_clauses()`, or counting methods to use the replacement service classes.

## Technical details

- The primary change occurs in `plugins/woocommerce/src/Blocks/QueryFilters.php`, where the class and all its public methods are marked `@deprecated 11.0.0`.
- Each method (`__construct`, `init`, `main_query_filter`, `add_query_clauses`, `get_filtered_price`, `get_stock_status_counts`, `get_rating_counts`, `get_attribute_counts`) now calls `wc_deprecated_function()` pointing to `QueryClauses` or `FilterDataProvider` as replacements.
- The internal query logic (previously handling stock, price, rating, and attribute clauses via direct `$wpdb` queries and `WP_Query` hooks) has been removed in favor of delegation through `wc_get_container()->get( FilterDataProvider::class )->with( QueryClauses::class )`.
- In `plugins/woocommerce/src/Blocks/Domain/Bootstrap.php`, the initialization call `$this->container->get( QueryFilters::class )->init();` has been removed from the non-Store API bootstrap path, ensuring the deprecated class is no longer instantiated on boot.
- PHPStan baseline suppressions for `QueryFilters` have also been cleaned up.

## Contribution

Submitted by @dinhtungdu and merged in commit [`a9a659c`](https://github.com/woocommerce/woocommerce/commit/a9a659c), this change closes issue #56848 as part of the broader Blocks monorepo migration effort. The PR description notes that the class had already been unused since the blocks migration, making the deprecation a cleanup task rather than a functional overhaul. Standard automated reviews passed without requiring substantive code changes beyond the stub conversion and bootstrap cleanup.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
