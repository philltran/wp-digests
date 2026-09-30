# #68761: Harden Back in Stock Notifications feature gate after alpha constant move

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @xristos3490
- **Labels:** `plugin: woocommerce`
- **Merged:** [`879eda5`](https://github.com/woocommerce/woocommerce/commit/879eda5bf21f114da624b15cfa61b885d606da87)
- **Discussion:** [#68761](https://github.com/woocommerce/woocommerce/pull/68761) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This WooCommerce PR hardens the Back in Stock Notifications feature gate after #64490 moved it from the `WOOCOMMERCE_BIS_ALPHA_ENABLED` constant to the `customer_stock_notifications` feature toggle. It honors the alpha constant until the 11.2.0 option migration has run, so alpha opt-in sites no longer risk a throw on the first 11.2 request. `NotificationQuery` now fails soft when the feature is off. It also reverts two signature changes that #64490 introduced and restores `init_hooks()` as a deprecated shim. It was targeted at 11.2.0 and cherry-picked to `release/11.2`.

## Impact

**Site owners**
- Sites that opted into the alpha via `WOOCOMMERCE_BIS_ALPHA_ENABLED` no longer risk an exception on the first 11.2 request, before `wc_update_1120_migrate_stock_notifications_alpha_constant()` copies the constant into the option.
- Fresh installs are unaffected by the constant. Once `woocommerce_db_version` reaches 11.2.0, the Features toggle is the only switch.

**Plugin & theme developers**
- Third-party calls into `NotificationQuery` no longer throw `Invalid data store.` while the feature is disabled. They now get `[]`, `0`, `false` or `null`, and the exception is logged through `wc_caught_exception()`.
- `StockNotifications::init_hooks()` is back as a deprecated shim (`@deprecated 11.2.0`). It calls `wc_deprecated_function()` and forwards to `maybe_init_services()`. Code that removed it from `plugins_loaded` to disable the feature is now a silent no-op, because the constructor does not attach it. Use the feature toggle instead.
- `SignupService::init()` is back to its 11.1.0 three-argument form. Callers passing a fourth argument keep working, since PHP ignores extra arguments.
- `SignupRateLimiter::is_rate_limited()` and `::apply()` are now static. The PR notes the instance form and the four-argument `init()` only existed on unreleased trunk and `release/11.2`, so this had to land before 11.2.0 GA.

**Other**
- No hooks, filters or options were added, removed or renamed.
- The PR notes that `wc_stock_notifications` and `wc_stock_notificationmeta` are still created on every install regardless of the toggle. This has been so since #64490 dropped the alpha guard from `get_database_schema()`, and it was left as-is.

## Technical details

**`StockNotifications.php`**
- New private static `is_alpha_enabled()` returns true only when `Constants::is_true( 'WOOCOMMERCE_BIS_ALPHA_ENABLED' )` and `version_compare( get_option( 'woocommerce_db_version', '0' ), '11.2.0', '<' )`.
- New private static `is_enabled()` returns `FeaturesUtil::feature_is_enabled( self::FEATURE_NAME ) || self::is_alpha_enabled()`. `maybe_init_services()` and `on_install_or_update()` now use it.
- `register_data_stores()` still reads `ENABLE_OPTION_NAME` directly, because a data store can load before `init` and building translated feature definitions that early is unsafe. It now bails only when the option is not `yes` and `is_alpha_enabled()` is false.
- `init_hooks()` is re-added with no native return type, so subclass overrides don't fatal. It calls `wc_deprecated_function( __METHOD__, '11.2.0', ... )`. If `did_action( 'init' )` is true it calls `maybe_init_services()` immediately. Otherwise it defers via `add_action( 'init', array( $this, 'maybe_init_services' ), 1 )`, so the check runs after the textdomain loads.

**`NotificationQuery.php`**
- New private static `load_data_store(): ?\WC_Data_Store` wraps `WC_Data_Store::load( 'stock_notification' )` in try/catch. On `\Exception` it calls `wc_caught_exception( $e, __METHOD__ )` and returns null.
- `run_query()` returns null when the data store is unavailable. `product_has_active_notifications()`, `notification_exists_by_email()` and `notification_exists_by_user_id()` return `false`.

**`SignupRateLimiter.php` / `SignupService.php`**
- `is_rate_limited()`, `apply()`, `get_rate_limits()`, `get_ip_address()`, `get_options()` and `to_bool()` became `static`, with `$this->` replaced by `self::`.
- `SignupService` drops the `$rate_limiter` property and constructor argument and calls `SignupRateLimiter::is_rate_limited()` / `::apply()` statically.

```php
// Before (trunk / 11.2 pre-fix)
$service->init( $eligibility, $management, $email_manager, $rate_limiter );
// After (matches 11.1.0)
$service->init( $eligibility, $management, $email_manager );
```

Tests: `SignupRateLimiterTests` switched to static calls. The PR description says new tests cover the constant fallback on both sides of the 11.2.0 DB version boundary, the `init_hooks()` shim, and `NotificationQuery` failing soft. The diff shown is truncated, so those test files were not reviewed here. A changelog entry (patch, fix) was added.

## Contribution

@xristos3490 opened the PR as a follow-up to #64490 and linked it to issue #68759. The description says the code and tests were drafted with Claude Code from a hand-written plan and then reviewed by the author. It was merged, and the bot reported a successful cherry-pick to `release/11.2` as backport PR #68771. CodeRabbit rated the merge risk low but flagged that the compatibility impact should be documented. The PR text notes manual browser testing had not yet been run when it was opened.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
