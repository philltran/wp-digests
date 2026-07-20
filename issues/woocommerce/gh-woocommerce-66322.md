# #66322: Remove Sample badge from products generated during the CYS flow

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @Aljullu
- **Labels:** `needs: documentation`, `plugin: woocommerce`
- **Merged:** [`4c88743`](https://github.com/woocommerce/woocommerce/commit/4c88743167e40340cb19b76bb683f51f8f1850b3)
- **Discussion:** [#66322](https://github.com/woocommerce/woocommerce/pull/66322) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

WooCommerce 11.1.0 removes the legacy "Sample" badge and associated `_headstart_post` postmeta logic used during the Customize Your Store (CYS) flow. The change cleans up deprecated admin rendering, deprecates the `OnboardingProducts` REST controller and `UpdateProducts` class, and introduces a database migration to purge leftover `_headstart_post` meta from existing stores.

## Impact

- **Plugin & theme developers:** The `add_sample_product_badge` method in `WC_Admin_List_Table_Products` is deprecated. The `Automattic\WooCommerce\Admin\API\OnboardingProducts` REST controller and its `create_products()` endpoint are deprecated. The `Automattic\WooCommerce\Blocks\AIContent\UpdateProducts` class is deprecated.
- **Store owners / admins:** Products previously created via the CYS flow will no longer display the "Sample" badge in the admin product list. A background database migration will automatically remove the `_headstart_post` meta key.
- **Hosting & platform:** No configuration changes required. The migration runs automatically on plugin update.
- **Action required:** Update any custom code relying on `_headstart_post` meta or the deprecated REST/controller classes. No immediate action needed for standard stores.

## Technical details

- `includes/admin/list-tables/class-wc-admin-list-table-products.php`: Removes the `manage_product_posts_custom_column` hook for `add_sample_product_badge` and replaces the method body with `wc_deprecated_function( __METHOD__, '11.1.0' )`.
- `includes/wc-update-functions.php`: Registers an 11.1.0 DB update entry and adds `wc_update_1110_remove_headstart_post_product_meta()` to delete `_headstart_post` postmeta for products, with logging.
- `src/Admin/API/OnboardingProducts.php`: Marks the class and `create_products()` method as `@deprecated 11.1.0`. The endpoint now returns `success: false` and triggers `wc_deprecated_function()`.
- `src/Admin/Features/OnboardingTasks/Tasks/Products.php`: Replaces the complex `$wpdb` query and `is_valid_product()` validator with a simplified check using `wp_count_posts( 'product' )` and `ProductStatus::PUBLISH`.
- `src/Blocks/AIContent/UpdateProducts.php`: Stripped of its dummy product generation logic and marked `@deprecated`.
- `phpstan-baseline.neon`: Updated to reflect removed/deprecated code paths.

## Contribution

Opened and merged by @Aljullu, this PR evolved from a targeted badge removal into a broader cleanup of the CYS flow. During review, the author expanded the scope to fully remove the `_headstart_post` meta logic, deprecate the associated REST controller and `UpdateProducts` class, and simplify onboarding product checks. The diff reflects the final merged state, with supporting PHPStan baseline updates and a changelog entry included.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
