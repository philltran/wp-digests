# #80733: Fix template `modified` and `date` return value for file templates

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Bug`, `REST API Interaction`, `Backported to WP Core`
- **Merged:** [`ba4d03e`](https://github.com/WordPress/gutenberg/commit/ba4d03eb7b1eeb85a7c17ba68649f2ba30788dce)
- **Discussion:** [#80733](https://github.com/WordPress/gutenberg/pull/80733) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The REST API for `wp_template` and `wp_template_part` now returns `null` for the `modified` field when the template is file-backed, replacing the previous `false` value. Previously, `mysql_to_rfc3339()` converted the missing modification timestamp to `false`, which violated the documented `string` schema and caused type mismatches in client-side date formatters. This fix aligns the API response with the schema and prevents downstream parsing errors in headless or block editor clients.

## Impact

- **Headless & REST consumers**: API responses for file-backed templates will now return `null` instead of `false` for the `modified` property. Clients expecting a string or `null` will no longer encounter type mismatches.
- **Plugin & theme developers**: No code changes required. The schema change is backward-compatible for consumers that already handle `null`, but strict type checkers may need to accept `null` alongside `string`.
- **No action required** for standard site owners or theme authors.

## Technical details

The change introduces `gutenberg_allow_null_modified_wp_template_field()` in `lib/compat/wordpress-7.1/rest-api.php`, hooked to `rest_api_init`. It uses `register_rest_field()` to override the `modified` property for `wp_template` and `wp_template_part` endpoints. The schema is updated to `'type' => array( 'string', 'null' )`, and a custom `get_callback` intercepts the prepared value: `return isset( $item['modified'] ) && false !== $item['modified'] ? $item['modified'] : null;`. This replaces the `false` value produced by `WP_REST_Templates_Controller::prepare_item_for_response()` calling `mysql_to_rfc3339()` on a missing timestamp. A PHPUnit test in `phpunit/class-gutenberg-rest-templates-controller-test.php` verifies the behavior.

## Contribution

Opened by @ntsekouras to address a regression noted during the implementation of the template `date` field in PR #77134. Reviewers @Mamaduka and @tyxla agreed the fix should be backported to the WP 7.1 cycle rather than waiting for 7.2. The PR was merged and immediately cherry-picked to the `wp/7.1` branch to ensure inclusion in the upcoming release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
