# #67884: Enable variation galleries for all stores

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @kmanijak
- **Labels:** `needs: documentation`, `plugin: woocommerce`, `developer advisory`, `feature highlight`
- **Merged:** [`bf74658`](https://github.com/woocommerce/woocommerce/commit/bf74658ba5ce3806b0694d1bca4414d1ad69c62f)
- **Discussion:** [#67884](https://github.com/woocommerce/woocommerce/pull/67884) · 7 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

WooCommerce 11.1 rolls the native variation gallery (multiple images per variation) out to 100% of stores, up from a 5% canary cohort. The experimental "Variation gallery" toggle under WooCommerce > Settings > Advanced > Features is removed, `Package::is_enabled()` now always returns `true`, and the option-gated code paths in the classic and block gallery renderers are made unconditional. A DB update deletes the legacy feature option, and the Additional Variation Images (AVI) plugin is deactivated with its data migrated.

## Impact

**Site owners**
- The Variation gallery row disappears from the Features settings screen, and the feature can no longer be turned off. Stores that had explicitly set the option to `no` are enabled as well.
- If the Additional Variation Images plugin is active, it is deactivated and its data is migrated, per the PR's test steps.

**Plugin & theme developers**
- **Option checks break.** `wc_feature_woocommerce_additional_variation_images_enabled` is deleted by update `11.1.0-1`, so `get_option()` checks no longer reflect the feature state. Use `FeaturesUtil::feature_is_enabled( 'variation_gallery' )`, which continues to return `true`.
- **Deprecated symbols.** `Package::is_in_canary_cohort()` now calls `wc_deprecated_function()` (replacement: `Package::is_enabled()`) and returns `true`. `Package::CANARY_MAX_VARIANT` is marked `@deprecated 11.1.0`. Both are kept for compatibility, so existing callers keep working but will log deprecation notices.
- **Template change.** `templates/single-product/add-to-cart/variable.php` is bumped to `@version 11.1.0`. It now always outputs the `wc-product-gallery-default-template` script tag. Themes overriding this template should sync.
- Output of `WC_Product_Variable::get_available_variation()` now always includes variation gallery image IDs and gallery HTML, and `ProductGalleryUtils` variation gallery data always includes variation gallery images. Code that assumed the single-image shape with the flag off will see different data.

**Hosting & platform / telemetry**
- The tracker snapshot drops `feature_option_explicit`, `remote_variant_assignment` and `remote_variant_cohort`. `feature_enabled` is now hard-coded to `'yes'`.
- On multisite, merged-package deactivation now also considers network-activated plugins.

## Technical details

**Feature registration** (`FeaturesController::init_feature_definitions()`): the `VariationGallery\Package::FEATURE_ID` entry drops `option_key` and the canary-based `enabled_by_default`. It now sets `is_experimental => false`, `enabled_by_default => true`, `disable_ui => true`, `deprecated_since => '11.1.0'` and `deprecated_value => true`. The description is shortened to "Add multiple images per product variation."

**`Package.php`**
- `is_enabled()` now returns `true` unconditionally.
- `is_in_canary_cohort()` is deprecated and delegates to `is_enabled()`.
- The private `REMOTE_VARIANT_OPTION_NAME` constant is removed.
- `init()` no longer early-returns when disabled, so `ClassicVariationGalleryAdmin` and `LegacyVariationGalleryCompatibility` always register.
- `ENABLE_OPTION_NAME` remains as a public constant.

**Unconditional gallery paths** — the `Package::is_enabled()` guards are removed from:
- `WC_Product_Variable::get_available_variation()`
- `ProductGalleryUtils::build_variation_gallery_entry()`
- `woocommerce_variable_add_to_cart()` in `wc-template-functions.php`, where the reset snapshot is now attached once per product
- the `variable.php` template

**DB update:** `class-wc-install.php` registers `'11.1.0-1' => array( 'wc_update_11101_remove_deprecated_variation_gallery_option' )`. The function in `wc-update-functions.php` calls `delete_option( VariationGalleryPackage::ENABLE_OPTION_NAME )`. Note that the CodeRabbit walkthrough describes this update as converting `no` to `yes`, but the diff shows a plain `delete_option()`, so the diff is the source of truth here.

**Multisite:** `Packages::deactivate_merged_packages()` now casts `active_plugins` to an array and, when `is_multisite()`, merges in `array_keys( get_site_option( 'active_sitewide_plugins' ) )` with `array_unique()` before iterating.

**Tests:** the feature-off test cases and option toggling are removed. `tests/legacy/bootstrap.php` adds `cancel_variation_gallery_migration_action()` on `init` (priority 21), which cancels the queued `Migration::run` action in the `woocommerce-db-updates` group so it doesn't leak into unrelated tests.

```php
// Before: option could be 'yes'/'no'/unset (canary fallback)
get_option( 'wc_feature_woocommerce_additional_variation_images_enabled' );

// After: option is deleted; use the Features API
\Automattic\WooCommerce\Utilities\FeaturesUtil::feature_is_enabled( 'variation_gallery' ); // true
```

## Contribution

The PR was authored by @kmanijak with Codex assisting on implementation review and PR preparation, and it follows the earlier Brands rollout precedent for a canary-to-100% move. In review, @Aljullu could not reproduce the test steps for the explicit-`no` case. The cause was leftover DB state blocking the migration (`woocommerce_db_version` and `wc_variation_gallery_migration_completed_at`), so the testing steps were updated with reset commands. Redundant checks were removed at the reviewer's request. The author kept the deprecated `Package` symbols rather than deleting them, noting that WooCommerce historically treats such code as public interface even though the class was only added a few versions ago. The Highlight Template Changes CI job failed because it compares against 11.2.0 dev while this targets 11.1.0. A backport PR to `release/11.1` was generated automatically.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
