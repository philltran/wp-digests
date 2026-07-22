# #66880: Add check to background Stripe installer redirect

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @jorgeatorres
- **Labels:** `plugin: woocommerce`
- **Merged:** [`15820b5`](https://github.com/woocommerce/woocommerce/commit/15820b579ac9fa52830ad09f7124b8e664c91df9)
- **Discussion:** [#66880](https://github.com/woocommerce/woocommerce/pull/66880) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

WooCommerce hardens the legacy `?wc-install-plugin-redirect=woocommerce-gateway-stripe` admin redirect by requiring a valid nonce and capability check before proceeding. The change also deprecates `WC_Install::background_installer()` and `WC_Install::theme_background_installer()` in version 11.1.0, as these methods are no longer called internally but remain public for backward compatibility. This prevents unauthorized or forged redirect requests while formally signaling that the background installer pattern is retired.

## Impact

- **Plugin & theme developers**: Direct calls to `WC_Install::background_installer()` or `WC_Install::theme_background_installer()` will now trigger a `wc_deprecated_function()` notice in WooCommerce 11.1.0. Remove these calls or migrate to standard WordPress plugin installation routines.
- **Custom admin flows using the legacy redirect**: Requests to `?wc-install-plugin-redirect=woocommerce-gateway-stripe` without a valid `_wpnonce` will now fall back to the WooCommerce search page instead of triggering an install. Update any custom links to include a nonce generated via `wp_nonce_url()` with the action `wc-install-plugin-redirect_woocommerce-gateway-stripe`.
- **Site owners & hosting teams**: No action required. This is a backward-compatible security hardening and deprecation notice.

## Technical details

- In `plugins/woocommerce/includes/admin/class-wc-admin.php`, `WC_Admin::admin_redirects()` now sanitizes `$_GET['wc-install-plugin-redirect']` and `$_GET['_wpnonce']` with `sanitize_text_field()`, then validates the nonce via `wp_verify_nonce( $redirect_nonce, 'wc-install-plugin-redirect_' . $plugin_slug )`. The previous `// phpcs:disable WordPress.Security.NonceVerification.Recommended` suppression was removed.
- In `plugins/woocommerce/includes/class-wc-install.php`, `WC_Install::background_installer()` and `WC_Install::theme_background_installer()` now call `wc_deprecated_function()` with version `11.1.0`. The manual installation fallback link in `background_installer()` was updated to include a nonce.
- Before/after pattern for the redirect check:
  ```php
  // Before
  if ( current_user_can( 'install_plugins' ) && in_array( $plugin_slug, array( 'woocommerce-gateway-stripe' ), true ) ) {

  // After
  if ( wp_verify_nonce( $redirect_nonce, 'wc-install-plugin-redirect_' . $plugin_slug ) && current_user_can( 'install_plugins' ) && in_array( $plugin_slug, array( 'woocommerce-gateway-stripe' ), true ) ) {
  ```
- Unit tests in `plugins/woocommerce/tests/php/includes/admin/class-wc-admin-test.php` verify that invalid nonces, disallowed slugs, and valid nonces correctly trigger or block the install redirect.

## Contribution

The record carries no discussion detail beyond the author’s implementation notes and automated review checks.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
