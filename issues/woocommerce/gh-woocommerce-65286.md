# #65286: Fix: Shop Manager role cannot reorder Local Pickup locations (#59894)

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @mikejolley
- **Labels:** `needs: documentation`, `plugin: woocommerce`, `focus: e2e tests`
- **Merged:** [`0fe325b`](https://github.com/woocommerce/woocommerce/commit/0fe325b80febfa4be2873848f8a5f74693631885)
- **Discussion:** [#65286](https://github.com/woocommerce/woocommerce/pull/65286) · 8 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

WooCommerce now exposes a dedicated `POST /wc/v3/pickup-locations` REST endpoint for saving Local Pickup settings, resolving a `403 Forbidden` error that Shop Managers encountered when reordering or editing locations. Previously, the blocks frontend posted directly to WordPress core’s `/wp/v2/settings` route, which requires the `manage_options` capability that Shop Managers lack. The new endpoint uses WooCommerce’s standard `manage_woocommerce` permission check and includes explicit sanitization for nested location payloads.

## Impact

- **Shop Managers & Store Staff:** Can now successfully save Local Pickup settings (reordering, editing locations, changing costs/titles) without permission errors.
- **Plugin & Theme Developers:** The internal tracking mechanism for Local Pickup saves has moved. `ShippingController::track_local_pickup()` is deprecated in favor of tracking inside the new controller. Custom code intercepting `rest_pre_serve_request` for this route will no longer trigger.
- **Hosting & Platform Teams:** No action required. The legacy `/wp/v2/settings` route remains registered for administrators and external tools that read it.
- **Frontend/JS Developers:** The blocks settings context now targets `/wc/v3/pickup-locations`. Custom integrations that mock or intercept the old route may need updating.

## Technical details

- Introduces `PickupLocationsRestController` (`plugins/woocommerce/src/Blocks/Shipping/PickupLocationsRestController.php`) extending `WP_REST_Controller`, registering `POST /wc/v3/pickup-locations` with `wc_rest_check_manager_permissions( 'settings', 'edit' )`.
- The controller manually sanitizes nested `pickup_location_settings` and `pickup_locations` arrays before calling `update_option()` for `woocommerce_pickup_location_settings` and `pickup_location_pickup_locations`, since the WP REST dispatcher only sanitizes top-level arguments.
- Updates `ShippingController::register_settings()` to instantiate and register the new controller, and removes the `rest_pre_serve_request` hook that previously intercepted `/wp/v2/settings` for tracking.
- The old `track_local_pickup()` method in `ShippingController` is marked `@deprecated 11.0.0` and now just calls `wc_deprecated_function()`.
- Frontend migration in `settings-context.tsx`:
  ```diff
  -			path: '/wp/v2/settings',
  +			path: '/wc/v3/pickup-locations',
  ```
- PHPUnit and E2E tests verify permission boundaries (`shop_manager` allowed, `editor` denied) and sanitization (e.g., `wp_kses_post` for `details`, `sanitize_text_field` for address/name, formula preservation for `cost`).

## Contribution

Opened and merged by @mikejolley. During review, @senadir suggested leveraging the existing WooCommerce v3 settings API to avoid a niche endpoint, but @mikejolley clarified that the v3 settings API only supports scalar key-value pairs and cannot handle the nested, repeatable object arrays required for pickup locations. The team proceeded with the dedicated controller approach, which aligns with how other complex shipping structures are handled. Tracking logic was refactored from a global `rest_pre_serve_request` callback into the controller itself to ensure it fires reliably on the new route.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
