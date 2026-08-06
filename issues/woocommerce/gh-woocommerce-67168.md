# #67168: Deprecate coupon updated props hook in favor of per-save successor

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @mordeth
- **Labels:** `plugin: woocommerce`, `developer advisory`
- **Merged:** [`3ed08f0`](https://github.com/woocommerce/woocommerce/commit/3ed08f0792c555bb52dd6f56b49ef0c0ae992363)
- **Discussion:** [#67168](https://github.com/woocommerce/woocommerce/pull/67168) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

WooCommerce deprecated the `woocommerce_coupon_object_updated_props` action in favor of a new `woocommerce_coupon_updated_props` hook that reports only the properties modified during the current save. The change resolves a backward-compatibility regression introduced in 11.0 when the original hook's payload was switched to per-save semantics. The legacy action now fires via `do_action_deprecated()` with its historical accumulated payload, while core tracking and modern extensions should use the replacement hook.

## Impact

- **Plugin & theme developers:** If you hook into `woocommerce_coupon_object_updated_props`, you will receive a deprecation notice under `WP_DEBUG` and must migrate to `woocommerce_coupon_updated_props` to receive per-save properties. Existing listeners continue to receive the exact historical payload (accumulated across saves with duplicates) to prevent breakage.
- **Hosting & platform teams:** No configuration changes required; the deprecation only triggers when legacy listeners are registered.
- **No action required** if you do not listen to this hook or directly read `WC_Coupon_Data_Store_CPT::$updated_props` (which is also deprecated).

## Technical details

The diff modifies `plugins/woocommerce/includes/data-stores/class-wc-coupon-data-store-cpt.php` to split the hook dispatch inside `update_post_meta()`:
- `$this->updated_props` now appends to itself on every meta update and is never cleared, feeding the legacy payload.
- The legacy `woocommerce_coupon_object_updated_props` fires via `do_action_deprecated()` with the accumulated list.
- A new `woocommerce_coupon_updated_props` action fires immediately after, passing the current save's `$updated_props` array.
- `plugins/woocommerce/includes/tracks/events/class-wc-coupon-tracking.php` switches its listener from the old hook to the new one.
- `WC_Coupon_Data_Store_CPT::$updated_props` is marked `@deprecated 11.1.0`.

Before/after dispatch pattern:
```php
// Before (11.0+):
do_action( 'woocommerce_coupon_object_updated_props', $coupon, $updated_props );
$this->updated_props = array();

// After (11.1+):
do_action_deprecated(
    'woocommerce_coupon_object_updated_props',
    array( $coupon, $this->updated_props ),
    '11.1.0',
    'woocommerce_coupon_updated_props'
);
do_action( 'woocommerce_coupon_updated_props', $coupon, $updated_props );
```
Tests in `class-wc-coupon-data-store-cpt-test.php` verify nested-save isolation, shared-store accumulation, and that the deprecation notice only fires when legacy listeners exist.

## Contribution

The change originated from a backward-compatibility regression in PR #66724, which switched the hook to per-save semantics and was flagged in #67147. After review, the author initially considered a full revert but pivoted to a deprecation strategy that preserves the historical accumulated payload for legacy listeners while introducing `woocommerce_coupon_updated_props` for modern use. Review feedback emphasized discoverability and notice hygiene, leading to the final approach where the deprecated action only fires when listeners are registered and the core Tracks listener was migrated to avoid triggering notices itself. The PR merged with 9 passing PHPUnit tests covering nested saves, shared store instances, and deprecation notices.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
