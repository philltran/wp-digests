# #65472: Load stable WC Admin features directly

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @gigitux
- **Labels:** `needs: documentation`, `Documentation`, `plugin: woocommerce`, `focus: monorepo infrastructure`, `plugin: woocommerce beta tester`, `focus: e2e tests`, `developer advisory`
- **Merged:** [`f87fa99`](https://github.com/woocommerce/woocommerce/commit/f87fa99f4b47d242d67c5edfd25563e67983824e)
- **Discussion:** [#65472](https://github.com/woocommerce/woocommerce/pull/65472) · 8 comments · 0 reactions
- **Usefulness:** 5/5

## Summary

WooCommerce 11.1 retires a large set of stable WooCommerce Admin feature flags from the legacy configuration pipeline (`core.json`, `development.json`, and `window.wcAdminFeatures`). Stable features are now loaded directly by the `Features` class, bypassing the old flag-based gating system. This change simplifies the admin feature architecture and removes experimental toggles for core functionality while providing deprecated compatibility shims to prevent immediate breakage for custom code.

## Impact

- **Plugin & extension developers:** Deprecated legacy helper methods (`Features::is_enabled()`, `Features::exists()`, `Features::get_optional_feature_options()`, `Features::enable()`, `Features::disable()`) and direct `window.wcAdminFeatures` checks for retired stable flags will emit deprecation warnings. Code relying on these flags to gate UI, routes, or tasks must migrate to the underlying options or remove the conditionals entirely.
- **Custom code authors:** The `woocommerce_admin_features` filter should no longer be used to disable retired stable features; filtering them out will not suppress the underlying behavior.
- **Site owners & platform teams:** No immediate action required. Functionality remains available, but custom scripts or plugins that previously toggled these flags may surface deprecation warnings in debug logs.

## Technical details

- The `Features` class (`plugins/woocommerce/src/Admin/Features/Features.php`) now instantiates explicit feature classes in `load_features()` instead of parsing legacy JSON configs or checking `window.wcAdminFeatures`.
- Legacy checks like `Features::is_enabled( 'analytics' )` are routed through deprecated compatibility shims that return legacy values but trigger warnings. The canonical replacement is `FeaturesUtil::feature_is_enabled( 'analytics' )`.
- Remote inbox notifications are now gated by the `woocommerce_show_marketplace_suggestions` option instead of a feature flag.
- Client-side code replaces `window.wcAdminFeatures` reads with the `isFeatureEnabled()` utility (e.g., `plugins/woocommerce/client/admin/client/activity-panel/display-options/index.js`).
- TypeScript typings in `plugins/woocommerce/client/admin/client/typings/global.d.ts` are updated to reflect the new `wcAdminFeatures` shape, removing deprecated slugs.
- Before/After example:
  ```php
  // Before
  if ( Features::is_enabled( 'launch-your-store' ) ) { ... }
  // After
  // Feature is always loaded; remove the conditional or check the underlying option if configurable.
  ```
- The diff removes the `set-legacy-minicart-flag.js` bin script, trims feature flags from `core.json` and `development.json`, and adds changelog files documenting the deflagging and compatibility shims.

## Contribution

Opened and merged by @gigitux as the parent of a 10-PR stacked series to systematically deflag WooCommerce Admin features. The PR establishes the new loading architecture and compatibility shim layer, while subsequent PRs in the stack remove legacy gates for specific feature areas (analytics, onboarding, mini-cart, etc.). The discussion highlights the need for a backward-compatible migration path, resulting in the inclusion of deprecated shims that emit warnings rather than breaking existing checks immediately.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
