# #79403: Connectors: add application password settings UI

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jorgefilipecosta
- **Labels:** `[Type] Enhancement`, `[Feature] Connectors screen`
- **Merged:** [`f61955d`](https://github.com/WordPress/gutenberg/commit/f61955d76acb05cd92d8a1704d1d6a35ff5eb2f4)
- **Discussion:** [#79403](https://github.com/WordPress/gutenberg/pull/79403) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Gutenberg WordPress 7.0 compatibility layer now supports `application_password` as a valid authentication method for the Connectors API. This extends the existing `api_key` and `none` methods, enabling plugins to register connectors that authenticate via WordPress application passwords. The change introduces backend logic to parse, resolve, and sanitize `username:password` credentials from environment variables, PHP constants, or the database, and updates the REST settings endpoint to mask these credentials in API responses.

## Impact

- **Plugin & theme developers:** Developers registering connectors can now use `application_password` authentication. The auto-generated `setting_name` fallback now uses `connectors_{$type}_{$id}_{$method}` instead of hardcoding `_api_key`, which may affect custom code that constructs option names dynamically.
- **Hosting & platform teams:** No immediate configuration changes required. Environments using the Connectors REST endpoint will now see masked application password credentials alongside API keys.
- **No action required** for sites not registering custom connectors or interacting with the Connectors API.

## Technical details

The diff modifies three files in `lib/compat/wordpress-7.0/`:
- `class-wp-connector-registry.php`: Updates `WP_Connector_Registry::register()` to accept `'application_password'` in the `$args['authentication']['method']` validation array. The auto-generated `setting_name` fallback now uses `str_replace( '-', '_', "connectors_{$connector['type']}_{$id}_{$args['authentication']['method']}" )` instead of hardcoding `_api_key`.
- `default-connectors.php`: Introduces three private helper functions:
  - `_gutenberg_parse_application_password_credentials()` splits a `username:password` string on the first colon.
  - `_gutenberg_get_application_password_credentials()` resolves credentials by checking `env_var_name`, then `constant_name`, then falling back to `get_option( $auth['setting_name'] )`.
  - `_gutenberg_sanitize_application_password_credentials()` sanitizes submitted `username` and `password` fields, preserves masked passwords (`\u{2022}\u{2022}...`) to prevent accidental overwrites, and discards credentials if the username is empty.
- `connectors.php`: Updates PHPDoc for `wp_get_connector()` and `wp_register_connector()` to reflect the new authentication method and credential resolution sources.
- The REST settings dispatch logic now masks application password credentials in `/wp/v2/settings` responses and skips validation for them on POST/PUT, unlike API keys which are validated against the provider.

## Contribution

Opened and merged by @jorgefilipecosta, with co-authorship noted for @gziolo. The PR implements the application password connector form and authentication type, mirroring the companion Core API changes in `wordpress-develop#12264`. The implementation includes a guard to skip the new browser suite tests when the Core API is unavailable, allowing independent landing. Testing covers save, masking, reload, and removal flows via a real PHP connector fixture and targeted ESLint/PHPCS checks.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
