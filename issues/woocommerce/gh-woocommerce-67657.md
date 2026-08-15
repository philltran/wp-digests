# #67657: Deprecate product brand recount methods

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @gigitux
- **Labels:** `plugin: woocommerce`, `developer advisory`
- **Merged:** [`60ada59`](https://github.com/woocommerce/woocommerce/commit/60ada59df147aef458f2579fdaa5d86e0166e9a2)
- **Discussion:** [#67657](https://github.com/woocommerce/woocommerce/pull/67657) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Two public methods on `WC_Brands` are now deprecated as of WooCommerce 11.2.0: `recount_after_stock_change()` is superseded by the global `wc_recount_after_stock_change()`, and `recount_all_brands()` by `wc_recount_all_terms()`. Both methods keep their signatures and remain callable, but they now emit a `wc_deprecated_function()` notice and delegate straight to the shared term-recount functions instead of carrying their own `product_brand`-specific implementation. This is the cleanup half of PR #67630, which folded product-brand counting into WooCommerce's shared term recount path and left these two methods with no internal callers. Carries the `developer advisory` label and ships a `minor`/`update` changelog entry.

## Impact

**Plugin & extension developers**
- Any code calling `WC_Brands::recount_after_stock_change( $product_id )` or `WC_Brands::recount_all_brands()` will start logging deprecation notices in 11.2.0. Replace with the global functions `wc_recount_after_stock_change()` and `wc_recount_all_terms()` respectively.
- Not a breaking change today — both methods remain public, keep their signatures, and still perform a recount by delegation.
- Behavior nuance worth testing: `recount_all_brands()` previously recounted only `product_brand` terms (via `get_terms()` scoped to that taxonomy); it now calls `wc_recount_all_terms()`, the centralized full-term recount, with no taxonomy argument.

**Store owners / hosting & platform**
- No action required. No UI, install-layout, or multisite behavior changed; the out-of-stock recount path is unchanged in effect, just routed through the shared implementation.

**Headless / REST consumers**
- Unaffected — no REST schema, option, or database changes.

## Technical details

All functional changes are in `plugins/woocommerce/includes/class-wc-brands.php`, where both method bodies are replaced by a deprecation notice plus a delegating call:

```php
public function recount_after_stock_change( $product_id ) {
	wc_deprecated_function( __METHOD__, '11.2.0', 'wc_recount_after_stock_change()' );
	wc_recount_after_stock_change( $product_id );
}

public function recount_all_brands() {
	wc_deprecated_function( __METHOD__, '11.2.0', 'wc_recount_all_terms()' );
	wc_recount_all_terms();
}
```

The deleted implementation of `recount_after_stock_change()` was the brand-specific one: bail unless `get_option( 'woocommerce_hide_out_of_stock_items' )` is `yes`, fetch `get_the_terms( $product_id, 'product_brand' )`, and — when `wp_defer_term_counting()` is active — hand the `term_taxonomy_id` column to `wp_update_term_count()`; otherwise build a `term_id => parent` map and call `_wc_term_recount( $product_brands, get_taxonomy( 'product_brand' ), false, false )`. `recount_all_brands()` similarly did a `get_terms()` query with `'fields' => 'id=>parent'` on `product_brand` followed by `_wc_term_recount( …, true, false )`. Both `@deprecated 11.2.0` docblock tags name the replacement.

Caller-side migration:

```php
// Before
$brands = new WC_Brands();
$brands->recount_after_stock_change( $product_id );
$brands->recount_all_brands();

// After
wc_recount_after_stock_change( $product_id );
wc_recount_all_terms();
```

Because the direct `_wc_term_recount()` and `foreach` calls are gone, three suppressions for `includes/class-wc-brands.php` drop out of `plugins/woocommerce/phpstan-baseline.neon`: one `foreach.nonIterable` on `array<WP_Term>|WP_Error`, and two `argument.type` entries for `_wc_term_recount()` (`$terms` receiving `array<int,int>|WP_Error`, and `$taxonomy` receiving `WP_Taxonomy|false`). A changelog file, `plugins/woocommerce/changelog/deprecate-brand-recount-methods`, records `Significance: minor` / `Type: update`. No hooks, filters, options, or schema were touched.

## Contribution

A mechanical follow-up to PR #67630, which had already moved brand counting into the shared recount functions. The record carries no human review discussion — the only comments are bots: the reviewer-ping template addressed to @dinhtungdu and @dilirity, a CodeRabbit pass that generated no actionable comments, and a Playground preview link. The author disclosed that OpenAI Codex was used to implement the deprecations, run validation, and draft the PR description; validation cited was the 14-test `WC_Brands_Test` suite on PHP 8.1.34, PHP lint, and a clean PHPStan run on the changed file.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
