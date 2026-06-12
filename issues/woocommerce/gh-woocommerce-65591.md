# #65591: Performance: coalesce variation batch transient deletion

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @chubes4
- **Labels:** `needs: author feedback`, `plugin: woocommerce`, `type: community contribution`
- **Merged:** [`7c1a127`](https://github.com/woocommerce/woocommerce/commit/7c1a127bef814dd72fd56cec7ccb64817b011bb6)
- **Discussion:** [#65591](https://github.com/woocommerce/woocommerce/pull/65591) · 7 comments · 0 reactions

## Summary

Coalesces repeated product transient deletion during REST API variation batch writes on `/wc/v3/products/{product_id}/variations/batch`. Previously, each individual variation update triggered immediate transient clears and action hooks, creating significant overhead during bulk imports or migrations. This change defers all deletions to a single flush at the end of the batch operation, reducing transient clear operations by ~79% and cutting related query counts in benchmarked workloads.

## Impact

- Plugin & theme developers: The `woocommerce_delete_product_transients` action now fires once per unique product ID at the end of a deferred block rather than immediately per call. If you hook into this action to clear caches, sync external systems, or trigger side effects based on transient deletion, expect execution to occur after batch operations complete rather than inline with each individual update.
- Platform & hosting teams: No code changes or configuration required; the optimization applies automatically within WooCommerce's internal REST controller and cache layers.
- Store owners/merchants: No visible behavior change; import and bulk editing performance improves transparently without altering API contracts or data structures.

## Technical details

The change introduces `ProductTransientsDeferrer` (new class in `plugins/woocommerce/src/Internal/Caches/ProductTransientsDeferrer.php`) which manages a nesting-level counter and a set of deferred product IDs. It registers a `shutdown` handler on the first `start_deferring()` call and flushes collected IDs when the outermost `stop_deferring()` completes.

- `wc_delete_product_transients()` (`plugins/woocommerce/includes/wc-product-functions.php`) delegates to `$deferrer->maybe_defer_deletion( $post_id )`, returning early if deferral is active, otherwise falling through to `ProductUtil`.
- `class-wc-rest-product-variations-v2-controller.php` wraps `parent::batch_items($request)` in a try/finally block that calls `$transients_deferrer->start_deferring()` and `$transients_deferrer->stop_deferring()`, ensuring flush occurs after the batch completes or fails.
- `ProductUtil::delete_product_transients_for_products()` (new method in `plugins/woocommerce/src/Internal/Utilities/ProductUtil.php`) deduplicates the collected IDs, deletes fixed transients (`wc_products_onsale`, `wc_featured_products`, etc.) exactly once, fires `do_action( 'woocommerce_delete_product_transients', $product_id )` per unique ID, and coalesces parent variation transient deletion (`delete_transient_wc_product_children_...`) to a single execution per parent.

```php
// Previous behavior (called per variation in loop)
wc_delete_product_transients( $variation_id ); // Immediate delete & hook fire

// New batch controller wrapper
deferrer->start_deferring();
try {
    // parent::batch_items() internally calls wc_delete_product_transients()
    // IDs are collected, not deleted immediately
} finally {
    deferrer->stop_defering(); // Flushes once at end
}
```

## Contribution

Opened by @chubes4 as a targeted performance fix for WooCommerce issue #26029. Initial iterations proposed a filter-based approach but were revised per review feedback to use an internal service class (`ProductTransientsDeferrer`) in `Internal\Caches`. PHPStan baseline entries were cleaned up during review, and benchmark evidence was validated across multiple Homeboy rig runs with varying batch sizes. Merged as commit 7c1a127 into the upcoming milestone as a patch-level performance improvement.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
