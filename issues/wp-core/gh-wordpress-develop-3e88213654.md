# REST API: expose privacy policy page in settings endpoint.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** André Maneiro
- **Committed:** 2026-09-08
- **Commit:** [`3e88213654`](https://github.com/WordPress/wordpress-develop/commit/3e882136546b8cc053d97dc7db232fdf67fd05a3)
- **Usefulness:** 4/5

## Summary

The `wp_page_for_privacy_policy` option is now registered as a setting and exposed in the core `/wp/v2/settings` REST endpoint as `page_for_privacy_policy` (integer page ID). A new `rest_pre_update_setting` callback blocks updates from users who lack `manage_privacy_options`, so the REST endpoint follows the same permission rule as the Settings > Privacy screen, including on multisite.

## Impact

**Headless & REST consumers / block editor and app developers**
- `GET /wp/v2/settings` now returns `page_for_privacy_policy` (0 when unset). `PUT`/`POST` accepts it as an integer.
- Clients can read and set the privacy policy page without a custom route or a custom `register_setting()` call.

**Plugin & theme developers**
- If you already register your own REST setting for `wp_page_for_privacy_policy` or use the name `page_for_privacy_policy`, check for a conflict with the core registration.
- Code that validates the settings schema or snapshots its keys (e.g. tests asserting the exact key list) will see one extra property.

**Multisite / permissions**
- On multisite, `manage_privacy_options` maps to `manage_network`. A site administrator (who has `manage_options`) can read the value, but attempts to change it are silently ignored.
- The request still returns HTTP 200 and the response reports the previous value. No error is raised.

**Site owners**
- No action required.

## Technical details

**Registration** (`src/wp-includes/option.php`, `register_initial_settings()`): registers `wp_page_for_privacy_policy` in the `reading` group with `type => integer`, `show_in_rest => array( 'name' => 'page_for_privacy_policy' )`, and a description string. The docblock is tagged `@since 7.2.0`.

**Permission gate** (`src/wp-includes/rest-api.php`): new function `rest_restrict_privacy_policy_page_setting_update( $updated, $name )`, hooked in `default-filters.php`:

```php
add_filter( 'rest_pre_update_setting', 'rest_restrict_privacy_policy_page_setting_update', 10, 2 );
```

It returns `true` when `$name === 'page_for_privacy_policy'` and `! current_user_can( 'manage_privacy_options' )`. Returning `true` from `rest_pre_update_setting` short-circuits the update in `WP_REST_Settings_Controller`, so the option is not written. The controller then builds the response from the stored value, which is why the old value is returned.

The settings endpoint itself still only checks `manage_options`. The gate applies to this one setting only.

**Tests**: `rest-settings-controller.php` gains `test_update_item_privacy_policy_page` (super admin on multisite) and `test_update_item_privacy_policy_page_without_capability`, which uses a `map_meta_cap` filter to map `manage_privacy_options` to `do_not_allow`. `test_get_items` and the QUnit `wp-api-generated.js` fixtures (schema and settings response) are updated.

## Contribution

The record carries no discussion detail beyond the commit message crediting the props list and closing Trac #66045. The permission-gate callback and the multisite test case suggest the capability mismatch was considered during review, but the provided material does not describe any debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
