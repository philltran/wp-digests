# #65636: Add deprecated WC Admin feature flag shims

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @gigitux
- **Labels:** `plugin: woocommerce`, `focus: monorepo infrastructure`, `developer advisory`
- **Merged:** [`0a5622a`](https://github.com/woocommerce/woocommerce/commit/0a5622a282b7e64a9b491c16ba834b5a5fe7e93d)
- **Discussion:** [#65636](https://github.com/woocommerce/woocommerce/pull/65636) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

WooCommerce 11.1 introduces backward-compatibility shims for retired WC Admin feature flags that were removed from the active feature registry in a prior release. These shims intercept access via `Features::is_enabled()`, `Features::exists()`, `window.wcAdminFeatures`, and `isFeatureEnabled()`, returning the legacy `true` value while emitting deprecation warnings. This prevents third-party extensions that still reference retired flags from breaking immediately, while clearly signaling that these flags are deprecated and will be removed in a future version.

## Impact

- **Plugin & theme developers:** Extensions that read retired WC Admin feature flags (e.g., `launch-your-store`, `customize-store`, `analytics`) will continue to function but will trigger deprecation warnings in PHP and JS consoles. Developers should audit their code to remove references to these flags and migrate to the current feature registry or stable feature checks.
- **Hosting & platform teams:** No immediate action required. The shims are internal compatibility layers and do not affect server configuration or database schema.
- **Headless & REST consumers:** No direct impact; the changes are confined to the WooCommerce Admin JS/PHP feature flag utilities.

## Technical details

The diff introduces a synchronized PHP/JS deprecation layer for 32 retired feature flags. In PHP, `Automattic\WooCommerce\Admin\Features\Features` now maintains a `$retired_feature_compatibility_versions` array and routes calls to `is_enabled()` and `exists()` through these shims, returning `true` and emitting deprecation warnings. In JavaScript, `client/admin/client/utils/features/features.ts` wraps `window.wcAdminFeatures` in a `Proxy` that intercepts property access, checks against `isRetiredFeatureFlag()`, and calls `warnRetiredFeatureFlag()` (which delegates to `@wordpress/deprecated`). The TypeScript definitions in `client/admin/client/typings/global.d.ts` separate retired flags into a `DeprecatedWcAdminFeatureFlags` interface, keeping active flags like `'settings-ui'` and `'product-data-views'` in the main `window.wcAdminFeatures` type. A new `@wordpress/deprecated` dependency was added to `client/admin/package.json`. The retired flags are explicitly excluded from `wcSettings.admin.features` to keep the configurable registry focused on active toggles.

## Contribution

Opened and merged by @gigitux. The PR was developed as a targeted compatibility shim following the retirement of these flags in PR #65472. During review, the author adjusted the deprecation logic to attach metadata per feature flag rather than globally, noting that while a broader feature flag architecture revisit was considered, shipping this incremental shim was prioritized to stabilize third-party extension compatibility.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
