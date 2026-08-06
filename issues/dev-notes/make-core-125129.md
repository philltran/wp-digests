# JSON Schema preparation for client compatibility in WordPress 7.1

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Milana Cap
- **Published:** 2026-07-31
- **Tags:** `General`, `#abilities-api`, `7.1`, `dev-notes`, `dev-notes-7-1`, `rest-api`
- **Link:** [https://make.wordpress.org/core/2026/07/31/json-schema-preparation-for-client-compatibility-in-wordpress-7-1/](https://make.wordpress.org/core/2026/07/31/json-schema-preparation-for-client-compatibility-in-wordpress-7-1/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 introduces `wp_prepare_json_schema_for_client()`, a new helper that converts WordPress-internal JSON schemas into portable, client-facing JSON Schema Draft 4 representations. The function automatically strips server-only keywords (like `sanitize_callback` and `validate_callback`), normalizes `required` property declarations, and fixes empty object defaults to prevent client-side validation failures. This ensures that schemas exposed via the REST API, AI tooling, and custom endpoints remain consistent and compatible with external validators.

## Impact

- **Plugin & theme developers:** No action required. Core now applies this preparation automatically to Abilities API schemas in REST responses and AI Client declarations.
- **Custom endpoint & tool developers:** If you expose WordPress-style schemas outside the server-side validation boundary (e.g., custom REST endpoints, JavaScript configuration objects, MCP tools, or AI function declarations), you must call `wp_prepare_json_schema_for_client()` before serialization.
- **Breaking changes / Deprecations:** None. The original server-side schemas remain unchanged, and `WP_Ability::get_input_schema()` / `WP_Ability::get_output_schema()` continue returning canonical arrays.
- **Configuration / Migration:** None required. Existing ability registration and execution code continues to work without modification.

## Technical details

The new `wp_prepare_json_schema_for_client( array $schema, string $schema_profile = 'draft-04' ): array` function performs recursive schema transformations before exposing data to external consumers. It accepts two profiles: `draft-04` (default, preserves composition/reference keywords like `$ref`, `allOf`, `not`) and `rest-api` (narrower set matching REST route conventions). Key transformations include:
- Moving property-level `'required' => true` booleans to the parent object's `'required' => [ 'property_name' ]` array.
- Recursively stripping server-only keywords: `sanitize_callback`, `validate_callback`, and `arg_options`.
- Converting empty array defaults for object types from `[]` to `{}` to match the declared `type: object`.
- Filtering retained keywords via the `wp_json_schema_allowed_keywords` filter, which passes the current keyword list and `$schema_profile`.

Core automatically applies this to Abilities API schemas in `/wp-json/wp-abilities/v1/abilities` responses, while `WP_Ability` methods continue returning unprepared schemas for server-side PHP. The AI Client now prepares ability input schemas before converting them to function declarations.

Before/after example for `required` normalization:
```php
// Before (WordPress internal)
'properties' => array(
    'title' => array( 'type' => 'string', 'required' => true ),
),

// After (Client-facing)
'required' => array( 'title' ),
'properties' => array(
    'title' => array( 'type' => 'string' ),
),
```

## Contribution

The change originated from ticket #64955 to resolve AI tool-calling compatibility issues, with implementation split across changesets [62549], [62449], and [62591]. Milana Cap authored the core preparation logic, with peer review and refinements provided by @jorbin. The team deliberately separated server-side validation from client-facing serialization to avoid breaking existing ability execution flows, and deferred provider-specific schema adaptation to a follow-up in the `WordPress/php-ai-client` repository.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
