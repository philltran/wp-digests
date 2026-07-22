# #66321: Avoid repeated payment provider checks in admin requests

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @dmallory42
- **Labels:** `plugin: woocommerce`
- **Merged:** [`4147321`](https://github.com/woocommerce/woocommerce/commit/4147321daf816fed021fdd598a4a8915213b2b3b)
- **Discussion:** [#66321](https://github.com/woocommerce/woocommerce/pull/66321) · 13 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

WooCommerce now caches payment provider lists and gateway details at the request level to eliminate redundant account state checks and extension matching on every admin page load. Previously, classes like `Payments::get_payment_providers()` and `PaymentsProviders::get_payment_gateway_details()` recomputed expensive gateway enumeration, account checks, and suggestion matching on each call, causing significant TTFB delays on stores with incomplete setup task lists. The change introduces request-scoped memoization via `wp_cache_add_non_persistent_groups`, drastically reducing database and external API calls while preserving per-user state and ordering behavior.

## Impact

- **Plugin & theme developers / WooCommerce admins:** No breaking changes or API removals. The only public-facing change is an additive `Payments::clear_cache()` method (and a deprecated `reset_memo()` in `PaymentsProviders`).
- **Stores with incomplete setup task lists & active payment gateways (e.g., WooPayments):** Significant reduction in dashboard TTFB and backend account cache reads. No configuration or migration required.
- **Multi-site / shared hosting:** Caching is strictly request-scoped and non-persistent, avoiding cross-request staleness for per-user preferences or live gateway states.

## Technical details

The diff introduces two non-persistent cache groups registered via `wp_cache_add_non_persistent_groups`: `woocommerce_payments_providers` in `plugins/woocommerce/src/Internal/Admin/Settings/Payments.php` and `woocommerce_payment_gateway_details` in `plugins/woocommerce/src/Internal/Admin/Settings/PaymentsProviders.php`.

- `Payments::get_payment_providers()` now checks a composite cache key (`user_id__can_install_plugins__location__for_display__remove_shells`) before deriving the provider list. Mutators like `update_payment_providers_order_map()`, `attach_payment_extension_suggestion()`, `hide_payment_extension_suggestion()`, and `dismiss_extension_suggestion_incentive()` now call `$this->clear_cache()` to invalidate the group.
- `PaymentsProviders::get_payment_gateway_details()` caches results per `user_id__gateway_id__country_code`. The original in-memory memo arrays (`$payment_gateways_memo`, `$payment_gateways_for_display_memo`) were renamed to `$payment_gateways_cache` and `$payment_gateways_for_display_cache` and now also use `wp_cache_get/set`.
- `PaymentsProviders::reset_memo()` is deprecated in favor of `clear_cache()`, which clears both the in-memory arrays and the persistent cache groups.
- The `_order` parameter is applied on every call, even on cache hits, preserving existing ordering semantics.

Cache invalidation pattern changed from internal-only resets to explicit public calls:
```php
// Before (internal only, no public API)
$this->providers->reset_memo();

// After (public additive method, deprecated alias remains)
$this->payments->clear_cache();
```

## Contribution

Opened by @dmallory42 and merged in commit `4147321`, the PR addresses performance regression #66317. Reviewers @kalessil and @oaratovskyi discussed whether to implement a cross-request cache via the object cache. @dmallory42 argued against it due to staleness risks with per-user hidden suggestions, incentive dismissals, and live gateway account states, noting that a ten-minute cache could leak another admin’s preferences. The team agreed to keep the cache strictly request-scoped, and the PR was slated for WooCommerce 11.1.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
