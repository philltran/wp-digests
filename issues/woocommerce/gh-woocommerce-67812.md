# #67812: [Performance] Product ordering: start legacy hooks deprecation cycle

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @kalessil
- **Labels:** `plugin: woocommerce`
- **Merged:** [`707ee6c`](https://github.com/woocommerce/woocommerce/commit/707ee6c3a204ece4859ab1b8ff8f59dfc8fb4778)
- **Discussion:** [#67812](https://github.com/woocommerce/woocommerce/pull/67812) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

WooCommerce begins a deprecation cycle for the legacy product-ordering algorithm in `WC_AJAX::product_ordering()`. Any callback attached to `woocommerce_after_single_product_ordering` or `woocommerce_after_product_ordering` now triggers a `wc_deprecated_hook()` notice (since 11.2) because it forces a slow, full-catalog reorder path. The optimized path gains two bulk-oriented actions, `woocommerce_product_ordering_process_moved_products` and `woocommerce_product_ordering_process_reindexed_products`, as replacements for cache, sync and backup extensions.

## Impact

**Plugin & extension developers**
- Code hooked to `woocommerce_after_single_product_ordering` or `woocommerce_after_product_ordering` is deprecated as of 11.2. It still runs, but WooCommerce logs a deprecation notice. The notice text says these hooks force a non-optimized reordering path that hurts performance on large catalogs.
- Migrate to the new `woocommerce_product_ordering_process_moved_products` and `woocommerce_product_ordering_process_reindexed_products` actions. Each receives `$product_id` plus an array of product ID => `menu_order`.
- The filter `woocommerce_single_product_ordering_clean_post_cache` is removed before it shipped in a stable release (it was tagged `@since 11.2.0`). If you had started to use it to opt out of per-product `clean_post_cache`, that opt-out no longer exists.
- The reindexed action fires only when a full reindex was triggered. The moved action fires for repositioned products.

**Site owners / hosting**
- No functional change to product ordering in the admin. Sites running a plugin that hooks the legacy actions may see deprecation notices in logs or Query Monitor, and `WP_DEBUG` output on dev/staging.
- Removing the legacy hooks (by updating the offending plugin) lets the range-based fast path run, which is the intended performance benefit.

**Tests**
- Test suites that attach the legacy hooks must now call `setExpectedDeprecated()` for them.

## Technical details

**`includes/class-wc-ajax.php` (`WC_AJAX::product_ordering()`)**
- The former `$use_legacy_algorithm` check is split into `$has_per_product_hook` and `$has_post_ordering_hook`, still via `has_action()`. If either is set, the legacy branch runs as before, but now first calls `wc_deprecated_hook( '<hook>', '11.2', null, 'Using this hook forces a non-optimized reordering path...' )` for each hook present.
- In the fast path, after `ProductsOrderingMoveService::move()`, the code computes `$moved` and `$reindexed` booleans. When either is true it calls `WC_Post_Data::delete_product_query_transients()`, then:
  - if reindexed: `do_action( 'woocommerce_product_ordering_process_reindexed_products', $product_id, $modifications->reindexed )` and then `unset( $modifications->reindexed )`;
  - if moved: `do_action( 'woocommerce_product_ordering_process_moved_products', $product_id, $modifications->moved )`.
- The JSON response is still `$modifications->moved`. Both new actions are documented `@since 11.2.0`.
- An inline comment points migration to `clean_post_cache`, `wp_ajax_woocommerce_product_ordering`, and the two new actions.

**`ProductsOrderingMoveService` / `ProductsOrderingReindexService`**
- The `woocommerce_single_product_ordering_clean_post_cache` filter and its targeted-invalidation branch (`wp_cache_delete_multiple( $ids, 'posts' )` plus `wp_cache_set_posts_last_changed()`) are removed.
- Both services now always run `array_walk( $ids, 'clean_post_cache' )` when rows were updated. The comment says the targeted approach is faster but insufficient for the extensibility surface.

**Tests (`class-wc-ajax-test.php`)**
- The two legacy-path tests add `setExpectedDeprecated()` for `woocommerce_after_single_product_ordering` and `woocommerce_after_product_ordering`.
- New `test_product_ordering_fires_fast_path_hooks` moves Gamma between Delta and Echo. It asserts the reindexed payload is `[ Delta => 1 ]` and the moved payload is `[ Gamma => 2, Echo => 3, Alpha => 4, Beta => 5 ]`.

```php
// Before (legacy, now deprecated)
add_action( 'woocommerce_after_product_ordering', $cb, 10, 2 );

// After
add_action( 'woocommerce_product_ordering_process_moved_products', function ( $product_id, $moved ) { /* ID => menu_order */ }, 10, 2 );
add_action( 'woocommerce_product_ordering_process_reindexed_products', function ( $product_id, $reindexed ) { /* ... */ }, 10, 2 );
```

## Contribution

This is a follow-up to #66603, which added the test coverage the PR description says guards against regressions from this change. The record carries no substantive review debate; the discussion consists of bot comments only. The diff shows a design decision: the opt-out filter for targeted cache invalidation was dropped in favor of always firing `clean_post_cache`, favoring extension compatibility over speed.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
