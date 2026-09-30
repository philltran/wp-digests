# REST API: Prevent fatal error when a null template reaches prepare_item_for_response()

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** ramonopoly
- **Committed:** 2026-09-04
- **Commit:** [`db9e1cd5ca`](https://github.com/WordPress/wordpress-develop/commit/db9e1cd5ca4055ce9dc84049be9b0d6540c55117)
- **Usefulness:** 3/5

## Summary

`WP_REST_Templates_Controller::prepare_item_for_response()` now returns a `WP_Error` (`rest_template_not_found`, HTTP 404) when passed a `null` template, instead of triggering an uncaught fatal (`Attempt to assign property "content" on null`). The null case can be reached from `update_item()`, which passes its `get_block_template()` refetch to this method unchecked, including after the revert-to-theme path deletes the template's post.

## Impact

- **Site owners / REST consumers:** Requests that previously ended in a PHP fatal (a 500 or broken response) on template/template-part updates now get a clean 404 JSON error, `rest_template_not_found`.
- **Plugin & theme developers:** If you call or override `prepare_item_for_response()` on this controller (or subclasses for `wp_template` / `wp_template_part`), the return type is now `WP_REST_Response|WP_Error` and the `$item` param is `WP_Block_Template|null`. Code that assumes a `WP_REST_Response` return should handle `WP_Error`.
- **Hosting / platform:** Fewer fatals in error logs from the templates endpoints.
- No configuration or migration required.

## Technical details

In `src/wp-includes/rest-api/endpoints/class-wp-rest-templates-controller.php`, an early guard is added at the top of `prepare_item_for_response()`:

```php
if ( ! $item ) {
	return new WP_Error(
		'rest_template_not_found',
		__( 'No templates exist with that id.' ),
		array( 'status' => 404 )
	);
}
```

The guard runs before the HEAD-request short-circuit and before any property access on `$item`. The docblock is updated: `$item` is `WP_Block_Template|null`, the return is `WP_REST_Response|WP_Error`, and a `@since 7.2.0` note is added. The inline comment explains that `update_item()` passes its `get_block_template()` refetches here unchecked, after both a normal update and the revert-to-theme path that deletes the template's post.

A new PHPUnit test, `test_prepare_item_for_response_with_null_template`, in `tests/phpunit/tests/rest-api/wpRestTemplatesController.php`, calls the method with `null` and asserts a `WP_Error` with code `rest_template_not_found`. The diff does not change `update_item()` itself.

## Contribution

Developed in a wordpress-develop pull request and committed by ramonopoly with props to aaronrobertshaw and westonruter. The record carries no further discussion detail.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
