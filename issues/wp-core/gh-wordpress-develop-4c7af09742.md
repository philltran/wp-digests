# REST API: Reject non-string custom CSS in the global styles controller

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** ramonopoly
- **Committed:** 2026-07-16
- **Commit:** [`4c7af09742`](https://github.com/WordPress/wordpress-develop/commit/4c7af097421fc573824b854ac88422366e32a047)
- **Usefulness:** 4/5

## Summary

This commit introduces input validation for the `styles` field in the Global Styles REST API controller to prevent a PHP 8+ `TypeError`. Prior to this change, sending non-string values (like arrays or integers) via a `PUT` request to `/wp/v2/global-styles/` would bypass schema constraints and trigger a fatal error inside `validate_custom_css()`. The fix enforces a strict string type check, returning a `400` error instead of crashing the request.

## Impact

- **Headless & REST consumers**: Any client or plugin sending non-string values to the Global Styles REST endpoint will now receive a `400` response with the error code `rest_custom_css_invalid_type` instead of triggering a fatal error.
- **Plugin & theme developers**: No immediate code changes required, but integrations using the Global Styles REST API should ensure they validate and cast `styles.css` values to strings before making API calls.
- **No action required** for standard dashboard usage or themes that do not interact with the Global Styles REST API programmatically.

## Technical details

The diff modifies `WP_REST_Global_Styles_Controller::validate_custom_css()` in `src/wp-includes/rest-api/endpoints/class-wp-rest-global-styles-controller.php`. It adds a type guard at the start of the method:
```php
if ( ! is_string( $css ) ) {
    return new WP_Error(
        'rest_custom_css_invalid_type',
        __( 'CSS must be a string.' ),
        array( 'status' => 400 )
    );
}
```
This check precedes the existing `strlen()` call that previously caused the fatal error when passed non-strings. The docblock for the `$css` parameter is updated to accept `mixed`, and a new unit test `test_update_item_non_string_styles_css()` verifies the `400` response for arrays, integers, booleans, null, and objects. The REST schema itself remains unchanged; validation is now enforced strictly within the controller's custom validation logic rather than the JSON Schema definition.

## Contribution

Opened and merged on 2026-07-16 by `ramonopoly`, with props from `andrewserong`. The change was developed alongside pull request #12549 to address issue #65640. The implementation was straightforward, focusing on type guarding the internal validation method rather than modifying the request schema, which kept the fix localized and avoided broader schema migration overhead.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
