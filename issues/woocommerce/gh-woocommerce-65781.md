# #65781: Skip block registration on non-rendering requests

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @ralucaStan
- **Labels:** `plugin: woocommerce`, `needs: dev note`, `Performance`, `developer advisory`
- **Merged:** [`f615ae5`](https://github.com/woocommerce/woocommerce/commit/f615ae500bdb5d2f2d9e01c3446b33590d3a1e92)
- **Discussion:** [#65781](https://github.com/woocommerce/woocommerce/pull/65781) · 7 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

WooCommerce now skips block and pattern registration on requests that never render blocks, including the Store API, WooCommerce REST namespaces, cron, AJAX, XML-RPC, favicon, robots.txt, XML sitemaps, and WooCommerce admin pages. The change also fixes `is_store_api_request()` to correctly detect plain-permalink Store API calls via `?rest_route=`. A new `woocommerce_should_register_blocks` filter allows extensions to override the skip behavior if they need to render blocks in these contexts.

## Impact

- **Plugin & theme developers:** No immediate action required. If your extension renders WooCommerce blocks during a REST, cron, AJAX, or WooCommerce admin request, you must hook into `woocommerce_should_register_blocks` and return `true` to force registration.
- **Hosting & platform teams:** Expect ~13–18 ms latency reduction on Store API and WooCommerce REST requests due to skipped registration overhead.
- **Site owners:** No action required. Front-end, admin, and block editor experiences remain unchanged.

## Technical details

The diff introduces `BlockRegistrationContext` (`plugins/woocommerce/src/Blocks/Domain/BlockRegistrationContext.php`) with a `should_register(): bool` method that evaluates `$_SERVER['REQUEST_URI']`, `$_GET`, and early-loaded constants. It returns `false` for:
- Store API requests (pretty and plain permalinks)
- WooCommerce REST namespaces (`wc/v1`–`v4`, `wc/private`, `wc-admin`, `wc-analytics`, `wc-telemetry`)
- Cron, AJAX (`wp_doing_ajax()` or `$_GET['wc-ajax']`), XML-RPC
- Static paths (`/favicon.ico`, `/robots.txt`, `/wp-sitemap.xml`, `/wp-sitemap.xsl`)
- WooCommerce admin pages (`admin.php?page=wc-admin`, `wc-settings`, `wc-orders`, `wc-reports`, `wc-status`, `wc-addons`)

`Bootstrap::init()` now gates `BlockPatterns` and `BlockTypesController` instantiation behind `( new BlockRegistrationContext() )->should_register()`. The `woocommerce_should_register_blocks` filter runs inside `should_register()` and passes the computed boolean, allowing overrides. The filter fires on `plugins_loaded`, so it only has access to `$_SERVER`, `$_GET`, and constants. `is_store_api_request()` in `class-woocommerce.php` was updated to parse `$_GET['rest_route']` and validate the path prefix, fixing plain-permalink detection.

## Contribution

Opened and merged by @ralucaStan, with performance benchmarking and testing guidance from @kalessil and @jorgeatorres. The initial scope covered REST and cron; during review, @kalessil requested admin page coverage. After discussing whitelist vs. blacklist trade-offs, the team settled on a blacklist targeting WooCommerce-owned admin pages while preserving core admin and block editor registration. The change shipped as an interim optimization ahead of a larger registration refactor.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
