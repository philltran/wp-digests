# #64758: Move global admin notices into Site Health

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @mikejolley
- **Labels:** `needs: documentation`, `plugin: woocommerce`
- **Merged:** [`bbf90f2`](https://github.com/woocommerce/woocommerce/commit/bbf90f2b61a6f6ed4596a8ee6e3f61be74257f3c)
- **Discussion:** [#64758](https://github.com/woocommerce/woocommerce/pull/64758) · 12 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

WooCommerce has migrated a large set of global admin notices into the WordPress Site Health Status screen and moved background regeneration progress indicators to the Status > Tools table. This reduces dashboard noise by surfacing diagnostic checks (SSL, uploads protection, template staleness, MaxMind licensing, HPOS sync, base tables, and WooCommerce.com plugin risk) where they align with WordPress’s native severity badges and action-link patterns. Operational statuses like thumbnail and product lookup table regeneration now display inline with their respective tools and poll for live progress updates.

## Impact

- **Plugin & theme developers:** Custom code calling `WC_Admin_Notices::add_notice()` for the migrated notice IDs will no longer render global notices. The corresponding view templates (`html-notice-*.php`) have been deleted.
- **Site owners & administrators:** Global admin notices for status checks are removed. Checks now appear under **Tools > Site Health > Status**. Dismissal filters (`woocommerce_maxmind_geolocation_display_notices`, `woocommerce_hide_redirect_method_nag`, `woocommerce_hide_download_directories_sync_complete`, `woocommerce_hide_base_tables_missing_nag`) and per-user `dismissed_*` metas are no longer honored; Site Health items cannot be dismissed per-user.
- **Hosting & platform teams:** No configuration or migration steps required. The change applies automatically on plugin update.
- **Headless & REST consumers:** The `GET /wc/v3/system_status/tools` endpoint now returns `requires_refresh`, `disabled`, and `status_text` fields for tools in progress, though the current client polling implementation scrapes HTML rather than consuming this JSON payload directly.

## Technical details

- **Site Health registry:** `plugins/woocommerce/src/Internal/Admin/SiteHealth.php` introduces `handle_site_status_tests()` to register WooCommerce checks and `run_test()` to normalize callables into Site Health result payloads (label, status, badge, description, actions). Checks cover HTTPS, uploads-directory protection, template overrides, MaxMind license, download-method deprecation, DB table presence, HPOS sync-on-read, legacy shipping, missing shipping methods, approved download directories, and WooCommerce.com plugin risk.
- **Notice removal:** `plugins/woocommerce/includes/admin/class-wc-admin-notices.php` and `plugins/woocommerce/includes/integrations/maxmind-geolocation/class-wc-integration-maxmind-geolocation.php` now strip notice IDs from the queue instead of rendering them. View templates like `plugins/woocommerce/includes/admin/views/html-notice-secure-connection.php` and `html-notice-template-check.php` are deleted.
- **Tools polling:** `plugins/woocommerce/includes/rest-api/Controllers/Version2/class-wc-rest-system-status-tools-v2-controller.php` marks `regenerate_product_lookup_tables` and `regenerate_thumbnails` with `requires_refresh`. When jobs run, the controller disables the tool row and injects `status_text` HTML. Client-side, `plugins/woocommerce/client/legacy/js/admin/system-status.js` adds `maybePollTools()` and `pollTools()`, which interval-polls the tools page every 10s (localized via `woocommerce_admin_system_status.tools_poll_interval`) and replaces matching rows via `data-tool-action`.
- **Connect notice:** `plugins/woocommerce/client/admin/client/marketplace/components/connect-notice/connect-notice.tsx` now returns `null` when `noticeType === 'long'`, suppressing the outdated-plugin risk banner in favor of the Site Health check.
- **Before/After pattern:** Previously, `WC_Admin_Notices::add_notice( 'download_directories_sync_complete' )` rendered a global admin notice. Now, the same trigger persists an option, and `SiteHealth.php` registers a check that surfaces the status under Site Health with action links.

## Contribution

Opened and merged by @mikejolley, this PR consolidates accumulated WooCommerce admin notice noise into WordPress’s native diagnostic surface. During review, @ralucaStan flagged that several `woocommerce_hide_*` dismissal filters and per-user `dismissed_*` metas were dropped, noting that sites relying on those filters will now see unsuppressable Site Health items. The reviewer also suggested polling the `/wc/v3/system_status/tools` REST endpoint instead of scraping HTML, but the author noted the current HTML-scraping approach is acceptable for the initial release. The diff reflects a straightforward migration with parity checks confirming no action links were lost.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
