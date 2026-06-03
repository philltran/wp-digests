# #65395: Remove async product editor category field

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @gigitux
- **Labels:** `plugin: woocommerce`, `plugin: woocommerce beta tester`
- **Merged:** [`8b1bde0`](https://github.com/woocommerce/woocommerce/commit/8b1bde085be3d9156449c38b4e0c2eaf96369fee)
- **Discussion:** [#65395](https://github.com/woocommerce/woocommerce/pull/65395) · 4 comments · 0 reactions

## Summary

WooCommerce has removed the dormant `async-product-editor-category-field` feature flag and its associated product category metabox assets, as the functionality was never enabled in production. This cleanup eliminates unused React-based async components, reduces frontend bundle size, and formally deprecates the underlying AJAX callback to prevent future reliance on it. The change streamlines the product editor pipeline without altering core product categorization logic or database schemas.

## Impact

- **Plugin & theme developers**: No functional breakage to existing implementations. `WC_AJAX::json_search_categories_tree()` is now deprecated and will trigger a deprecation notice if invoked directly, though its underlying handler remains active for backward compatibility.
- **Beta testers & feature flag consumers**: The `async-product-editor-category-field` flag has been stripped from core/development configs and the `woocommerce-beta-tester` live branches script. Explicitly enabling this dormant feature is no longer possible.
- **Site owners & REST API consumers**: No action required. Product categorization continues to operate via standard product edit screens or backend mechanisms, unaffected by the frontend async component removal.

## Technical details

The unified diff removes `async-product-editor-category-field` from `plugins/woocommerce/client/admin/config/core.json` and `plugins/woocommerce/client/admin/config/development.json`. It deletes the entire `wp-admin-scripts/product-category-metabox/` directory, stripping React components (`category-metabox.tsx`, `all-category-list.tsx`, `category-add-new.tsx`) and their webpack entry point (`index.js`). In `includes/class-wc-ajax.php`, `WC_AJAX::json_search_categories_tree()` is wrapped with `wc_deprecated_function()`. The beta tester userscript at `woocommerce-beta-tester/userscripts/wc-live-branches.user.js` removes the flag from its hardcoded `featureFlags` array. PHPStan baseline suppressions in `phpstan-baseline.neon` were also cleared. No REST schema changes, database alterations, or core WordPress hooks were modified.

## Contribution

Opened by @gigitux to address issue #49120, the pull request progressed through automated size-reduction checks and a standard review cycle before merge. The design discussion centered on the fact that the async metabox was never shipped or enabled in production, making removal a straightforward debt-cleanup effort rather than a breaking migration. No significant alternative approaches were debated; the PR focused solely on stripping dormant assets and formally deprecating the associated AJAX endpoint.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
