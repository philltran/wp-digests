# #68938: [Performance] Eliminate duplicate variation price aggregation on tax-enabled stores in read_price_data method of variable data store

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @kalessil
- **Labels:** `plugin: woocommerce`
- **Merged:** [`28b4574`](https://github.com/woocommerce/woocommerce/commit/28b4574c71b625d9defde5c97fb8fb9275361a07)
- **Discussion:** [#68938](https://github.com/woocommerce/woocommerce/pull/68938) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

`WC_Product_Variable_Data_Store_CPT::read_price_data()` no longer aggregates variation prices twice on tax-enabled stores. The aggregation now lives in a new private `prime_price_data_cache()`, which computes display (tax-adjusted) and raw price arrays in a single pass and stores both under separate hashes. `read_price_data()` becomes a thin lookup by hash. Benchmarks in the PR show cold-cache cost dropping roughly 1.3x-1.6x, depending on variation count.

## Impact

**Store owners / hosts**
- Tax-enabled stores (e.g. EU) with variable products get faster cold-cache price generation, mainly after product updates, checkout stock changes, or transient invalidation. No configuration needed.
- Warm-cache raw lookups move from about 0.001 ms to about 0.010 ms per call, per the PR benchmarks. This is negligible.

**Plugin & theme developers**
- `WC_Product_Variable_Data_Store_CPT::taxes_influence_price()` is deprecated since 11.3.0 and now calls `wc_deprecated_function()`. There is no replacement, since the optimization makes it obsolete.
- The filter `woocommerce_variable_product_taxes_influence_price` (added in 10.9.0) is deprecated via `apply_filters_deprecated()`. Callbacks still run but trigger deprecation notices. Remove them.
- `woocommerce_variation_prices_array` now fires twice per variation with `$for_display` set to `true` and then `false`, whenever `has_filter()` is true. Previously it fired for the requested mode plus an opposite-mode call only when taxes influenced the price. Callbacks that assume a single mode, or that are expensive or have side effects, should be reviewed.
- `woocommerce_variation_prices` is now applied to both the display and raw arrays on every cache prime, even when only one mode was requested.
- The output of `get_price_hash()` changes because `$for_display` is now folded into the hashed payload. Any code or fixture that depends on specific hash values, or that reads `wc_var_prices_{id}` transients directly, will see different keys. Existing transients are regenerated on demand.
- Subclasses of the data store that override `read_price_data()` or `get_price_hash()` should check the new flow.
- The `wc_doing_it_wrong` notice for malformed `woocommerce_variation_prices` output now reports `prime_price_data_cache` instead of `read_price_data` as the method. Tests asserting on the old name need updating.

**Headless / REST consumers**
- No API change. Output should be identical to before.

## Technical details

Changes in `includes/data-stores/class-wc-product-variable-data-store-cpt.php`:

- `read_price_data( &$product, $for_display )` now calls `prime_price_data_cache()`, which returns an object `{ hash_display, hash_raw }`. It then returns `$this->prices_array[ $hash ]` for the requested hash, or `array()` if the key is missing.
- A new private `$prices_hashes` property caches the `hash_display => hash_raw` mapping, so the second `get_price_hash()` call happens only once per display hash.
- The cache check is `empty( prices_array[hash_display] ) || empty( prices_array[hash_raw] )`. The transient check works the same way.
- The variation loop always reads `woocommerce_tax_display_shop`. It captures raw prices into `$raw_prices_array` **before** tax adjustment, then applies `wc_get_price_including_tax()` or `wc_get_price_excluding_tax()` into `$display_prices_array`. The redundant `'qty' => 1` argument was dropped.
- With `woocommerce_variation_prices_array`, each array is filtered separately:

```php
$display_prices_array = apply_filters( 'woocommerce_variation_prices_array', $original_display_prices_array, $variation, true );
$raw_prices_array     = apply_filters( 'woocommerce_variation_prices_array', $original_raw_prices_array, $variation, false );
```

- Both hashes are written to the transient, then each gets `woocommerce_variation_prices` applied with `true` / `false` and its own `validate_prices_data()` check.
- `get_price_hash()` now returns `md5( wp_json_encode( array( $price_hash, (bool) $for_display ) ) )`, scoping hashes by mode to reduce collision risk.
- The old `opposite_price_hash` logic and its `taxes_influence_price()` gating are removed.

In `class-wc-product-variable.php`, `get_price_html()` reorders a condition to `$min_reg_price === $max_reg_price && $this->is_on_sale()`. This short-circuits the `is_on_sale()` call, which is the second-aggregation entry point.

Tests: the VAT-exempt test now asserts that raw prices are not tax-adjusted. The malformed-filter test expects `prime_price_data_cache` as the incorrect-usage method. A changelog entry (`Significance: minor`, `Type: performance`) was added. The PR description's stated new test coverage also includes cache population and omission of empty-price variations.

## Contribution

Opened by @kalessil to close #66892 (WOOPLUG-7172), motivated by high-volume stores during peak sale events, and merged after automated review. The visible thread contains only bot comments (Playground link, CodeRabbit, testing guidelines) and no human design debate. CodeRabbit's merge-risk note flagged two concerns: extension callbacks now receive an extra unrequested mode, and custom data-store subclasses could return raw prices for display requests. The hash-scoping change (`(bool) $for_display` in the hashed payload) appears to address the second concern.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
