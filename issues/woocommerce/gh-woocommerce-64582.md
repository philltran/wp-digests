# #64582: Fix `robots.txt` hardcoding `/wp-content/uploads/` path

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @retlehs
- **Labels:** `plugin: woocommerce`, `type: community contribution`
- **Merged:** [`f9dca5f`](https://github.com/woocommerce/woocommerce/commit/f9dca5f1a141a45166cd77501d33f4968e95b1f1)
- **Discussion:** [#64582](https://github.com/woocommerce/woocommerce/pull/64582) · 2 comments · 3 reactions
- **Usefulness:** 4/5

## Summary

WooCommerce's `robots_txt()` method previously generated `Disallow` rules for its internal directories using a hardcoded `/wp-content/uploads/` path combined with the WordPress install URL. This broke `robots.txt` output on sites that relocate `wp-content`, use the `UPLOADS` constant, set a custom `upload_path`, or run WordPress in a subdirectory. The fix derives the uploads base path directly from `wp_get_upload_dir()['baseurl']`, ensuring the generated `Disallow` directives always point to the actual uploads location.

## Impact

- **Plugin & theme developers / WooCommerce users:** No code changes required. The fix applies automatically on update.
- **Hosting & platform teams (Bedrock, custom directory structures):** `robots.txt` will now correctly block access to WooCommerce's internal directories (`wc-logs/`, `woocommerce_transient_files/`, `woocommerce_uploads/`) instead of pointing to non-existent paths.
- **Site owners:** No action required. The change is a transparent bug fix that improves SEO and security hygiene on non-standard WordPress layouts.

## Technical details

The change modifies `WooCommerce::robots_txt()` in `plugins/woocommerce/includes/class-woocommerce.php`. Previously, the method parsed `site_url()` to extract an install subdirectory prefix and appended `/wp-content/uploads/` literally. The diff replaces this with a call to `wp_get_upload_dir()`, parses its `baseurl` property, and extracts the path component. A fallback to `/wp-content/uploads` is retained for edge cases where `wp_parse_url()` returns an unexpected structure.

Before:
```php
$site_url = wp_parse_url( site_url() );
$path     = ( ! empty( $site_url['path'] ) ) ? $site_url['path'] : '';
// ...
$above[] = "Disallow: $path/wp-content/uploads/wc-logs/";
```

After:
```php
$upload_dir   = wp_get_upload_dir();
$upload_url   = wp_parse_url( $upload_dir['baseurl'] );
$uploads_path = ( is_array( $upload_url ) && ! empty( $upload_url['path'] ) ) ? rtrim( $upload_url['path'], '/' ) : '/wp-content/uploads';
// ...
$above[] = "Disallow: $uploads_path/wc-logs/";
```
The method still hooks into the `robots_txt` filter (via the `$output` parameter) and prepends the corrected `Disallow` lines before the default WordPress rules.

## Contribution

The record carries no substantive design debate or alternative approaches; the change was submitted as a straightforward community fix by @retlehs and merged after passing automated checks.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
