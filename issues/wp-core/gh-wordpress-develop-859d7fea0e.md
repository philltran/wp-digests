# REST API: Add a shared helper for JSON Schema allowed keywords.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Greg Ziółkowski
- **Committed:** 2026-06-23
- **Commit:** [`859d7fea0e`](https://github.com/WordPress/wordpress-develop/commit/859d7fea0e316b1bfc6bdfe8efa88e4e9402666b)
- **Usefulness:** 4/5

## Summary

Introduces `wp_get_json_schema_allowed_keywords()` as a centralized helper that dictates which JSON Schema keywords remain in REST API response payloads. The function supports two profiles: `rest-api` (default, narrower set for standard route output) and `draft-04` (broader set preserving structural vocabulary like `$ref`, `allOf`, and `definitions` for the Abilities API). It exposes the new `wp_json_schema_allowed_keywords` filter, enabling extensible schema generation without altering core validation internals.

## Impact

- **Plugin & theme developers:** No immediate action required unless you are generating OpenAPI/Swagger specs or building JSON Schema consumer tooling. The `wp_json_schema_allowed_keywords` filter provides a supported hook to inject custom keywords into REST route schema output.
- **Abilities API consumers:** The `WP_REST_Abilities_V1_List_Controller` now explicitly uses the `draft-04` profile, ensuring external validators receive the richer keyword set they expect for structural validation.
- **Platform & hosting teams:** Request/response validation behavior is untouched; internal validation still calls `rest_get_allowed_schema_keywords()` directly. No breaking changes or migration steps are required.

## Technical details

The commit creates `src/wp-includes/json-schema.php` containing `wp_get_json_schema_allowed_keywords( string $schema_profile = 'rest-api' ): array`. The function branches on `$schema_profile`, returning the base `rest_get_allowed_schema_keywords()` list for unknown or `rest-api` contexts, while merging additional draft-04 vocabulary (`$schema`, `id`, `$ref`, `required`, `allOf`, `not`, `definitions`, `dependencies`, `additionalItems`) for the `draft-04` context. It exposes a filter via `apply_filters( 'wp_json_schema_allowed_keywords', $allowed_keywords, $schema_profile )`.

Consumption changes:
- `WP_REST_Server::get_data_for_route()` now calls `wp_get_json_schema_allowed_keywords( 'rest-api' )` instead of directly invoking `rest_get_allowed_schema_keywords()`.
- `WP_REST_Abilities_V1_List_Controller` removes its hardcoded `ADDITIONAL_ALLOWED_SCHEMA_KEYWORDS` constant and delegates to `wp_get_json_schema_allowed_keywords( 'draft-04' )` via a new lazily-loaded class property.

Example usage for injecting a custom keyword into route schemas:
```php
add_filter( 'wp_json_schema_allowed_keywords', function( $keywords, $profile ) {
    if ( 'rest-api' === $profile ) {
        $keywords[] = 'x-custom-docs';
    }
    return $keywords;
}, 10, 2 );
```

## Contribution

Merged by Greg Ziółkowski on 2026-06-23 as part of ticket #64955. The change consolidates previously scattered schema keyword handling from the Abilities API controller into a single shared function. The diff shows a direct refactor that removes hardcoded constants, standardizes schema filtering across `WP_REST_Server` and REST endpoint classes, and adds targeted PHPUnit tests for both profile outputs and the new filter.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
