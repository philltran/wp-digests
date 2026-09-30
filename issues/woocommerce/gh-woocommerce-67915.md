# #67915: [Backport to release/11.1] Enable variation galleries for all stores

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @woocommercebot
- **Labels:** `needs: documentation`, `plugin: woocommerce`, `metric: feature freeze exception`, `developer advisory`, `feature highlight`
- **Merged:** [`1fbf7fb`](https://github.com/woocommerce/woocommerce/commit/1fbf7fb769420beb8e0ff51b289dd7e8f3f15201)
- **Discussion:** [#67915](https://github.com/woocommerce/woocommerce/pull/67915) · 2 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

WooCommerce 11.1 enables the native variation gallery (multiple images per product variation) for every store, moving it from a 5% canary cohort to 100%. The experimental feature toggle is removed from WooCommerce > Settings > Advanced > Features, and the legacy `wc_feature_woocommerce_additional_variation_images_enabled` option is deleted by a DB update. This PR is the cherry-pick of trunk PR #67884 onto `release/11.1`.

## Impact

**Site owners**
- Variation galleries are always on; there is no longer a "Variation gallery" row in the Features settings.
- The Additional Variation Images (AVI) plugin is deactivated on update (including network-activated instances on multisite) and its data is migrated.

**Plugin & theme developers**
- `get_option( 'wc_feature_woocommerce_additional_variation_images_enabled' )` no longer reflects feature state, because the option is deleted on upgrade. Use `FeaturesUtil::feature_is_enabled( 'variation_gallery' )`, which returns `true`.
- `Automattic\WooCommerce\Internal\VariationGallery\Package::is_in_canary_cohort()` is deprecated (11.1.0) and now calls `wc_deprecated_function()`, so callers will log a deprecation notice. `Package::CANARY_MAX_VARIANT` is deprecated. `Package::is_enabled()` is kept and always returns `true`.
- The `available_variation` data (`gallery_image_ids`, `gallery_images_html`) and the Product Gallery block variation data now always include variation gallery images.
- Theme authors overriding `templates/single-product/add-to-cart/variable.php` should sync their copy: the template version is bumped to 11.1.0 and the `wc-product-gallery-default-template` script tag is now output unconditionally.

**Hosting & platform**
- A new DB update `11.1.0-1` runs on upgrade.
- Tracker telemetry drops `feature_option_explicit`, `remote_variant_assignment`, and `remote_variant_cohort`; `feature_enabled` is hardcoded to `'yes'`.

## Technical details

- **Feature definition** (`FeaturesController::init_feature_definitions`): the `variation_gallery` entry now has `is_experimental => false`, `enabled_by_default => true`, `disable_ui => true`, `deprecated_since => '11.1.0'`, and `deprecated_value => true`. The `option_key` and canary-based default are removed.
- **`Package`**: `is_enabled()` returns `true`. `is_in_canary_cohort()` is deprecated and proxies to `is_enabled()`. The private `REMOTE_VARIANT_OPTION_NAME` constant is removed. `init()` no longer early-returns, so `ClassicVariationGalleryAdmin` and `LegacyVariationGalleryCompatibility` always register. The class is deliberately retained, as the author notes, because WooCommerce historically treats such classes as public interface.
- **DB update**: `WC_Install::$db_updates['11.1.0-1']` runs the new `wc_update_11101_remove_deprecated_variation_gallery_option()`, which calls `delete_option( VariationGalleryPackage::ENABLE_OPTION_NAME )`.
- **Gate removal**: `Package::is_enabled()` checks are removed from `WC_Product_Variable::get_available_variation()`, `ProductGalleryUtils::build_variation_gallery_entry()`, and `woocommerce_variable_add_to_cart()` (reset snapshot via `wp_add_inline_script`).
- **`Packages::deactivate_merged_packages()`**: now casts `active_plugins` to an array and, on multisite, merges in the keys of `active_sitewide_plugins` so network-activated AVI is also deactivated.
- **Tests**: variation-gallery flag tests are removed or simplified. The legacy test bootstrap cancels the scheduled `VariationGalleryMigration::run` action so it does not leak into the shared `woocommerce-db-updates` queue. The diff is truncated, so later test changes are not visible.

```php
// Before
get_option( 'wc_feature_woocommerce_additional_variation_images_enabled' );
// After
\Automattic\WooCommerce\Utilities\FeaturesUtil::feature_is_enabled( 'variation_gallery' ); // true
```

## Contribution

This is an automated cherry-pick of trunk PR #67884 into `release/11.1`, labeled as a feature freeze exception. The bot flagged a conflict in `tests/php/includes/wc-product-functions-test.php`, which the description marks as resolved. The original work followed the earlier Brands rollout precedent, and the author noted Codex assisted with implementation review and verification. PHPUnit could not be run locally because of low disk space for the Docker MySQL container; PHP syntax, PHPCS, lint, and PHPStan passed.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
