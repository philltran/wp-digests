# #64407: fix(analytics): prevent fatal when plain WC_Order passed to customer DataStore

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @faisalahammad
- **Labels:** `needs: author feedback`, `plugin: woocommerce`, `type: community contribution`
- **Merged:** [`5963d78`](https://github.com/woocommerce/woocommerce/commit/5963d78299d3ee86ae78e47c9eb38e310c06c3a7)
- **Discussion:** [#64407](https://github.com/woocommerce/woocommerce/pull/64407) · 13 comments · 6 reactions
- **Usefulness:** 3/5

## Summary

Resolves three fatal error paths in WooCommerce's Analytics Customers DataStore (`DataStore.php`). The patch prevents crashes when a plain `WC_Order` instance is passed without the `woocommerce_order_class` filter, hardens date field resolution against null values that trigger `getTimestamp()` failures, and adds a strict guard for failed `wc_get_order()` lookups.

## Impact

- **Plugin & theme developers**: Custom integrations that instantiate or mock `WC_Order` directly will no longer trigger fatal class-method errors when passed to customer reporting APIs. No migration required.
- **Site owners & platform teams**: Analytics customer sync and report caching are hardened against corrupted, missing, or improperly instantiated order objects, preventing admin/API 500s.
- **Action required**: None. This is a transparent data-store boundary fix.

## Technical details

- Modified `plugins/woocommerce/src/Admin/API/Reports/Customers/DataStore.php`.
- `get_or_create_customer_from_order()` & `get_customer_order_data_and_format()`: Both methods now validate `$order instanceof OverridesOrder`. If a plain `\WC_Order` is passed, it is re-resolved via `new \Automattic\WooCommerce\Admin\Overrides\Order( $order->get_id() )` before proceeding, ensuring consistent name resolution (user meta → billing → shipping).
- Date fallback cascade: Replaces direct `$order->get_date_created( 'edit' )->getTimestamp()` with a null-safe chain:
  ```php
  $date_created = $order->get_date_created( 'edit' )
      ?? $order->get_date_modified( 'edit' )
      ?? $order->get_date_paid( 'edit' );
  // ... uses `null` instead of calling getTimestamp() on false/null
  ```
- `sync_order_customer()`: Replaces implicit type checks with explicit `! $order instanceof \WC_Order`, returning `-1` immediately when `wc_get_order()` fails.
- PHPStan baseline updated to remove false-positive warnings for object-type method calls on the DataStore's internal parameters.

## Contribution

Opened by @faisalahammad and merged in `5963d78`. Reviewers @gigitux, @kraftbj, and @Konamiman debated the handling strategy for plain vs. overridden order instances (@kraftbj warned against dropping straight to billing field accessors; @Konamiman advocated for early class conversion, which was ultimately adopted). The author iterated on feedback to implement the `new OverridesOrder` conversion, added null-date cascading, tightened type guards, and shipped 6+ unit/integration tests verifying existing behavior preservation.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
