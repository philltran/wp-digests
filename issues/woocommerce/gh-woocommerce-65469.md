# #65469: Fix order totals only filling half width on edit order screen (on mobile)

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @jorgeatorres
- **Labels:** `needs: documentation`, `plugin: woocommerce`
- **Merged:** [`e513b78`](https://github.com/woocommerce/woocommerce/commit/e513b786105f07307e143d83b3f193f0f26d74a6)
- **Discussion:** [#65469](https://github.com/woocommerce/woocommerce/pull/65469) · 4 comments · 2 reactions
- **Usefulness:** 3/5

## Summary

WooCommerce now renders the order totals and applied coupons sections as full-width stacked columns on the admin order edit screen when the viewport is 782px or narrower. Previously, these sections were constrained to 50% width side-by-side, causing cramped layouts and awkward text wrapping on mobile devices. This responsive CSS adjustment improves readability and usability for store managers editing orders on smaller screens.

## Impact

- **Store admins & agency staff:** No action required. The admin UI automatically adapts to mobile viewports, improving the editing experience on tablets and phones.
- **Plugin & theme developers:** No action required. This change is scoped to WooCommerce's legacy admin CSS and does not affect frontend storefronts, REST API, or block-based admin interfaces.
- **Hosting & platform teams:** No configuration or migration needed.

## Technical details

The change modifies `plugins/woocommerce/client/legacy/css/admin.scss` by injecting a `@media screen and (max-width: 782px)` block. It targets `.wc-used-coupons` and `.wc-order-totals`, resetting their `float` to `none` and setting `width: 100%`. It also adds a `margin-bottom: 12px` to `.wc-used-coupons` to prevent visual overlap when stacked. The diff shows a straightforward CSS layout adjustment with no PHP, JS, or database changes. A changelog entry (`plugins/woocommerce/changelog/63967-fix-order-totals-mobile-width`) documents the patch.

## Contribution

The PR was opened to resolve a reported mobile layout quirk, and reviewers validated the 782px breakpoint against WordPress's standard admin sidebar collapse threshold. The author confirmed the fix handles both coupon and non-coupon order states without side effects, leading to a straightforward merge with no alternative approaches debated.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
