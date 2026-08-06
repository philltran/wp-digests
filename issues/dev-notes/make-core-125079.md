# Abilities API improvements in WordPress 7.1

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Milana Cap
- **Published:** 2026-07-31
- **Tags:** `General`, `#abilities-api`, `7.1`, `dev-notes`, `dev-notes-7-1`, `rest-api`
- **Link:** [https://make.wordpress.org/core/2026/07/31/abilities-api-improvements-in-wordpress-7-1/](https://make.wordpress.org/core/2026/07/31/abilities-api-improvements-in-wordpress-7-1/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 expands the Abilities API with custom validation filters, a universal invocation action, selective field responses, and automatic type coercion for REST requests. These changes give developers finer control over ability execution, improve telemetry and auditing capabilities, and align core ability schemas for better programmatic discovery. The updates make the API more robust for headless, AI, and REST-driven architectures.

## Impact

- **Plugin & theme developers:** Callbacks for `wp_before_execute_ability` and `wp_after_execute_ability` must update their signature and `$accepted_args` to receive the new `WP_Ability` instance. REST ability run callbacks will now receive natively typed inputs instead of raw strings, which may require adjusting strict type comparisons.
- **Headless & REST consumers:** `core/get-user-info` now returns five additional profile fields and supports a `fields` input parameter for selective responses. The ability is also exposed via the REST API at `/wp-json/wp-abilities/v1/abilities`.
- **Hosting & platform teams:** No immediate action required, but auditing and telemetry plugins can now hook into `wp_ability_invoked` for comprehensive invocation tracking.
- **No breaking changes** to existing ability execution flows, though strict type checks in REST-run callbacks may need adjustment due to automatic input coercion.

## Technical details

- New filters `wp_ability_validate_input` and `wp_ability_validate_output` accept `( $is_valid, $value, $ability_name )` and allow runtime validation beyond JSON Schema. Returning `true`, `false`, or a `WP_Error` controls the outcome.
- New action `wp_ability_invoked` fires at the start of `WP_Ability::execute()` with `( $ability_name, $input, $ability )`, running before normalization, validation, permission checks, and short-circuit filters.
- `wp_before_execute_ability` and `wp_after_execute_ability` now pass the `WP_Ability` instance as a fourth parameter. Callbacks must update `$accepted_args` to receive it.
- `core/get-user-info` and `core/get-environment-info` now support an optional `fields` input array to selectively return output properties. Unknown fields trigger an `ability_invalid_input` error.
- `core/get-user-info` output now includes `first_name`, `last_name`, `nickname`, `description`, and `user_url`. The `roles` property is normalized via `array_values()`.
- REST `/run` endpoints now coerce query string parameters to types defined in the ability’s `input_schema` before validation. This changes how callbacks receive data:

  **Before (raw strings):**
  ```php
  // Query: ?input[limit]=10&input[featured]=true
  // Callback receives: [ 'limit' => '10', 'featured' => 'true' ]
  ```

  **After (coerced types):**
  ```php
  // Query: ?input[limit]=10&input[featured]=true
  // Callback receives: [ 'limit' => 10, 'featured' => true ]
  ```
- Core ability schemas now include translatable `title` and `description` metadata for programmatic clients.

## Contribution

The improvements were developed by Milana Cap with peer review from @benjamin_zekavica and @annezazu, and collaboration from @jorgefilipecosta, consolidated across tickets #64311, #65248, #65234, and #65355. The record carries no discussion detail regarding design debates or rejected alternatives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
