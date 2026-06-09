# #64326: Fix order items being lost when order resume fails mid-checkout

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @kraftbj
- **Labels:** `plugin: woocommerce`
- **Merged:** [`49441af`](https://github.com/woocommerce/woocommerce/commit/49441af86a02ed076f676054a1958a407c11d7ee)
- **Discussion:** [#64326](https://github.com/woocommerce/woocommerce/pull/64326) · 5 comments · 1 reactions

## Summary

WooCommerce defers order item database deletions to prevent data loss during checkout resume failures. Previously, `WC_Abstract_Order::remove_order_items()` executed immediate row deletions, leaving orders with zero line items if a subsequent exception prevented `$order->save()`. The fix schedules bulk deletions until the next `save_items()` call, ensuring items are only removed from the database atomically alongside their persistence. A defensive check in `WC_Checkout::create_order()` now verifies that persisted orders contain line items when the cart still contains products, throwing an exception to catch silent save failures.

## Impact

- **Plugin & Theme Developers:** The `woocommerce_removed_order_items` action now fires during `save_items()` rather than synchronously within `remove_order_items()`. Callbacks that inspect database state immediately after calling `remove_order_items()` will see rows still present until the next save completes. In-memory state via `get_items()` remains unaffected.
- **Checkout & Payment Integrations:** Shortcode-based checkout (`[woocommerce_checkout]`) and Store API paths now include a post-save guard that throws an `Exception` if a persisted order ends up with zero items while the cart still contains products. No direct code changes are required unless custom gateways or extensions explicitly handle this new exception.
- **Action Required:** Review any mu-plugins or themes hooking into `woocommerce_removed_order_items` and verify they no longer assume immediate database persistence state. Existing integrations should remain functional but may need adjustment if they paired pre/post hooks on the same call stack expecting synchronous DB removal.

## Technical details

- **File:** `plugins/woocommerce/includes/abstracts/abstract-wc-order.php`
  - Added protected properties `$item_types_to_bulk_delete` (array) and `$bulk_delete_all_items_pending` (bool) to queue deletions.
  - Modified `remove_order_items()` to clear in-memory item groups immediately (`$this->items[ $group ] = array()`) but only populate the deletion queue. Non-string `$type` values now trigger `wc_doing_it_wrong()` and return early.
  - Relocated DB deletion and hook firing into `save_items()`, which drains the queue type-by-type. Each successful `delete_items()` call fires `woocommerce_removed_order_items`. If a callback throws during this phase, remaining queued types persist for the next `save()` call.
  - Introduced `get_item_types_to_group()` to ensure extension-registered item groups from `woocommerce_order_type_to_group` are properly cleared in-memory alongside defaults.
- **File:** `plugins/woocommerce/includes/class-wc-checkout.php`
  - In `WC_Checkout::create_order()`, after `$order->save()`, a new guard re-fetches the order via `wc_get_order()` and checks `count($persisted_order->get_items())`. Throws an `Exception` with message "Order items could not be saved. Please try again." if empty.
- **Before/After Behavior:**
  ```php
  // Before: DB deleted immediately, hook fires synchronously
  $order->remove_order_items(); 
  // DB rows gone here. woocommerce_removed_order_items fires now.

  // After: DB deleted on save(), hook fires post-delete
  $order->remove_order_items(); 
  // In-memory cleared, but DB rows persist. Hook queued until save().
  $order->save(); // Triggers actual deletion and hook firing.
  ```

## Contribution

Merged by @kraftbj into the trunk branch (`49441af`). The PR originated from production reports where off-site gateway resumptions and stale coupon meta caused mid-checkout exceptions that left orders with zero line items. Review feedback focused on tightening type guards for `remove_order_items()`, ensuring custom item groups registered via `woocommerce_order_type_to_group` are cleared in-memory, and expanding test coverage to verify hook timing and failure resilience. The author explicitly noted that full DB transaction support during the save lifecycle was considered but left out of scope for this patch.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
