# #66320: Fix Revenue report double-counting returns on partial-then-full refunds

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @louwie17
- **Labels:** `needs: documentation`, `plugin: woocommerce`
- **Merged:** [`440d3af`](https://github.com/woocommerce/woocommerce/commit/440d3aff26f5976bf33a76a6077fab5e198a1d69)
- **Discussion:** [#66320](https://github.com/woocommerce/woocommerce/pull/66320) · 6 comments · 2 reactions
- **Usefulness:** 4/5

## Summary

Fixes a bug in WooCommerce Analytics where the Revenue report double-counted returns when an order received a partial refund followed by a full refund. Previously, the full-refund row in `wc_order_stats` recorded the entire parent order total regardless of prior partial refunds, causing the Returns metric to overstate the actual refunded amount. The fix adjusts how full-refund rows calculate their totals and how refund rows are classified in the aggregation query.

## Impact

- **WooCommerce store owners & analytics consumers:** Revenue reports will now correctly reflect Returns, Net Sales, Taxes, and Shipping for orders with multiple refunds.
- **Existing stores:** Affected historical data will remain double-counted until analytics are re-synced. Run `wp action-scheduler run` and `wp wc tool run clear_woocommerce_analytics_cache` to refresh `wc_order_stats`.
- **Plugin/theme developers:** No direct API changes or breaking changes. The fix operates internally within the Analytics data store.
- **No action required** for stores that do not issue partial-then-full refunds or do not rely on the Analytics Revenue report.

## Technical details

The diff modifies `plugins/woocommerce/src/Admin/API/Reports/Orders/Stats/DataStore.php`:
- In `assign_report_columns()`, the refund detection SQL now checks `( net_total + tax_total + shipping_total ) < 0` instead of `net_total < 0`. This ensures rows where the net portion is already zeroed out by a prior partial refund are still captured if tax or shipping remain negative.
- In `update()`, when a full refund derives amounts from the parent order (`$use_parent_refund_amounts`), the code now iterates through `$parent_order->get_refunds()` to subtract amounts already recorded by earlier refunds. This prevents the full-refund row from re-booking the partial amount.
- Before/after logic for the full-refund row calculation:
  ```php
  // Before: always used full parent totals
  $data['net_total'] = -1 * self::get_net_total( $parent_order );
  // After: subtracts prior refund amounts
  foreach ( $parent_order->get_refunds() as $prior_refund ) {
      if ( $prior_refund->get_id() === $order->get_id() ) continue;
      $data['net_total'] -= self::get_net_total( $prior_refund );
      // ... same for tax, shipping, num_items_sold
  }
  ```
- A regression test `test_partial_then_full_refund_does_not_double_count_returns` was added to `DataStoreTest.php` to assert correct aggregation in `wc_order_stats`.

## Contribution

Opened and merged by @louwie17, addressing issue #66217 diagnosed by @PypWalters. The PR was intentionally scoped to only fix the Revenue report double-counting, with two related follow-ups (Orders report cosmetic bug and refund deletion re-import) explicitly deferred to keep the change low-risk. The author incorporated reviewer feedback to trim the scope and ensure the fix remained focused on the `Stats/DataStore` aggregation logic.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
