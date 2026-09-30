# #68771: [Backport to release/11.2] Harden Back in Stock Notifications feature gate after alpha constant move

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @woocommercebot
- **Labels:** `plugin: woocommerce`, `metric: feature freeze exception`
- **Merged:** [`dd0b743`](https://github.com/woocommerce/woocommerce/commit/dd0b7432a5eb8e542e1dd47c5ad5861d043c0be2)
- **Discussion:** [#68771](https://github.com/woocommerce/woocommerce/pull/68771) · 2 comments · 0 reactions
- **Usefulness:** 2/5

## Summary

Backport to `release/11.2` of a hardening pass on the Back in Stock Notifications feature gate, after #64490 moved it from the `WOOCOMMERCE_BIS_ALPHA_ENABLED` constant to the `customer_stock_notifications` feature toggle. The `StockNotifications` class now honors the old constant until the 11.2.0 DB migration has run, `NotificationQuery` returns empty values instead of throwing while the feature is off, and two signatures that #64490 changed are restored before 11.2.0 GA: `SignupService::init()` and `StockNotifications::init_hooks()`.

## Impact

- **Sites that opted into the alpha via the constant:** the first 11.2 request after upgrade no longer risks an `Invalid data store.` exception before the option migration copies the constant into the option.
- **Plugin developers calling `NotificationQuery` (an `Internal` class):** calls made while the feature is disabled now return `[]`, `0`, `false` or `null` and log via `wc_caught_exception()`, where they previously threw.
- **Code calling `StockNotifications::init_hooks()`:** it works again but is `@deprecated 11.2.0` and emits a `wc_deprecated_function()` notice. `maybe_init_services()` is the replacement.
- **Code relying on removing `init_hooks()` from `plugins_loaded` to disable the feature:** this is now a silent no-op, since the constructor doesn't attach it. Use the feature toggle instead.
- **Anyone who coded against the unreleased 4-argument `SignupService::init()` or the instance-based `SignupRateLimiter`:** these existed only on trunk and `release/11.2`, so the signatures change back. Extra arguments are ignored by PHP, but instance calls to `is_rate_limited()` and `apply()` now resolve as static calls.
- No hooks, filters or options are added, removed or renamed. Other site owners need take no action.

## Technical details

**Constant bridge (`StockNotifications.php`).** Two private static helpers are added. `is_alpha_enabled()` returns true only if `Constants::is_true( 'WOOCOMMERCE_BIS_ALPHA_ENABLED' )` and `woocommerce_db_version` is below `11.2.0`. `is_enabled()` returns `FeaturesUtil::feature_is_enabled( self::FEATURE_NAME ) || self::is_alpha_enabled()`. `maybe_init_services()` and `on_install_or_update()` now call `is_enabled()`. `register_data_stores()` still reads `ENABLE_OPTION_NAME` directly (a data store can load before `init`) and additionally falls back to `is_alpha_enabled()`.

**Fail-soft queries (`NotificationQuery.php`).** A new private `load_data_store()` wraps `WC_Data_Store::load( 'stock_notification' )` in try/catch, logs with `wc_caught_exception( $e, __METHOD__ )` and returns `null`. `run_query()`, `product_has_active_notifications()`, `notification_exists_by_email()` and `notification_exists_by_user_id()` all go through it.

**Static rate limiter.** Every method in `SignupRateLimiter` becomes `static`, including the private helpers. `SignupService` drops its `$rate_limiter` property and constructor argument, and calls `SignupRateLimiter::is_rate_limited()` and `::apply()` statically. `init()` is back to three arguments.

**Deprecated shim.**
```php
public function init_hooks() {
    wc_deprecated_function( __METHOD__, '11.2.0', __CLASS__ . '::maybe_init_services()' );
    if ( did_action( 'init' ) ) { $this->maybe_init_services(); return; }
    add_action( 'init', array( $this, 'maybe_init_services' ), 1 );
}
```
It deliberately has no native return type, since adding one would fatal subclasses that override it. A changelog file (patch, fix) is included. Tests switch `SignupRateLimiterTests` to static calls, and per the description add coverage for the constant fallback either side of the 11.2.0 boundary, the shim, and `NotificationQuery` failing soft. Those test changes were truncated from the diff, so this is taken from the description.

## Contribution

This is an automated cherry-pick of trunk PR #68761 into `release/11.2`, carrying the `feature freeze exception` label. The hardening targets regressions that #64490 introduced before 11.2.0 GA. The description says the code and tests were drafted with Claude Code from a hand-written plan and then reviewed by the author. The only discussion in the record is bot-generated testing guidance and a Playground link.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
